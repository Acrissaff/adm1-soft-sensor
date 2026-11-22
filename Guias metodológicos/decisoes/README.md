# Registro de decisões (ADRs)

Cada arquivo desta pasta registra **uma** decisão técnica relevante: o contexto, a escolha
feita, as alternativas descartadas e o que a decisão obriga o projeto a fazer daqui para
frente.

O guia metodológico (`../NN-*.md`) conta *o que* foi implementado e *como*; este registro
guarda *por quê*. Quando, meses depois, surgir a dúvida "podemos mudar o integrador?" ou
"por que o CSV está em mg/L?", a resposta está aqui.

## Quando criar uma decisão

Crie um registro quando a escolha:

- for difícil de reverter (estrutura de dados, formato do dataset, biblioteca-base);
- puder ser questionada depois (parâmetros de referência, horizonte de simulação);
- tiver alternativas razoáveis que alguém tentaria novamente;
- tiver custo de aprendizado embutido (armadilhas de unidade, ordem de chamadas).

Não registre trivialidades (nome de variável, estilo de código) nem o que já está óbvio no
guia.

## Convenção

| Item | Regra |
|---|---|
| Nome do arquivo | `NNNN-tema-curto.md`, quatro dígitos, minúsculas, sem acentos |
| Numeração | sequencial a partir de `0001`, **sem lacunas** |
| Remoção de registro | renumerar os seguintes para fechar a sequência, atualizando o índice e as referências nos guias |
| Substituição | uma decisão revista **não** é apagada: seu status passa a `substituída por NNNN` |
| Status possíveis | `proposta` · `aceita` · `substituída por NNNN` · `revogada` |
| Tamanho | 1 página; contexto e consequências valem mais que texto longo |

## Template

```markdown
# NNNN — Título

**Status:** aceita · **Data:** DD/MM/AAAA · **Etapa:** NN

## Contexto
Situação, restrições e o problema que precisava ser resolvido.

## Decisão
Uma frase objetiva com a escolha feita.

## Alternativas consideradas
| Alternativa | Por que não foi escolhida |

## Consequências
- (+) ...
- (−) ...
- Obrigações: ...

## Evidências
Execuções, números, trechos de código, referências.

## Relacionadas
- Guia: ../NN-tema.md
```

## Índice

| Nº | Decisão | Status | Data |
|---|---|---|---|
| [0001](0001-qsdsan-como-motor-de-simulacao.md) | QSDsan como motor de simulação do ADM1 | aceita | 20/11/2026 |
| [0002](0002-influente-dinamico-em-csv.md) | Influente dinâmico em CSV (mg/L, t em dias) | aceita | 20/11/2026 |
| [0003](0003-ph-no-pos-processamento.md) | pH calculado no pós-processamento | aceita | 21/11/2026 |
| [0004](0004-alcalinidade-no-pos-processamento.md) | Alcalinidade (bicarbonato, AGVs e total) no pós-processamento | proposta | 21/11/2026 |
