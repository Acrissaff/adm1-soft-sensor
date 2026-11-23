# 0008 — Exportação da série simulada em CSV

**Status:** proposta · **Data:** 22/11/2026 · **Etapa:** 01

## Contexto

O dataset rotulado é a saída da Fase 1: os estados completos mais os observáveis "fáceis"
(ADR 0003–0007) têm de sair num único arquivo, alinhados no tempo. Duas armadilhas: as
convenções de estado **mudam com a fase** da corrente, e o eixo de tempo gravado **não** é o
`t_eval` pedido ao integrador.

`WasteStream._init_state` e `AnaerobicCSTR._update_state` definem:

| Corrente | `state[:-1]` | `state[-1]` |
|---|---|---|
| líquida (`phase == 'l'`) | concentração na substância medida, mg/L | vazão volumétrica, m³/d |
| gasosa | concentração na substância medida, mg/L **de gás** | vazão de biogás do modelo, m³/d |

## Decisão

Dois arquivos na raiz do projeto, com as mesmas 108 colunas, uma linha por instante:

- `series_dyn.csv` — **grade nativa** (passos do solver): auditoria e conferência;
- `dataset_1h.csv` — **grade regular de 1 h** (241 linhas): o dataset de trabalho.

Colunas prefixadas pela corrente:

| Grupo | Colunas | Unidade |
|---|---|---|
| tempo | `t` | d |
| estados — influente | `inf_<ID>`, 27 componentes + `inf_Q` | mg/L; m³/d |
| estados — efluente | `eff_<ID>`, idem | mg/L; m³/d |
| estados — biogás | `gas_<ID>`, idem | mg/L (gás); m³/d |
| composição do biogás | `gas_x_<ID>` (base úmida) e `gas_x_seca_<ID>` (base seca); só os componentes da fase gasosa (`S_h2`, `S_ch4`, `S_IC`, `H2O`) | mol/mol |
| pós-processamento | `pH`, `alk_hco3`, `alk_vfa`, `alk_total`, `alk_conservativa`, `acidez_volatil_total` | mg/L como CaCO₃ (pH adimensional) |
| | `ts_inf`, `vss_inf`, `ts_eff`, `vss_eff`, `cod_inf`, `cod_eff`, `tan_inf`, `tan_eff`, `fan_eff` | mg/L |

No `ADM1_Dyn`, a física do pós-processamento vive em `derivar(estados_efluente,
estados_influente, estados_gas, Q_gas)`, aplicada **duas vezes**: à grade nativa e à grade
reamostrada. Assim não há duas versões da mesma conta, só duas grades.

Reamostragem para 1 h — **os estados são interpolados e os derivados são recalculados**:

| Corrente | Método | Motivo |
|---|---|---|
| `eff_*` e `gas_*` | interpolação linear | séries contínuas |
| `inf_*` | último valor (`ffill`) | o `DynamicInfluent` é degrau; interpolar inventaria composições de transição |

O recálculo importa porque pH, alcalinidade e FAN **não são lineares** no estado: interpolar
essas variáveis quebraria o balanço de cargas. No recálculo, o resíduo do balanço permanece
~2,4 × 10⁻³ mg/L como CaCO₃ em todos os 241 pontos.

A grade nativa é limitada a 1 h com `max_step=t_step` no `sys.simulate` (o `t_eval` continua
ignorado pelo solver), o que reduz o erro de interpolação a praticamente zero — 278 registros,
passo máximo exatamente 1/24 d.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Um CSV por corrente | exigiria junção por tempo e multiplicaria arquivos |
| Exportar só os estados | perderia os observáveis, que são o rótulo/entrada do soft-sensor |
| Interpolar as variáveis derivadas (1a) | pH/alcalinidade/FAN são não lineares no estado: a interpolação não fecha o balanço de cargas; recalcular custa apenas um `solve_pH` por ponto |
| Exportar só a grade de 1 h | perderia o rastro nativo para auditoria; os dois arquivos custam uma passada extra de pós-processamento |
| Só composição em base úmida | o relato de biogás é convencionalmente em base seca; ambas saem para não perder informação |
| Descartar `H2O` | é solvente; manter permite conferir o balanço de massa (no influente é 0) |
| Calcular a composição sobre os 27 componentes | o `scope` registra todos, mas os 23 que não são da fase gasosa são zero e têm MW placeholder; a máscara `diss_gas \| H2O` deixa a intenção explícita e não depende de eles permanecerem zero |

## Consequências

- (+) Um arquivo por grade, tudo alinhado no tempo, sem conversão escondida.
- (+) Base úmida e seca lado a lado na composição do gás.
- (+) O dataset de trabalho sai em grade exata de 1 h sem violar o equilíbrio ácido-base.
- (+) `max_step` também tornou a execução mais rápida (28,7 s → 8,4 s no teste), apesar dos
  278 registros: passos menores convergem com menos iterações de Newton.
- (−) A grade nativa continua irregular (18 passos < 10⁻³ d no arranque); só o
  `dataset_1h.csv` é uniforme.
- (−) Estatísticas descritivas calculadas sobre a grade nativa não são comparáveis às da grade
  de 1 h (a densidade de amostragem muda a média).
- (−) `gas_S_*` é o estado do modelo, não uma densidade de gás ideal: a concentração implícita
  (~98 mol/m³) é ~2,4× a ideal a 1 atm e 308 K. Use `gas_Q` e as frações molares; não use
  `gas_S_*` como densidade.
- (−) `gas_Q` é zero nos primeiros instantes (headspace ainda sem sobrepressão).
- Obrigação: toda variável nova de pós-processamento entra em `derivar`, nunca só na grade
  nativa — senão o dataset de 1 h fica incompleto.

## Evidências

`python ADM1_Dyn` (10 d, BDF, `max_step = 1/24` d) → `series_dyn.csv` 278 × 108 e
`dataset_1h.csv` 241 × 108.

| Verificação | Resultado |
|---|---|
| grade do `dataset_1h` | 241 linhas, passo 1/24 d em **todos** os intervalos |
| resíduo do balanço de cargas | 1,9 × 10⁻³ (nativa) e 2,4 × 10⁻³ mg/L como CaCO₃ (1 h) |
| `inf_*` na grade | constante (ffill correto), `inf_Q` = 170 m³/d |
| `gas_Q` | 0 → 2.238 m³/d |
| CH₄ base seca | 92,5% → 95,4% |
| `tan_inf` / `tan_eff` | 140,07 → 1.323,6 / 1.705,9 mg N/L |
| `fan_eff` | máx. 45,7 mg N/L |
| `ts_inf` / `vss_inf` | 32.933 / 32.680 mg/L |
| valor em t = 10 d (nativa × 1 h) | `alk_total` 6.891,324 × 6.891,324; `cod_eff` 35.840,6 × 35.840,6 |

## Relacionadas

- [0003 — pH](0003-ph-no-pos-processamento.md) · [0004 — Alcalinidade](0004-alcalinidade-no-pos-processamento.md) · [0005 — Acidez](0005-acidez-no-pos-processamento.md) · [0006 — Sólidos](0006-solidos-no-pos-processamento.md) · [0007 — DQO](0007-dqo-total-no-pos-processamento.md)
- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md) §5
