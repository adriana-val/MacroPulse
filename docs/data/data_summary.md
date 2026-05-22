# Reporte de Datos

Este documento contiene los resultados del análisis exploratorio de datos de MacroPulse, consolidando la información del proceso de recolección y el EDA de los datos financieros, macroeconómicos y de posición restrictiva o flexible de la FED.

## Resumen general de los datos

Para la construcción del modelo MacroPulse se integró un conjunto de datos unificado proveniente de tres fuentes principales:
* **Textos de la FED (FOMC):** Comunicados de política monetaria extraídos de corpus en HuggingFace y scraping directo en formato PDF de la Federal Reserve (2000-2024). Corresponde a alrededor de 200 comunicados.
* **Mercado Bursátil (Yahoo Finance):** Precios históricos desde 2003 hasta 2024 para 9 activos representativos de diversos sectores financieros (AMD, GOOGL, AGG, XOM, JNJ, WMT, CM, DUK, BNS).
* **Variables Macroeconómicas (FRED):** 13 series temporales base (como Tasa de desempleo, Inflación CPI, Fed Funds Rate, Rendimiento 10Y, WTI Oil) transformadas en 22 características distintas para uso predictivo (niveles, deltas y tasas de crecimiento).

- **Tamaño del dataset:** El tamaño aproximado de los conjuntos combinados ronda los 5.44 MB, sumando las bases del mercado como el corpus de los textos.
- **Frecuencia temporal:** Se consolidó una frecuencia de trabajo **mensual**, estructurando los datos en ventanas deslizantes de 6 meses.
- **Extensión de la muestra:** Se abarcan cerca de 250 meses (2003-2024), particionados cronológicamente respetando la causalidad del mercado bursátil: 70% entrenamiento (Train), 15% validación y 15% prueba (Test).

## Resumen de calidad de los datos

Durante la recolección, modelamiento preliminar de variables y el Análisis Exploratorio, se definieron estrategias clave para garantizar un dataset robusto y limpio:
- **Datos faltantes y asincronías temporales:** Existen brechas debido a cierres de mercados y a que las reuniones de la FED ocurren solamente ~8 veces al año pero necesitamos información mensual. Estas brechas fueron manejadas con *Forward Fill* (`merge_asof` con asincronía "backward"). Es decir, el mercado asume la información del "último comunicado de la FED" hasta uno nuevo, rellenando valores faltantes.
- **Limpieza de variables NLP:** Los PDFs de la FED contenían información cruda como listas de votación, líneas o artefactos nativos. Se aplicaron heurísticas y NLTK stopwords/regex para normalizarlos hacia minúscula e idoneidad antes de inyectarlos en el modelo *FinBERT*.
- **Outliers y Escalamiento:** El modelo LSTM y Markowitz son hipersensibles en convergencia ante diferencias de escala de retornos y números macroeconómicos enteros (ej. M2 o IndPro). Todo fue ajustado con escalamiento estándar `StandardScaler` después de manejar posibles outliers e imputarlos rigurosamente.

## Variable objetivo

La investigación propone un problema de regresión múltiple.
- **Variables Objetivo a estimar:** Los **retornos esperados (rentabilidad mensual) de los 9 activos** estudiados para el mes t+1. 
- **Tipo de dato:** Todas las clasificaciones objetivo son **continuas**. Se basan en cálculos de los precios de cierre diarios, convertidos al cierre de cada mes y calculando el retorno porcentual.
- **Distribución de las etiquetas:** Al simular regresiones temporales financieras, la distribución se encuentra predominantemente centrada cerca del cero (con ruido blanco relativo). No existe un problema de "desbalanceo" tradicional de clases ya que no clasificamos el comportamiento puntual por binarización (ej: sube/baja), sino mediante la dinámica de series e índices de continuidad.

## Variables individuales

El modelo usa variables numéricas y calculadas con técnicas Deep Learning para crear un *State space* completo de la economía para el agente.
Las variables explicativas adicionales se separan en 2 tipos:

- **Señales de Política Monetaria y NLP (Hawkish Index):**  
  Textos transformados en series continuas mediante FinBERT determinando el peso de probabilidad de *finbert_positive*, *finbert_negative*, y *finbert_neutral*. Destaca principalmente el **Hawkish Index** (diferencia direccional restrictiva del crédito vs expansiva de liquidez), un número continuo útil para inferir la influencia monetaria sobre la aceleración del mercado.
  
- **Series Macroeconómicas:** 
  Para captar y proveer un estado económico base se calcularon 22 features que incluyen:
  1. Niveles macro reales (ej. VIX, Precio barril Petróleo, Yield/Spread de las curvas de bonos).
  2. Variables derivadas o dinámicas, conformadas por el diferencial temporal *Delta* (variación porcentual en tasas) de las mismas series y crecimientos compuestos.

## Ranking de variables

Por medio del análisis de las matrices de correlación generadas (Pearson y Spearman), el dataset refleja atributos concretos:
- Se observa redundancia significativa y casos límite de multicolinealidad con \( |r| > 0.8 \) entre algunas variables exógenas como Yield 10Y frente al Fed Funds Rate o Inflación Core vs Índices de empleo dependientes de ciclos.
- Se espera que el entrenamiento final apoye un *Permutation Importance* para crear el veredicto final de cuáles variables dirigen los componentes de los portafolios cada mes.
- A diferencia de un ranking rígido tradicional, el modelo LSTM implícitamente regulariza e interioriza las relaciones, permitiéndole entender no singularidades fuertes inmediatas sino cómo el conjunto incide en el retorno esperado.

## Relación entre variables explicativas y variable objetivo

- **Análisis lineal pobre entre exógenas y Target:** La regresión estándar o las simples matrices de correlación (en simultáneo o el mismo mes t=0) evidenciaron relaciones y correlaciones bastante limitadas, típicamente con valores \( r < 0.15 \). Las variables de la Fed o Inflación no afectan los cierres del mes el mismo día de manera proporcional, sino que provocan movimientos en el largo y mediano plazo.
- **Relaciones móviles del tiempo (Rolling Correlations):** La relación entre sectores bursátiles (como tecnología vs bonos contra variables macro) no es un sistema lineal estático; las pruebas rodantes a 36 meses (Rolling Correlation de 3 años) demostraron cómo las respuestas cruzan de positiva correlación a negativa ante distintos choques históricos del NBER (National Bureau of Economic Research) y pánicos globales como en el 2008.
- Para remediar un mapa de dispersión no lineal débil, el uso de rezagos temporales (lags) y la ventana deslizante con secuencias históricas de 6 meses a ser procesada mediante una red **Long Short-Term Memory** es imperativo.
