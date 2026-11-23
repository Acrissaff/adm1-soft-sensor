# 01 — Contexto, objetivos e visão do projeto

**Status:** concluído · **Última atualização:** 21/11/2026

Guia inicial do projeto: registra o contexto, os objetivos e a visão de construção. Os
detalhes técnicos das implementações iniciais estão nos registros de decisão indicados na
seção 5.

## 1. Contexto

Este projeto propõe a construção de um **soft-sensor para processos de digestão anaeróbia**.
Dados reais de reatores são escassos e raramente trazem a rotulagem dos estados internos
(AGVs, acetato, biomassa por grupo trófico), o que inviabiliza o treinamento direto de um
modelo supervisionado.

A estratégia adotada é usar o **ADM1** (modelo IWA de digestão anaeróbia) para síntese de
dados: um modelo robusto que gera séries com todos os estados conhecidos e permite construir
o soft-sensor em um cenário controlado antes de aplicá-lo a um caso real.

## 2. Objetivo

Construir um modelo que estime, a partir de análises simples — alcalinidade, pH, acidez, DQO
e outros parâmetros de obtenção fácil —, variáveis de difícil estimativa em um cenário real:
concentração de VFAs (e de acetato), nitrogênio inorgânico (IN), quantidade de células e
biomassa digestora no reator.

## 3. Visão de construção

| Fase | Conteúdo | Situação |
|---|---|---|
| **1. Reator virtual** | Modelo robusto em ADM1 para síntese dos dados | em andamento |
| **2. Soft-sensor** | Modelo que mapeia medições fáceis → variáveis de difícil medição | a iniciar |

A Fase 1 fornece o campo de treinamento (rótulos completos, cenários controlados); a Fase 2
é o produto final do projeto.

> Os parâmetros usados hoje nos scripts (vazão, HRT, temperatura, composição de entrada e
> perturbação de vazão) são **valores de teste**, adotados para verificar se a implementação
> funciona. Não são especificações do projeto nem condições de operação definidas.

## 4. Implementações iniciais

| Artefato | O que é |
|---|---|
| [ADM1](../ADM1) | simulação do reator com influente constante |
| [Dynamic_builder](../Dynamic_builder) | gera a série temporal de influente em CSV (`influent_data.csv`) |
| [ADM1_Dyn](../ADM1_Dyn) | simulação dinâmica com influente variável, cálculo de pH, alcalinidade, acidez, sólidos e DQO, e exportação da série em CSV (nativa e em grade de 1 h) |
| `influent_data.csv` | saída do `Dynamic_builder` |
| `plots/` | figuras geradas pelos scripts de simulação |

Essas implementações foram executadas apenas para verificar que rodam e produzem as figuras
esperadas; os resultados numéricos não têm, por ora, valor de referência de projeto.

## 5. Decisões registradas

- [0001 — QSDsan como motor de simulação do ADM1](decisoes/0001-qsdsan-como-motor-de-simulacao.md)
- [0002 — Influente dinâmico em CSV, em mg/L, com t em dias](decisoes/0002-influente-dinamico-em-csv.md)
- [0003 — pH calculado no pós-processamento](decisoes/0003-ph-no-pos-processamento.md)
- [0004 — Alcalinidade no pós-processamento](decisoes/0004-alcalinidade-no-pos-processamento.md)
- [0005 — Acidez no pós-processamento](decisoes/0005-acidez-no-pos-processamento.md)
- [0006 — Sólidos no pós-processamento](decisoes/0006-solidos-no-pos-processamento.md)
- [0007 — DQO total no pós-processamento](decisoes/0007-dqo-total-no-pos-processamento.md)
- [0008 — Exportação da série simulada em CSV](decisoes/0008-exportacao-csv.md)

## 6. Próximos passos

1. **Gerador de cenários**: evoluir o `Dynamic_builder` para produzir múltiplos cenários de
   operação, com semente fixa e metadados por cenário.
2. **Observáveis "fáceis"**: derivar do ADM1 as variáveis que o soft-sensor terá como entrada
   (alcalinidade, acidez, DQO), que não são estados do modelo.
3. **Dataset rotulado**: exportar estados e observáveis em grade de tempo regular.
4. **Soft-sensor**: definir a arquitetura do modelo e a estratégia de validação.

## 7. Referências

1. Batstone, D. J. et al. *The IWA Anaerobic Digestion Model No 1 (ADM1)*. Water Science and
   Technology, 45(10), 65–73, 2002.
2. Rosen, C.; Jeppsson, U. *Aspects on ADM1 Implementation within the BSM2 Framework*. Lund
   University, 2006.
3. QSDsan — implementação do ADM1 utilizada no projeto (v. 1.5.3).

## 8. Histórico de revisões

| Data | Mudança |
|---|---|
| 21/11/2026 | Criação do guia |
| 21/11/2026 | Simplificação: mantidos apenas contexto, objetivos e visão; detalhes técnicos movidos para os registros de decisão |
| 22/11/2026 | `ADM1_Dyn` passa a calcular também a acidez (ADR 0005), a série de sólidos (ADR 0006), a DQO total (ADR 0007) e a exportar a série em CSV (ADR 0008) |
