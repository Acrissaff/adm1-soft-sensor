# 0007 — DQO total (influente e efluente) no pós-processamento

**Status:** proposta · **Data:** 22/11/2026 · **Etapa:** 01

## Contexto

A DQO é a medição "fácil" central do projeto. No ADM1 ela **já é a base dos estados**: todo
componente orgânico tem `i_COD = 1` e o estado está em kg DQO/m³ — ao contrário dos sólidos
(ADR 0006), aqui não há conversão de massa. Duas armadilhas, porém: `S_IC` e `S_IN` estão em
base **elementar** (kg C/m³ e kg N/m³, `i_COD = 0`) e não podem entrar na soma; e `S_h2` e
`S_ch4` são **gases dissolvidos** que são arrastados para o biogás, mas têm `i_COD = 1`.

## Decisão

Calcular a DQO total da série registrada somando apenas os componentes com `i_COD > 0` e que
não sejam gases dissolvidos, o que reproduz a convenção de `composite('COD')`:

```python
i_COD    = np.asarray(cmps.i_COD, dtype=float)
cod_idx  = np.flatnonzero((i_COD > 0) & ~diss_gas)
cod_mgL  = (effluent_states[:, cod_idx] @ i_COD[cod_idx]) * 1e3
cod_influente_mgL = (influent_states[:, cod_idx] @ i_COD[cod_idx]) * 1e3
```

O filtro é por `i_COD`, não por nome de componente: `S_IC`/`S_IN` saem automaticamente e a
regra continua válida se o conjunto de componentes mudar. Como `i_COD = 1` no ADM1, o estado
registrado em mg/L é numericamente igual à DQO em mg/L.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Somar todas as colunas do `scope` | incluiria `S_h2`/`S_ch4` (gases que saem no biogás) e misturaria bases (mg C/L, mg N/L) |
| Somar só os particulados | seria DQO suspensa; a DQO total inclui os solúveis |
| Usar `eff.composite('COD')` | correto, mas só vale para o instante atual; não reconstrói a série |
| Converter por `i_mass` (como nos sólidos) | errado: `i_mass` é massa seca, não demanda de oxigênio |
| Excluir `X_I` | `X_I` é orgânico (`i_COD = 1`): compõe a DQO total, ainda que inerte |

## Consequências

- (+) Mesma unidade e mesma base dos estados: nenhuma conversão além de mg/L ↔ kg/m³.
- (+) Comparável entre entrada e saída e coerente com `composite('COD')`.
- (−) A DQO total inclui inerte particulado (`X_I`) e AGVs; não é DQO biodegradável.
- (−) Excluir os gases dissolvidos é uma convenção: uma amostra de efluente aerada perderia
  `S_h2`/`S_ch4` antes da análise, mas o valor "no reator" seria maior.
- Obrigação: exportar DQO total de entrada e saída no mesmo arquivo de sólidos.

## Evidências

`python ADM1_Dyn` (10 d, BDF, 72 instantes), em mg/L:

| Grandeza | mín | média | máx |
|---|---|---|---|
| DQO total — influente | 57.096 | 57.096 | 57.096 |
| DQO total — efluente | 20.726 | 23.774 | 35.840 |

Teste sintético de validação (1 m³/d; `S_ac` 100 + `X_su` 100 + `X_I` 50 + `S_h2` 30 +
`S_ch4` 40 + `S_IC` 500 + `S_IN` 50 + `S_cat` 200 mg/L): a receita dá 250,0 mg/L, idêntico a
`composite('COD')`; somar sem excluir os gases dissolvidos daria 320,0 mg/L. Gera o painel
inferior de `plots/solids_dyn.png`.

## Relacionadas

- [0003 — pH calculado no pós-processamento](0003-ph-no-pos-processamento.md)
- [0004 — Alcalinidade no pós-processamento](0004-alcalinidade-no-pos-processamento.md)
- [0005 — Acidez volátil total no pós-processamento](0005-acidez-no-pos-processamento.md)
- [0006 — Sólidos totais e voláteis no pós-processamento](0006-solidos-no-pos-processamento.md)
- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md) §5
