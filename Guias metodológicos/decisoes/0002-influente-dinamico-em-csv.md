# 0002 — Influente dinâmico em CSV, em mg/L, com t em dias

**Status:** aceita · **Data:** 20/11/2026 · **Etapa:** 01

## Contexto

O `DynamicInfluent` do QSDsan não sintetiza séries: ele **interpola** uma série fornecida em
arquivo (`.csv`, `.tsv`, `.xlsx`). Era preciso definir um formato de intercâmbio entre o
gerador (`Dynamic_builder`) e o simulador (`ADM1_Dyn`) que fosse (i) legível por humanos,
(ii) estável para versionamento e (iii) compatível com medições experimentais futuras, que
normalmente vêm em mg/L.

## Decisão

Adotar um CSV com **uma linha por instante**, contendo:

- coluna `t` em **dias** (float, começando em 0);
- coluna `Q` em **m³/d**;
- uma coluna por componente do ADM1 (mesmos IDs: `S_su`, `X_pr`, …) em **mg/L**;
- decimais com ponto; sem índice do pandas.

Isso corresponde a `DynamicInfluent(..., by_mass=False)` (padrão), que internamente chama
`set_flow_by_concentration(flow_tot=Q, concentrations=y0, units=('m3/d','mg/L'))`.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| `by_mass=True` (vazões mássicas em kg/h) | o estado da unidade e do efluente passaria a ser mássico (g/d), incompatível com as premissas a jusante |
| Manter tudo em kg/m³ (base interna do ADM1) | contraria o formato esperado pelo `DynamicInfluent` e a convenção usual de laboratório (mg/L) |
| Representar o tempo como `datetime` | o modelo usa dias (float); datas criam ambiguidade de fuso e não são usadas pela unidade |
| Passar a série como `DataFrame` em memória | a unidade lê de arquivo; o CSV também serve de artefato versionável |

## Consequências

- (+) Formato único, auditável, independente do Python e compatível com dados reais futuros.
- (+) Conversão explícita e isolada (`Fac = 1000` no gerador), fácil de auditar.
- (−) Terceira convenção de unidade no projeto (kg/m³ no ADM1, mg/L no CSV/scope/init) —
  fonte provável de erros; obrigatório declarar unidades em todo código novo.
- (−) O `ADM1_Dyn` cita o CSV por **caminho absoluto** (herança da implementação inicial):
  portabilidade pendente.
- Obrigação: qualquer novo gerador deve produzir exatamente essas colunas e unidades, ou
  atualizar esta decisão.

## Evidências

- `influent_data.csv`: 241 linhas × 27 colunas (25 componentes + `t` + `Q`), `t` de 0 a 10 d.
- Execução do `ADM1_Dyn`: a série é lida e chega ao reator na coluna `Q` do escopo do influente.

## Relacionadas

- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md) §4
