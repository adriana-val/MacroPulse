# Reporte del Modelo Baseline

Este documento contiene los resultados del modelo baseline, un LSTM Multi-Output Global, diseñado para predecir los retornos mensuales de un portafolio de 9 activos.

## Descripción del modelo

El modelo baseline, denominado **FinancialLSTM_V2**, es una red neuronal recurrente (LSTM) que procesa datos secuenciales para capturar dependencias temporales y no lineales entre variables macroeconómicas, señales de política monetaria y retornos de activos.

**Arquitectura:**
La arquitectura del modelo es la siguiente:
`LSTM(stacked) → LayerNorm → Dropout → FC(fc_hidden) → ReLU → Dropout → FC(9)`

Se optó por un **modelo global** (un solo modelo para los 9 activos) en lugar de un modelo por activo por las siguientes razones:

- **Captura de dinámica cruzada:** La tesis del proyecto es sobre la rotación sectorial, donde las condiciones macroeconómicas afectan a todos los activos, pero de manera diferenciada. Un modelo global puede aprender estas interacciones.
- **Regularización implícita:** Con un número limitado de muestras de entrenamiento (~170), entrenar un modelo por activo aumentaría el riesgo de sobreajuste. El modelo global comparte parámetros, lo que actúa como una forma de regularización.
- **Predicciones coherentes:** Para la optimización de portafolios de Markowitz, es crucial tener predicciones de retorno que sean coherentes entre sí. Un modelo global genera estas predicciones desde una misma representación latente del estado macroeconómico.

## Variables de entrada

El modelo utiliza una ventana de 6 meses de 32 características para predecir los retornos del mes siguiente. Las variables se agrupan en tres categorías:

1.  **Datos de Mercado (9 Retornos):** Retornos históricos mensuales de los siguientes activos:
    - `AMD`, `GOOGL`, `AGG`, `XOM`, `JNJ`, `WMT`, `CM`, `DUK`, `BNS`.

2.  **Características NLP (3 variables):** Extraídas de los comunicados de la FOMC (Reserva Federal) usando FinBERT.
    - `finbert_positive`, `finbert_negative`, `hawkish_index`.

3.  **Variables Macroeconómicas (20 variables):** Obtenidas de FRED, incluyendo niveles y deltas.
    - **Niveles:** `UNRATE`, `FEDFUNDS`, `T10Y2Y`, `DGS10`, `VIXCLS`, `UMCSENT`, `DCOILWTICO`, `ICSA`.
    - **Deltas:** `d_UNRATE`, `d_FEDFUNDS`, `d_T10Y2Y`, `d_DGS10`, `d_BAA10Y`, `d_VIXCLS`, `d_UMCSENT`, `d_ICSA`.
    - **Tasas de cambio:** `Inflation`, `INDPRO_Growth`, `M2_Growth`, `DGORDER_Growth`.

## Variable objetivo

La variable objetivo del modelo son los **retornos mensuales del mes siguiente (t+1)** para cada uno de los 9 activos del portafolio. Es un problema de regresión multi-output.

## Evaluación del modelo

### Métricas de evaluación

Para evaluar el rendimiento del modelo, se utilizan métricas que evalúan tanto la magnitud del error como la dirección de la predicción:

- **Magnitud:** Mean Squared Error (MSE), Mean Absolute Error (MAE), R-cuadrado (R²).
- **Dirección:** Similitud Coseno (Cosine Similarity), Precisión Direccional (Directional Accuracy).
- **Distribución:** Divergencia de Kullback-Leibler (KL Divergence).

### Resultados de evaluación

Se realizó una búsqueda de hiperparámetros (Grid Search) con 96 configuraciones distintas. La mejor configuración encontrada fue la siguiente:

| Hiperparámetro            | Valor     |
| :------------------------ | :-------- |
| `hidden_size`             | 32        |
| `num_layers`              | 2         |
| `lr`                      | 0.001     |
| `dropout`                 | 0.25      |
| `fc_hidden`               | 32        |
| `batch_size`              | 32        |
| **Validation Loss (MSE)** | **1.077** |

## Análisis de los resultados

La búsqueda de hiperparámetros revela un patrón claro: a medida que se eliminaron características ruidosas en iteraciones previas del modelo, la capacidad necesaria del LSTM (medida por `hidden_size`) disminuyó de 128 a 32. Esto es una señal positiva, ya que un modelo más simple que logra un `Validation Loss` similar (en el rango de 1.06-1.08) indica una mejor generalización y menor riesgo de sobreajuste.

El hecho de que la mejor configuración tenga un `hidden_size` de 32 y un `fc_hidden` de 32 sugiere un equilibrio: el LSTM comprime las 32 características de entrada en una representación latente de 32 dimensiones, y la capa densa final necesita suficiente capacidad para mapear esta representación a los 9 retornos de los activos.

## Conclusiones

El modelo baseline LSTM global demuestra ser una aproximación prometedora. La arquitectura es capaz de procesar señales heterogéneas (mercado, NLP, macro) para generar predicciones de retorno. La selección de un modelo global y la optimización de hiperparámetros a través de Grid Search han llevado a una arquitectura más simple y robusta en comparación con versiones anteriores, las cuales comprendían variables extra y variables eliminadas en este modelo base, todo para evitar el sobreajuste del modelo.

Las futuras mejoras podrían centrarse en la evaluación final del modelo en el conjunto de prueba para obtener métricas definitivas de rendimiento y en la implementación del optimizador de portafolios de Markowitz utilizando las predicciones del modelo.

## Referencias

- **Datos de Mercado:** Yahoo Finance (`yfinance`).
- **Datos de Política Monetaria (FOMC):** HuggingFace (`vtasca/fomc-statements-minutes`) y scraping directo de `federalreserve.gov`.
- **Modelo NLP:** FinBERT (`ProsusAI/finbert`).
- **Datos Macroeconómicos:** Federal Reserve Economic Data (FRED) a través de `pandas-datareader`.
- **Librerías de Deep Learning:** PyTorch.
