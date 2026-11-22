# 0004 — Alcalinidade nas três parcelas, no pós-processamento

**Status:** proposta · **Data:** 21/11/2026 · **Etapa:** 01

## Contexto

A alcalinidade é uma das medições "fáceis" do soft-sensor e um indicador de estabilidade do
reator. Como o pH, **não é variável de estado do ADM1**: resulta do equilíbrio ácido-base
resolvido a cada avaliação do modelo. O `WasteStreamScope` registra apenas os estados, e a
propriedade `WasteStream.SAlk` do QSDsan 1.5.3 **não** calcula alcalinidade a partir da
composição — devolve o valor informado no construtor (ou 2,5 por padrão).

## Decisão

Calcular a alcalinidade no pós-processamento, reutilizando a rotina do pH (ADR 0003), com
três parcelas em **mg/L como CaCO₃** (1 equivalente = 50 g CaCO₃):

| Parcela | Expressão |
|---|---|
| bicarbonato | `[HCO3-] = S_IC·Ka_CO2/(Ka_CO2 + h)` |
| AGVs | `Σ [VFA-] = Σ S_vfa·Ka_vfa/(Ka_vfa + h)` para acetato, propionato, butirato, valerato |
| total | soma das duas |

`h` é o `[H+]` devolvido por `adm1.solve_pH` e as constantes `Ka` são corrigidas por van't Hoff
para a temperatura do reator. **`S_cat` e `S_an` não entram como parcela aditiva**: já estão
embutidos em `h`, porque o pH é obtido impondo o balanço de cargas. Somá-los de novo
contaria duas vezes o mesmo efeito.

## Uso de `S_cat` e `S_an`

Os dois são íons fortes **inertes**: têm coeficiente zero em todos os 22 processos do ADM1 e
variam apenas por diluição (verificado em `adm1.stoichiometry`). O balanço de cargas do modelo é

```
S_cat + H+ + NH4+ = S_an + OH- + HCO3- + Ac- + Pr- + Bu- + Va-
```

de modo que, exatamente,

```
bicarbonato + AGVs + NH3 + OH- - H+  =  (S_cat - S_an) + S_IN
```

Por isso eles entram em **dois papéis auxiliares**, não como quarta parcela:

1. **Fechamento numérico**: a diferença entre os dois lados é o resíduo que valida o cálculo
   (deve ser ~0; obtido 4,8 × 10⁻⁸ eq/L).
2. **Alcalinidade total conservativa**: `(S_cat - S_an) + S_IN` independe do pH e do
   carbono inorgânico; é o análogo da alcalinidade titulada até ~pH 4,4, que inclui a
   protonação da amônia. Difere da soma bicarbonato + AGVs justamente pelo termo `NH3`
   (pequeno na faixa usual de pH 7–7,5, mas não desprezível se o pH subir).

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Somar `S_cat - S_an` como parcela extra da total | dupla contagem: esse termo já está implícito em `h` e no balanço de cargas |
| Usar `WasteStream.SAlk` | na 1.5.3 é valor de entrada, não derivado da composição |
| Adicionar alcalinidade como estado do modelo | obrigaria a alterar o ADM1; a alcalinidade é derivada algébrica, não diferencial |
| Usar apenas `(S_cat - S_an) + S_IN` como total | correto, mas esconde as parcelas e não separa bicarbonato de AGVs, que são alvos do soft-sensor |
| Aproximação empírica (razão AGV/alcalinidade) | perde consistência com o modelo e não serve de rótulo "verdadeiro" |

## Consequências

- (+) Alcalinidade consistente com o equilíbrio do próprio modelo, sem alterar a simulação.
- (+) Separa bicarbonato e AGVs, que são alvos do soft-sensor, e fornece o fechamento de
  cargas como teste automático de consistência.
- (+) Nenhuma constante nova é introduzida; só `Ka_base`, `Ka_dH`, `T_base` e os fatores de
  `mass2mol_conversion`, já usados no pH.
- (−) As parcelas são grandezas **de equilíbrio**, não uma titulação: os pontos finais de
  laboratório (5,75 e 4,4) não coincidem com a protonação completa de cada espécie, então
  os valores não são idênticos aos de um método titulométrico.
- (−) A conversão de base é a mesma armadilha do pH: `S_IC` está em kg C/m³ e os AGVs em
  kg DQO/m³; tratar o `Scope` como massa de molécula erra o resultado sem aviso.
- Obrigação: ao gerar o dataset, exportar as parcelas separadas (bicarbonato, AGVs, total e
  conservativa) — não apenas a soma.
- Nota de nomenclatura: a literatura de digestão (Ripley et al., 1986) chama de **alcalinidade
  parcial** a titulada a pH 5,75 (bicarbonato) e de **intermediária** a diferença até pH 4,4
  (AGVs). O projeto adota o inverso; se a nomenclatura for mantida, registrá-la no guia.

## Evidências

`python ADM1_Dyn` (10 d, BDF, T = 308,15 K, 103 instantes registrados):

| Grandeza | mín | média | máx |
|---|---|---|---|
| bicarbonato (mg CaCO₃/L) | 2.988 | 3.909 | 4.711 |
| AGVs (mg CaCO₃/L) | 85 | 1.475 | 3.303 |
| bicarbonato + AGVs | 4.562 | 5.384 | 6.661 |
| `(S_cat - S_an) + S_IN` | 4.713 | 5.493 | 6.789 |

Resíduo máximo do balanço de cargas: 4,8 × 10⁻⁸ eq/L. Gera `plots/reactor_alkalinity_dyn.png`.

## Relacionadas

- [0003 — pH calculado no pós-processamento](0003-ph-no-pos-processamento.md)
- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md) §5
