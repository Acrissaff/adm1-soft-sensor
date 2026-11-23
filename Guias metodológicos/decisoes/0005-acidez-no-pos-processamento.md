# 0005 — Acidez volátil total no pós-processamento

**Status:** proposta · **Data:** 22/11/2026 · **Etapa:** 01

## Contexto

A acidez é outra medição "fácil" do soft-sensor e, junto com a alcalinidade, forma o índice
de estabilidade clássico (razão AGV/alcalinidade). Como o pH e a alcalinidade, **não é
variável de estado do ADM1**: os AGVs existem no modelo apenas como soma das espécies
(aniônica + não dissociada) e a partição depende do pH resolvido pelo balanço de cargas.

A nomenclatura pede cuidado: em química analítica, *acidez titulável* é a quantidade de
**base** consumida até um ponto final (fenolftaleína, pH 8,3). A fração aniônica dos AGVs,
`Σ S_vfa·Ka/(Ka + h)`, é o oposto — capacidade de **consumir ácido**, ou seja, exatamente a
parcela de alcalinidade já calculada no ADR 0004. Por ser a mesma grandeza, não é derivada
aqui.

## Decisão

Derivar **uma** grandeza da série do efluente, com a rotina ácido-base dos ADR 0003 e 0004
(`Ka` corrigido por van't Hoff, `h` de `adm1.solve_pH`), em mg/L como CaCO₃:

| Grandeza | Expressão no script | O que é |
|---|---|---|
| acidez volátil total | `acidez_volatil_total = Σ S_vfa · 50 000` | AGVs totais, todas as espécies; independe do pH |

É a acidez "potencial" do pool de AGVs: corresponde ao AGV de um método que destrói a
especiação (acidificação/destilação + titulação), apenas expresso em equivalentes de CaCO₃
(1 eq/mol, pois acetato, propionato, butirato e valerato são monopróticos). O complemento da
parcela de alcalinidade — `acidez_volatil_total − alk_vfa` — é o AGV **não dissociado**, que
não consome alcalinidade.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Calcular também a "acidez titulável" `Σ S_vfa·Ka/(Ka + h)` | é algebricamente idêntica a `alk_vfa` (mesmos `Ka`, mesmo `h`): série duplicada, sem informação nova |
| Acidez titulável potenciométrica até pH 8,3 (Standard Methods 2310 B) | exigiria somar a desprotonação de NH₄⁺ (pKa 9,25): ~9% de `S_IN` na faixa 7,0–7,5, ≈ 440 mg CaCO₃/L — domina o resultado e não é indicador de digestor |
| Titular até pH 4,4 | é a alcalinidade (ADR 0004), sentido oposto: mede ácido consumido, não base |
| Reportar AGV em mg/L como ácido acético | válido, mas a razão AGV/alcalinidade exigiria conversão a cada uso; o projeto adota CaCO₃ |
| Usar `S_va` como valerato isolado e ignorar butirato | ADM1 trata C4 e C5 como um único estado cada (`S_bu`, `S_va`); separá-los exigiria o ADM1p |

## Consequências

- (+) Mesma base ácido-base do pH e da alcalinidade; nenhuma constante nova.
- (+) Permite a razão AGV/alcalinidade e, por diferença com `alk_vfa`, a fração não dissociada.
- (−) É acidez **potencial** (independe do pH): compara-se com o AGV de um método
  titulométrico/destilação, não com uma titulação direta da amostra.
- (−) Em pH 7,0–7,5 a curva praticamente coincide com `alk_vfa` (diferem < 0,6%), porque
  `Ka/(Ka + h) ≈ 0,995`; a informação própria está na razão com a alcalinidade, não no valor
  absoluto.
- Obrigação: ao exportar o dataset, gravar `acidez_volatil_total` junto da soma
  `alk_total`/`alk_total_conservativa` para que a razão seja reproduzível.

## Evidências

`python ADM1_Dyn` (10 d, BDF, T = 308,15 K, 103 instantes, pH 7,02–7,50), em mg/L como CaCO₃:

| Grandeza | mín | média | máx |
|---|---|---|---|
| acidez volátil total | 85 | 1.482 | 3.321 |
| razão AGV/alcalinidade conservativa | 0,02 | 0,24 | 0,52 |

Resíduo máximo do balanço de cargas: 4,8 × 10⁻⁸ eq/L. Painel inferior de
`plots/reactor_alkalinity_dyn.png`.

## Relacionadas

- [0003 — pH calculado no pós-processamento](0003-ph-no-pos-processamento.md)
- [0004 — Alcalinidade no pós-processamento](0004-alcalinidade-no-pos-processamento.md)
- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md) §5
