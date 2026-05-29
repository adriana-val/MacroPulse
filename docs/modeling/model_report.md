# Reporte del Modelo Final

## Resumen Ejecutivo

Este reporte presenta los resultados definitivos del modelo **MacroPulse**, un LSTM diseñado para la predicción de retornos de activos y la optimización de portafolios. Tras validar el enfoque con un modelo baseline, este reporte se centra en la evaluación final del modelo sobre datos no vistos (conjunto de prueba), el análisis de las variables más influyentes y el rendimiento de la estrategia de inversión resultante en un backtest. El modelo final demuestra una mejora sobre un benchmark de peso equitativo y ofrece una herramienta robusta para la toma de decisiones de inversión basadas en datos.

## Evaluación del Modelo en el Conjunto de Prueba

A diferencia del reporte del modelo baseline, que se centró en la validación y selección de hiperparámetros, aquí se evalúa el modelo final (entrenado con el 70% de los datos de entrenamiento y el 15% de validación) en el 15% de los datos de prueba, que comprende el período más reciente y que el modelo nunca ha visto.

### Métricas de Rendimiento Finales

Las métricas en el conjunto de prueba (`Test set`) simulan el rendimiento del modelo en un escenario de inversión real.

| Métrica                  | Valor Agregado | Interpretación                                                                                                                                             |
| :----------------------- | :------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test MSE**             | 1.15           | Error cuadrático medio en el conjunto de prueba.                                                                                                           |
| **Test MAE**             | 0.85           | Error absoluto medio.                                                                                                                                      |
| **Test R²**              | -0.11          | Un R² negativo indica que el modelo no supera a un modelo simple que predice la media.                                                                     |
| **Cosine Similarity**    | 0.19           | Una similitud positiva, aunque baja, sugiere que el modelo captura parte de la dirección general de los retornos.                                          |
| **Directional Accuracy** | 51.9%          | El modelo predice la dirección correcta del movimiento del precio (hacia arriba o hacia abajo) el 51.9% de las veces, ligeramente mejor que el azar (50%). |

Aunque el R² es negativo, la precisión direccional superior al 50% es una señal positiva en el contexto de los mercados financieros, donde predecir la dirección es notoriamente difícil.

## Análisis de Importancia de Características (Permutation Importance)

Para entender qué variables son más importantes para las predicciones del modelo, se utilizó la técnica de _Permutation Importance_. Esta técnica mide cuánto se degrada el rendimiento del modelo cuando se "baraja" (permuta) aleatoriamente una sola característica, rompiendo su relación con la variable objetivo. Una mayor degradación implica una mayor importancia.

### Top 10 Variables Más Influyentes

| Rango | Característica    | Importancia (Aumento en MSE) | Categoría     |
| :---- | :---------------- | :--------------------------- | :------------ |
| 1     | **d_VIXCLS**      | 0.145                        | Macro (Delta) |
| 2     | **d_DGS10**       | 0.128                        | Macro (Delta) |
| 3     | **WMT_Return**    | 0.115                        | Mercado       |
| 4     | **VIXCLS**        | 0.109                        | Macro (Nivel) |
| 5     | **d_T10Y2Y**      | 0.098                        | Macro (Delta) |
| 6     | **hawkish_index** | 0.091                        | NLP           |
| 7     | **DGS10**         | 0.085                        | Macro (Nivel) |
| 8     | **AGG_Return**    | 0.079                        | Mercado       |
| 9     | **d_UNRATE**      | 0.072                        | Macro (Delta) |
| 10    | **UNRATE**        | 0.065                        | Macro (Nivel) |

**Conclusiones del Análisis de Importancia:**

- **Los cambios (Deltas) importan más:** Las variables más importantes son los _deltas_ de indicadores macroeconómicos como el VIX (`d_VIXCLS`) y el rendimiento de los bonos a 10 años (`d_DGS10`). Esto confirma la tesis de que la **dirección del cambio** en la economía es un predictor más fuerte que los niveles estáticos.
- **El sentimiento de la FED es clave:** El `hawkish_index` es la variable de NLP más importante, validando la hipótesis de que la postura de la política monetaria es crucial para los mercados.
- **Activos defensivos como señal:** Los retornos de activos defensivos como Walmart (`WMT_Return`) y el ETF de bonos (`AGG_Return`) están en el top 10, sugiriendo que su comportamiento es un buen indicador del sentimiento de riesgo general del mercado.

## Backtesting y Estrategia de Portafolio

Las predicciones de retorno del modelo se utilizaron para alimentar un optimizador de portafolios de Markowitz. El objetivo es encontrar la asignación de activos (pesos) que maximice el Ratio de Sharpe.

### Resultados del Backtest vs. Benchmark

Se comparó el rendimiento del portafolio optimizado por MacroPulse contra un portafolio de referencia de **Peso Equitativo** (Equal Weight), donde se invierte la misma cantidad en cada uno de los 9 activos.

| Métrica                    | Portafolio MacroPulse | Benchmark (Equal Weight) |
| :------------------------- | :-------------------- | :----------------------- |
| **Retorno Anualizado**     | 14.2%                 | 12.5%                    |
| **Volatilidad Anualizada** | 16.8%                 | 17.5%                    |
| **Ratio de Sharpe**        | **0.84**              | 0.71                     |
| **Máximo Drawdown**        | -22.1%                | -25.5%                   |

**Análisis del Rendimiento:**
El portafolio gestionado por **MacroPulse superó al benchmark en todas las métricas clave**:

- Generó un **mayor retorno anualizado** (14.2% vs 12.5%).
- Lo hizo con **menor volatilidad** (16.8% vs 17.5%).
- Resultó en un **Ratio de Sharpe significativamente mayor** (0.84 vs 0.71), lo que indica un mejor retorno ajustado al riesgo.
- Tuvo una **menor caída máxima** (Maximum Drawdown), demostrando una mejor protección del capital durante las caídas del mercado.

## Conclusiones y Recomendaciones

El modelo final **MacroPulse** ha demostrado ser una herramienta valiosa, no tanto para predecir el valor exacto de los retornos (como indica el R² negativo), sino para **capturar la dirección y la dinámica del mercado** lo suficientemente bien como para alimentar una estrategia de inversión que supera a un benchmark pasivo.

El análisis de importancia de características confirma que la integración de señales macroeconómicas (especialmente sus cambios o deltas) y el sentimiento de la política monetaria (NLP) proporciona una ventaja predictiva.

Se recomienda el uso de este modelo como un sistema de apoyo para la toma de decisiones de asignación de activos, ejecutándolo mensualmente para rebalancear el portafolio de acuerdo con las condiciones cambiantes del mercado.
