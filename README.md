# Soft-sensor para digestão anaeróbia a partir de dados sintéticos do ADM1

Projeto de pesquisa (Aprendizado de Máquina) voltado à construção de um **soft-sensor** para
reatores de **digestão anaeróbia**: estimar, em tempo real, variáveis de difícil medição a
partir de medições simples, baratas e rápidas.

| Papel | Variáveis |
|---|---|
| **Medições fáceis** (entradas do soft-sensor) | pH, alcalinidade, acidez, DQO, sólidos, vazão, temperatura, produção/composição do biogás |
| **Alvos de estimativa** (saídas do soft-sensor) | concentração de AGVs (acetato, propionato, butirato, valerato), nitrogênio inorgânico (N-NH4), alcalinidade de bicarbonato, biomassa ativa por grupo trófico |

## Estratégia em duas fases

| Fase | Objetivo | Situação |
|---|---|---|
| **1. Reator virtual** | Construir um modelo robusto em **ADM1** (IWA Anaerobic Digestion Model No. 1) para síntese dos dados | em andamento |
| **2. Soft-sensor** | Estimar variáveis de difícil medição a partir de análises simples | a iniciar |

**Por que começar pelo ADM1?** O ADM1 fornece todos os estados internos do reator, o que
permite gerar dados rotulados e construir o soft-sensor em um cenário controlado, antes de
aplicá-lo a um cenário real.

## Estrutura do repositório

```text
Projeto/
├── ADM1                      # Etapa 1: simulação com influente constante
├── Dynamic_builder           # Etapa 2: gerador da série temporal de influente (CSV)
├── ADM1_Dyn                  # Etapa 3: simulação dinâmica (influente variável + pH, alcalinidade, acidez, sólidos e DQO)
├── influent_data.csv         # saída do Dynamic_builder: 241 h x 25 componentes + t + Q
├── series_dyn.csv            # saída do ADM1_Dyn: estados de inf/eff/gás + pós-processamento (grade nativa)
├── dataset_1h.csv            # idem, reamostrado em grade regular de 1 h (241 instantes)
├── plots/                    # figuras geradas pelos scripts
├── Guias metodológicos/      # documentação metodológica e histórico de decisões
│   ├── README.md             # índice, convenções e templates
│   ├── 01-modelo-adm1-e-influente-dinamico.md
│   └── decisoes/             # registros de decisão (ADR)
└── Ref metodológica/         # material bibliográfico de referência
```

## Ambiente

| Item | Versão / observação |
|---|---|
| Python | 3.13.16 |
| QSDsan | 1.5.3 (implementação do ADM1, `AnaerobicCSTR`, `DynamicInfluent`, `WasteStreamScope`) |
| Dependências | `numpy`, `pandas`, `matplotlib`, `scipy`, `chemicals` (instaladas com o QSDsan) |
| Sistema | Windows; scripts executados na raiz do projeto |

## Como executar

Os scripts não têm extensão `.py` (são arquivos de script puros); rode-os com o interpretador:

```powershell
# 1) (opcional) regenerar a série de influente — o CSV já está no repositório
python Dynamic_builder

# 2) simulação dinâmica: lê influent_data.csv e gera as figuras *_dyn.png
python ADM1_Dyn

# simulação com influente constante (independente dos passos acima)
python ADM1
```

Tempo de execução observado: ~35 s por simulação (BDF, 10 d simulados).
Em execução sem interface gráfica, defina `MPLBACKEND=Agg` para evitar bloqueio em `plt.show()`.

> **Atenção:** `ADM1_Dyn` lê o CSV por **caminho absoluto** e grava em `plots/` relativo ao
> diretório de execução. Rode a partir da raiz do projeto.

## Verificação de execução

Os dois scripts de simulação foram executados apenas para verificar a implementação:

| Verificação | Resultado |
|---|---|
| `python ADM1` | conclui sem erros (~36 s) e gera as figuras de `plots/` |
| `python ADM1_Dyn` | conclui sem erros (~35 s) e gera as figuras `*_dyn.png` |

Os parâmetros usados nesses scripts (vazão, HRT, temperatura, composição de entrada e
perturbação de vazão) são **valores de teste** para validar a implementação — não são
especificações do projeto. Os números obtidos nessas execuções não devem ser tratados como
resultados de referência.

## Documentação

- [Guia metodológico 01](<Guias metodológicos/01-modelo-adm1-e-influente-dinamico.md>) — contexto, objetivos e visão do projeto.
- [Índice dos guias e convenções](<Guias metodológicos/README.md>) — como registrar cada nova etapa.
- [Registro de decisões](<Guias metodológicos/decisoes/README.md>) — decisões já tomadas e template.

## Próximos passos

1. **Cenários**: transformar `Dynamic_builder` em gerador de cenários (Monte Carlo sobre DQO,
   composição, vazão, temperatura e pulsos de carga), com semente fixa e metadados.
2. **Warm-up**: iniciar as simulações a partir de um estado estacionário, para que o dataset
   não seja dominado pelo transiente de partida.
3. **Observáveis "fáceis"**: derivar do ADM1 pH, alcalinidade total/de bicarbonato, acidez e
   DQO — variáveis que **não** são estados do modelo e precisam de pós-processamento
   (pH, alcalinidade, acidez, sólidos e DQO já implementados no `ADM1_Dyn`).
4. **Ruído e amostragem**: simular erro analítico, frequência de amostragem e atraso de
   laboratório para que o dataset se pareça com dados reais.
5. **Dataset rotulado**: `series_dyn.csv` (grade nativa) e `dataset_1h.csv` (grade de 1 h)
   já saem com estados e observáveis; falta avaliar deduplicação das primeiras linhas e
   formato parquet.
6. **Baseline de ML**: PLS/regressão regularizada, Random Forest, Gradient Boosting; métricas
   por variável-alvo e validação em cenários não vistos no treino.

## Referências

1. Batstone, D. J. et al. *The IWA Anaerobic Digestion Model No 1 (ADM1)*. Water Science and
   Technology, 45(10), 65–73, 2002.
2. Rosen, C.; Jeppsson, U. *Aspects on ADM1 Implementation within the BSM2 Framework*. Lund
   University, 2006.
3. QSDsan — Quantitative Sustainable Design for sanitation and resource recovery systems
   (documentação e código-fonte da implementação do ADM1).
