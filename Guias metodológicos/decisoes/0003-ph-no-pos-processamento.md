# 0003 — pH calculado no pós-processamento

**Status:** aceita · **Data:** 21/11/2026 · **Etapa:** 01

## Contexto

O pH é uma das variáveis mais importantes do soft-sensor (é medido em linha, barato e
confiável) e também um indicador-chave de estabilidade do reator. Porém, no ADM1 o pH **não
é uma variável de estado**: ele resulta de um equilíbrio ácido-base/balanço de cargas
resolvido algebricamente a cada avaliação do lado direito das EDOs. O `WasteStreamScope`
registra apenas os estados, então o pH não aparece automaticamente nos resultados.

## Decisão

Calcular o pH **após a simulação**, a partir da série registrada do efluente, replicando
exatamente o cálculo interno do modelo:

```python
adm1_params = adm1.rate_function.params
Ka = adm1_params['Ka_base'] * T_correction_factor(
    adm1_params['T_base'], Temp, adm1_params['Ka_dH'])
unit_conversion = mass2mol_conversion(cmps)
effluent_states = eff.scope.record[:, :-1] * 1e-3        # mg/L → kg/m³, sem a coluna Q
pH = np.array([-np.log10(adm1.solve_pH(state, Ka, unit_conversion))
               for state in effluent_states])
```

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Adicionar o pH como estado extra (ODE) | altera o modelo; o equilíbrio é instantâneo, não diferencial — introduziria rigidez artificial |
| Ler o pH de dentro do `_rhos_adm1` durante a integração | exigiria instrumentar o solver; mais frágil e mais lento |
| Usar `pH_ctrl` do `AnaerobicCSTR` | serve para **controlar** pH (dosagem de base), não para registrar a série |
| Aproximação empírica (ex.: relação com AGVs/alcalinidade) | perde a consistência termodinâmica e não serviria de rótulo "verdadeiro" |

## Consequências

- (+) pH consistente com o modelo, calculado com as mesmas constantes corrigidas por
  temperatura (van't Hoff) usadas internamente.
- (+) Nenhuma alteração no modelo ou no integrador; o cálculo é O(n) sobre a série.
- (−) Depende de três conversões em sequência (mg/L → kg/m³ → mol/L), cada uma com sua
  armadilha: esquecer `*1e-3`, incluir a coluna `Q` no estado ou usar `Ka` sem correção de
  temperatura produz pH errado **sem erro aparente**.
- (−) A série do `Scope` pode não coincidir com a grade de `t_eval` (em execuções
  verificadas, há menos registros do que instantes solicitados); o pH herda esse eixo de
  tempo e precisa ser reamostrado junto com as demais variáveis.
- Obrigação: reutilizar essa mesma rotina ao gerar os observáveis "fáceis" (alcalinidade,
  acidez), que também são derivados ácido-base e não estados.

## Evidências

- `python ADM1_Dyn` executa sem erros e gera `reactor_pH_dyn.png` a partir da série do
  efluente.
- A rotina usa as mesmas constantes ácido-base e o mesmo solver do modelo, aplicados à série
  registrada em mg/L.

## Relacionadas

- Guia: [../01-modelo-adm1-e-influente-dinamico.md](../01-modelo-adm1-e-influente-dinamico.md) §5
