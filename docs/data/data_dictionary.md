# Diccionario de datos

---

## Base de datos 1 — Corpus FOMC (Fuente: Federal Reserve / HuggingFace)

Comunicados oficiales de política monetaria de la Reserva Federal de EE.UU. Obtenidos de HuggingFace (`vtasca/fomc-statements-minutes`) y scraping directo de PDFs desde `federalreserve.gov`. Solo se incluyen *statements* (no *minutes*).

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `date` | Fecha de publicación del comunicado FOMC | datetime | 2000-01-01 a 2025-xx-xx | HuggingFace / FED |
| `content` | Texto completo del comunicado, limpiado (sin notas de implementación ni registros de votación) | string | Texto libre, ~200–2000 caracteres | HuggingFace / FED scraping |
| `source` | Origen del documento en el pipeline de extracción | string | `huggingface`, `fed_scraping` | Metadato del pipeline |

---

## Base de datos 2 — Features NLP (Fuente: FinBERT sobre corpus FOMC)

Variables numéricas extraídas del corpus FOMC mediante análisis de sentimiento con FinBERT (`ProsusAI/finbert`) y cálculos derivados. Frecuencia original: por comunicado (~8 veces/año). Alineadas al calendario mensual con forward fill lógico (`merge_asof`).

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `finbert_positive` | Probabilidad ponderada de sentimiento positivo (dovish) del comunicado. Peso proporcional a la "extremeness" de cada chunk | float | [0, 1] | FinBERT (`ProsusAI/finbert`) |
| `finbert_negative` | Probabilidad ponderada de sentimiento negativo (hawkish) del comunicado | float | [0, 1] | FinBERT (`ProsusAI/finbert`) |
| `finbert_neutral` | Probabilidad ponderada de sentimiento neutral del comunicado | float | [0, 1] | FinBERT (`ProsusAI/finbert`) |
| `hawkish_index` | Índice de postura monetaria: `finbert_negative − finbert_positive`. Positivo = hawkish (restrictivo), negativo = dovish (expansivo) | float | [-1, 1] | Derivado de FinBERT |
| `hawkish_delta` | Cambio mensual del `hawkish_index` (`diff()`). Captura si la FED está girando su postura | float | [-2, 2] aprox. | Derivado de `hawkish_index` |
| `hawkish_ma3` | Promedio móvil de 3 statements del `hawkish_index`. Captura la tendencia sostenida de política monetaria | float | [-1, 1] | Derivado de `hawkish_index` |
| `hawkish_vol3` | Desviación estándar rolling de 3 statements del `hawkish_index`. Alta volatilidad = incertidumbre sobre dirección monetaria | float | [0, ~1] | Derivado de `hawkish_index` |
| `hawkish_streak` | Número de comunicados consecutivos en la misma dirección. Positivo = racha hawkish, negativo = racha dovish | int | -N a +N | Derivado de `hawkish_index` |

**Nota:** En la versión final del modelo (V3), las features NLP activas son: `finbert_positive`, `finbert_negative`, `hawkish_index` (3 variables). `finbert_neutral`, `hawkish_delta`, `hawkish_ma3`, `hawkish_vol3` y `hawkish_streak` fueron evaluadas en versiones anteriores.

---

## Base de datos 3 — Precios de activos financieros (Fuente: Yahoo Finance)

Precios históricos diarios de 9 activos descargados con `yfinance`. Período: 2003-01-01 a 2024-12-31. Ajustados por dividendos y splits (`auto_adjust=True`).

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `AMD` | Precio de cierre ajustado — Advanced Micro Devices (Semiconductores / Tecnología) | float | > 0 (USD) | Yahoo Finance |
| `GOOGL` | Precio de cierre ajustado — Alphabet Inc. (Big Tech / Comunicaciones) | float | > 0 (USD) | Yahoo Finance |
| `AGG` | Precio de cierre ajustado — iShares Core US Aggregate Bond ETF (Renta Fija Agregada) | float | > 0 (USD) | Yahoo Finance |
| `XOM` | Precio de cierre ajustado — ExxonMobil Corp. (Energía / Petróleo) | float | > 0 (USD) | Yahoo Finance |
| `JNJ` | Precio de cierre ajustado — Johnson & Johnson (Salud / Farmacéutica) | float | > 0 (USD) | Yahoo Finance |
| `WMT` | Precio de cierre ajustado — Walmart Inc. (Consumo Básico / Retail) | float | > 0 (USD) | Yahoo Finance |
| `CM` | Precio de cierre ajustado — Canadian Imperial Bank of Commerce (Banca / Finanzas) | float | > 0 (CAD/USD) | Yahoo Finance |
| `DUK` | Precio de cierre ajustado — Duke Energy Corp. (Utilities / Energía Eléctrica) | float | > 0 (USD) | Yahoo Finance |
| `BNS` | Precio de cierre ajustado — Bank of Nova Scotia (Banca Canadiense / Finanzas) | float | > 0 (CAD/USD) | Yahoo Finance |
| `{TICKER}_Return` | Retorno porcentual mensual de cada activo: `pct_change()` sobre el precio de cierre del último día del mes | float | (-1, +∞) | Calculado desde Yahoo Finance |

---

## Base de datos 4 — Variables macroeconómicas (Fuente: FRED API)

Series macroeconómicas descargadas con `pandas_datareader` desde FRED. Período: 2002-01-01 a 2024-12-31. Resampleadas a frecuencia mensual (`resample("ME").last()`).

### 4.1 Variables de nivel (estado actual del ciclo — 8 variables activas en V3)

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `UNRATE` | Tasa de desempleo (%) de EE.UU. | float | [0, 100] | FRED: UNRATE |
| `FEDFUNDS` | Tasa de fondos federales efectiva (%). Instrumento principal de política monetaria | float | [0, ~22] | FRED: FEDFUNDS |
| `T10Y2Y` | Spread de la curva de rendimiento Tesoro 10Y − 2Y (%). Negativo = curva invertida (señal de recesión) | float | [-3, +3] aprox. | FRED: T10Y2Y |
| `DGS10` | Rendimiento del Bono del Tesoro a 10 años (%). Proxy del costo de capital | float | [0, ~16] | FRED: DGS10 |
| `VIXCLS` | Índice de volatilidad implícita CBOE (VIX). Indicador de incertidumbre del mercado | float | [0, ~90] | FRED: VIXCLS |
| `UMCSENT` | Índice de sentimiento del consumidor (Universidad de Michigan) | float | [0, 120] aprox. | FRED: UMCSENT |
| `DCOILWTICO` | Precio del petróleo crudo WTI (USD/barril). Indicador de costos energéticos e inflación | float | [0, ~150] | FRED: DCOILWTICO |
| `ICSA` | Solicitudes iniciales de seguro de desempleo (promedio mensual, miles). Indicador líder del mercado laboral | float | > 0 | FRED: ICSA |

**Nota:** `BAA10Y` (spread corporativo BAA − Tesoro 10Y, indicador de estrés financiero) fue incluida en modelos V1 y V2 pero eliminada en V3 por baja importancia de permutación.

### 4.2 Tasas de cambio (derivadas de series de nivel — 4 variables)

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `Inflation` | Tasa de inflación mensual: `pct_change()` del CPI (CPIAUCSL). Mide la variación del nivel de precios | float | (-0.05, +0.05) aprox. | Derivado de FRED: CPIAUCSL |
| `INDPRO_Growth` | Tasa de crecimiento mensual de la producción industrial | float | (-0.15, +0.10) aprox. | Derivado de FRED: INDPRO |
| `M2_Growth` | Tasa de crecimiento mensual de la oferta monetaria M2. Indicador de liquidez del sistema | float | (-0.05, +0.08) aprox. | Derivado de FRED: M2SL |
| `DGORDER_Growth` | Tasa de crecimiento mensual de pedidos de bienes durables. Indicador de inversión empresarial | float | (-0.30, +0.30) aprox. | Derivado de FRED: DGORDER |

### 4.3 Deltas mensuales (dirección del cambio — 8 variables activas en V3)

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `d_UNRATE` | Cambio mensual de la tasa de desempleo. Positivo = deterioro del mercado laboral | float | (-1, +2) aprox. | `diff()` de UNRATE |
| `FEDFUNDS` (delta) | Cambio mensual de la tasa de fondos federales | float | (-0.75, +0.75) aprox. | `diff()` de FEDFUNDS |
| `d_T10Y2Y` | Cambio mensual del spread 10Y−2Y. Captura aplanamiento/empinamiento de la curva | float | (-0.50, +0.50) aprox. | `diff()` de T10Y2Y |
| `d_DGS10` | Cambio mensual del rendimiento a 10 años | float | (-1, +1) aprox. | `diff()` de DGS10 |
| `d_BAA10Y` | Cambio mensual del spread corporativo. Señal de estrés financiero | float | (-1, +1) aprox. | `diff()` de BAA10Y |
| `d_VIXCLS` | Cambio mensual del VIX. Captura shocks de volatilidad | float | (-30, +40) aprox. | `diff()` de VIXCLS |
| `d_UMCSENT` | Cambio mensual del sentimiento del consumidor | float | (-20, +20) aprox. | `diff()` de UMCSENT |
| `ICSA` (delta) | Cambio mensual de solicitudes iniciales de desempleo | float | Variable | `diff()` de ICSA |

**Nota:** `d_DCOILWTICO` fue incluida en V1 y V2 pero eliminada en V3 por impacto negativo en métricas.

---

## Base de datos 5 — Dataset integrado y procesado (`df_combined`)

Dataset mensual consolidado que combina retornos de activos, features NLP y variables macroeconómicas. Período efectivo: 2003-06-01 a 2024-12-31 (~258 meses). Resultado del pipeline de alineación temporal y limpieza.

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `{TICKER}_Return` (×9) | Retorno porcentual mensual de cada uno de los 9 activos | float | (-1, +∞) | Base de datos 3 |
| `finbert_positive` | Prob. sentimiento positivo FED, forward-filled al calendario mensual | float | [0, 1] | Base de datos 2 |
| `finbert_negative` | Prob. sentimiento negativo FED, forward-filled al calendario mensual | float | [0, 1] | Base de datos 2 |
| `hawkish_index` | Índice hawkish/dovish de la FED, forward-filled | float | [-1, 1] | Base de datos 2 |
| Variables macro (×22) | 8 niveles + 4 tasas de cambio + 8 deltas + `BAA10Y` (residual) | float | Variable | Base de datos 4 |

---

## Base de datos 6 — Dataset normalizado y tensores de entrada (`df_scaled`, `X`, `y`)

Datos procesados y transformados para el entrenamiento del modelo LSTM.

| Variable | Descripción | Tipo de dato | Rango/Valores posibles | Fuente de datos |
| --- | --- | --- | --- | --- |
| `df_scaled` | Dataset mensual con todas las features escaladas a media 0 y desviación estándar 1 (StandardScaler) | DataFrame float | μ=0, σ=1 | Base de datos 5 + StandardScaler |
| `X` | Tensores de entrada del LSTM: ventanas deslizantes de 6 meses. Shape: `(n_muestras, 6, 34)` | ndarray float32 3D | Escalado | Sliding window sobre `df_scaled` |
| `y` | Retornos objetivo del mes `t+1` normalizados para los 9 activos. Shape: `(n_muestras, 9)` | ndarray float32 2D | Escalado | `df_scaled[return_cols]` desplazado 1 mes |
| `X_train` / `y_train` | Partición de entrenamiento (70% de muestras, período más antiguo) | ndarray | Escalado | Split cronológico de X, y |
| `X_val` / `y_val` | Partición de validación (15%, período intermedio) | ndarray | Escalado | Split cronológico de X, y |
| `X_test` / `y_test` | Partición de test (15%, período más reciente ~Oct 2022–Dic 2024) | ndarray | Escalado | Split cronológico de X, y |

- **Variable**: nombre de la variable o estructura de datos.
- **Descripción**: descripción de la variable y su rol en el pipeline.
- **Tipo de dato**: tipo Python/NumPy de la variable.
- **Rango/Valores posibles**: rango típico observado o rango teórico.
- **Fuente de datos**: origen primario o proceso de generación.
