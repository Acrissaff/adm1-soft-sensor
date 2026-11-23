# 0006 — Sólidos (totais e voláteis) no pós-processamento

**Status:** proposta · **Data:** 22/11/2026 · **Etapa:** 01

## Contexto

Sólidos são a terceira família de medições "fáceis" do soft-sensor e não são estado do ADM1:
os estados estão em **kg DQO/m³**, não em massa seca. A conversão exige
`Component.i_mass` (kg componente / kg substância medida), que varia de 0,35 (`X_li`) a 0,87
(`X_ch`, biomassa) nos particulados — somar as colunas do `scope` devolve DQO, não sólidos.

Duas armadilhas do conjunto ADM1: `get_VSS()` **subestima**, porque `f_Vmass_Totmass` é 0 para
`X_ch` e `X_li`; e o `S_IC` é gás dissolvido, cujo `i_mass = 3,66` converte para massa de CO₂,
não de HCO₃⁻.

## Decisão

Calcular as duas grandezas na série registrada, em mg/L de massa de componente, para o
efluente e para o influente com as mesmas máscaras:

| Grandeza | Componentes | Expressão |
|---|---|---|
| Sólidos totais (ST) | todos menos H₂O e gases dissolvidos (`S_h2`, `S_ch4`, `S_IC`) | `Σ estado·i_mass` |
| Sólidos voláteis (SVT) | todos os 12 particulados, inclusive `X_I` | `Σ estado·i_mass` |

```python
water    = np.array([ID in ('H2O', 'Water') for ID in IDs])
diss_gas = np.asarray(cmps.g, dtype=bool)
part     = np.array([c.particle_size == 'Particulate' for c in cmps])
ts_idx   = np.flatnonzero(~water & ~diss_gas)
vss_idx  = np.flatnonzero(part)
```

A rotina é aplicada a `eff.scope.record[:, :-1]` e a `inf.scope.record[:, :-1]`, ambos em
mg/L da substância medida (× 1e-3 → kg/m³ → `@ i_mass` → × 1e3 → mg/L).

Como todo particulado do ADM1 é orgânico (`i_COD = 1`), o SVT coincide numericamente com o
TSS; a classificação "inerte" de `X_I` é de **biodegradabilidade**, não de volatilidade.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| `eff.get_TSS()` / `composite('solids')` | corretos, mas só valem para o instante atual do stream; não reconstroem a série histórica |
| `eff.get_VSS()` | `f_Vmass_Totmass` = 0 em `X_ch` e `X_li`: carboidratos e lipídios entram com contribuição volátil nula |
| Somar as colunas do `scope` | estão em mg DQO/L; o fator por componente (0,35–0,87) seria ignorado |
| Incluir H₂O no ST | o resultado saltaria para ~994 g/L de água |
| Incluir `S_IC` no ST | é gás dissolvido; entraria como massa de CO₂ (44 g/mol C), não de bicarbonato |
| Excluir `X_I` do SVT (versão inicial, descartada) | `X_I` é inerte, mas orgânico (`i_COD = 1`): volatiliza na mufla, e o SVT deixaria de ser volátil **total**, virando "particulado biodegradável" |

## Consequências

- (+) Mesma convenção do QSDsan (`composite('solids')`), verificada numericamente.
- (+) ST e SVT comparáveis entre entrada e saída, na mesma unidade.
- (+) O SVT coincide com o TSS, análogo do VSS gravimétrico; antes ficava subestimado por
  excluir `X_I`.
- (−) O SVT não separa biomassa + substrato do inerte: se essa separação for necessária,
  calcular à parte como TSS − `X_I` (máscara por `degradability`).
- (−) A diferença ST − SVT é apenas a fração dissolvida (~250 mg/L no influente deste teste).
- (−) O ST inclui `S_cat` e `S_an`, cujo `i_mass` é placeholder (1,0 kg/kg), e `S_IN` como
  massa de N-NH₃; é um resíduo seco aproximado, não gravimétrico.
- Obrigação: exportar ST e SVT separados, para entrada e saída — não apenas o efluente.

## Evidências

`python ADM1_Dyn` (10 d, BDF, 72 instantes), em mg/L:

| Grandeza | mín | média | máx |
|---|---|---|---|
| ST — influente | 32.933 | 32.933 | 32.933 |
| SVT — influente | 32.680 | 32.680 | 32.680 |
| ST — efluente | 13.786 | 15.964 | 24.830 |
| SVT — efluente | 11.952 | 13.016 | 17.251 |

Influente constante porque a composição e a vazão do CSV são fixas (`Q` = 170 m³/d). Teste
sintético de validação (1 m³/d, `S_ac` 100 + `X_su` 100 + `X_li` 100 + `X_I` 50 +
`S_cat` 200 + `S_IN` 50 mg/L): ST 493,295 | TSS 140,241 | TDS 353,054 — idênticos ao
`composite('solids')`. Gera `plots/solids_dyn.png`.

## Relacionadas

- [0003 — pH calculado no pós-processamento](0003-ph-no-pos-processamento.md)
- [0004 — Alcalinidade no pós-processamento](0004-alcalinidade-no-pos-processamento.md)
- [0005 — Acidez volátil total no pós-processamento](0005-acidez-no-pos-processamento.md)
- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md) §5
