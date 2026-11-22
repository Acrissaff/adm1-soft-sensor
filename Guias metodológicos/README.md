# Guias metodológicos

Esta pasta guarda o **histórico técnico** do projeto: o que foi construído, como, com quais
ferramentas, quais parâmetros foram adotados e por quê. O objetivo é duplo:

1. **Memória do projeto** — permitir retomar qualquer etapa meses depois sem depender de
   memória ou de ler todo o código.
2. **Base do relatório** — cada guia é escrito para ser recortado e colado no relatório
   final (metodologia, resultados, discussão), com números já verificados.

## Organização

```text
Guias metodológicos/
├── README.md      # este arquivo: convenções, templates e índice
├── NN-tema.md     # um guia por etapa implementada (o que + como + resultados)
└── decisoes/
    └── NNNN-tema.md   # um registro por decisão relevante (por quê + alternativas)
```

**Guia ≠ decisão.** O guia conta *o que foi feito e como*; a decisão registra *por que* uma
alternativa foi escolhida em vez de outra. Uma etapa normalmente gera um guia e uma ou mais
decisões.

**Teste ≠ decisão.** Parâmetros provisórios usados só para verificar se o código funciona
(vazões, volumes, temperaturas, perturbações de teste) **não** são decisões de projeto e não
devem virar registro. Se forem mencionados, que fique explícito que são valores de teste.

## Convenções

| Item | Regra |
|---|---|
| Nome do guia | `NN-tema-curto.md` (dois dígitos, minúsculas, sem acentos, hífen) — ex.: `02-observaveis-faceis.md` |
| Nome da decisão | `decisoes/NNNN-tema-curto.md` (quatro dígitos, sequencial, nunca renumerar) |
| Idioma | português |
| Unidades | **sempre explícitas** ao lado do número (`m³/d`, `kg COD/m³`, `mg/L`, `d`, `K`) |
| Números | só registrar valores **verificados** em execução, com data e comando de origem |
| Referência ao código | citar arquivo e trecho, ex.: `ADM1_Dyn` (bloco de cálculo de pH) |
| Status | cada guia começa com uma linha `Status: concluído / em andamento / planejado` |
| Datas | formato `DD/MM/AAAA` |
| Versionamento | ao alterar uma etapa já documentada, **atualizar o guia** e registrar a mudança em "Histórico de revisões" |

## Template de guia de etapa

Copie o bloco abaixo para criar `NN-tema.md`:

```markdown
# NN — Título da etapa

**Status:** em andamento · **Última atualização:** DD/MM/AAAA

## 1. Objetivo
O que esta etapa entrega e por que ela é necessária para o soft-sensor.

## 2. Entradas e pré-requisitos
Arquivos, dados, etapas anteriores, bibliotecas e versões.

## 3. Método
Passo a passo do que foi implementado. Equações, unidades, hipóteses e justificativas.

### 3.1 Parâmetros adotados
Tabela: parâmetro | valor | unidade | origem/justificativa.

## 4. Implementação
Arquivo(s), estrutura do código, decisões de engenharia (armadilhas de unidade, ordem de
chamadas, etc.). Trechos curtos de código quando ajudarem a explicar.

## 5. Resultados
Tabelas e figuras (`plots/...`), com números verificados e a data da execução.

## 6. Verificação
Como sei que está correto: testes manuais, comparações, checagens de balanço, reprodutibilidade.

## 7. Limitações e riscos
O que a etapa não cobre.

## 8. Próximos passos
Lista acionável.

## 9. Decisões relacionadas
- [NNNN — título](decisoes/NNNN-tema.md)

## 10. Histórico de revisões
| Data | Mudança |
|---|---|
```

## Template de registro de decisão

Formato ADR (*Architecture Decision Record*) resumido — detalhes em
[`decisoes/README.md`](decisoes/README.md). No template, `decisoes/NNNN-tema.md` é apenas
um marcador a ser substituído pelo número/tema da decisão real.

```markdown
# NNNN — Título da decisão
**Status:** aceita · **Data:** DD/MM/AAAA

## Contexto
Situação e restrições que motivaram a decisão.

## Decisão
O que foi decidido, em uma frase objetiva.

## Alternativas consideradas
| Alternativa | Por que não foi escolhida |

## Consequências
Positivas (+), negativas (−) e o que passa a ser obrigatório no projeto.

## Evidências
Execuções, números, referências.
```

## Índice

| Etapa | Guia | Status | Data |
|---|---|---|---|
| 01 — Contexto, objetivos e visão do projeto | [01-modelo-adm1-e-influente-dinamico.md](01-modelo-adm1-e-influente-dinamico.md) | concluído | 21/11/2026 |

## Rotina ao final de cada etapa

1. Rodar a etapa e **anotar os números** obtidos (tempo de execução, faixas de variáveis).
2. Criar/atualizar o guia `NN-tema.md` usando o template.
3. Para cada escolha que poderia ter sido diferente **e que valha para o projeto**, criar um
   `decisoes/NNNN-*.md`. Parâmetros de teste não entram aqui.
4. Atualizar as tabelas de índice **deste README** e do `README.md` da raiz.
5. Conferir: unidades explícitas, caminhos de arquivo corretos, links funcionando.
6. Registrar o que ficou pendente em "Próximos passos" — é daqui que sairá a lista de tarefas
   da próxima sessão.
