# 0001 — QSDsan como motor de simulação do ADM1

**Status:** aceita · **Data:** 20/11/2026 · **Etapa:** 01

## Contexto

A Fase 1 precisa de um ADM1 que seja (i) fiel à publicação original (Batstone et al., 2002),
(ii) estável numericamente para horizontes de semanas, (iii) reutilizável por um pipeline de
geração de dados e (iv) inspecionável o suficiente para extrair estados internos, pH e taxas.
As alternativas eram implementar as 22 equações de taxa à mão, usar uma implementação
acadêmica isolada (MATLAB/Simulink, BSM2) ou usar uma biblioteca Python mantida.

## Decisão

Usar o **QSDsan 1.5.3**, que já traz o ADM1 (`create_adm1_cmps`, `ADM1`), o reator anaeróbio
(`AnaerobicCSTR`), o gerador de influente dinâmico (`DynamicInfluent`) e o rastreamento de
séries (`WasteStreamScope`) no mesmo ambiente Python do restante do projeto.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Implementar o ADM1 do zero em Python | alto custo e alto risco de erro silencioso (estequiometria, inibição por pH, ácido-base); desviaria o esforço do objetivo (soft-sensor) |
| BSM2 em MATLAB/Simulink | exige licença e segunda linguagem; integração ruim com o pipeline de ML em Python; menos transparente para inspeção de estados |
| Implementação acadêmica isolada | em geral sem manutenção, sem testes e sem tratamento numérico robusto |

## Consequências

- (+) Entrega os 27 componentes, os 22 processos, a resolução ácido-base (`solve_pH`,
  `acid_base_rxn`) e o reator com *headspace* já validados.
- (+) Mantém tudo em Python/NumPy/SciPy, mesmo ecossistema do ML (pandas, scikit-learn).
- (−) Duas dependências pesadas de contexto: é preciso chamar `create_adm1_cmps()` **antes**
  de `ADM1()` (senão `RuntimeError: no available 'Thermo' object`).
- (−) Convenções de unidade internas mistas (kg/m³ no modelo, mg/L nos scopes e no
  `DynamicInfluent`) — exigem disciplina em cada conversão.
- Obrigação: registrar a versão usada em todo guia/resultado (hoje 1.5.3), pois mudanças de
  versão podem alterar resultados numéricos.

## Evidências

- `python ADM1` e `python ADM1_Dyn` executam em ~35 s cada, gerando as figuras de `plots/`.
- Listagem de processos: `ADM1([disintegration, hydrolysis_carbs, hydrolysis_proteins,
  hydrolysis_lipids, uptake_sugars, uptake_amino_acids, uptake_LCFA, uptake_valerate,
  uptake_butyrate, uptake_propionate, uptake_acetate, uptake_h2, decay_Xsu, …, IC_transfer])`.
- 27 componentes: 12 solúveis + 12 particulados + `S_cat` + `S_an` + `H2O`.

## Relacionadas

- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md)
