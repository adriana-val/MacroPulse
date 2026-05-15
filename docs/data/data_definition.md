# Definición de los datos

## Origen de los datos

El proyecto **MacroPulse** integra datos de tres fuentes externas:

1. **FRED API** (Federal Reserve Economic Data): Series macroeconómicas descargadas con `pandas_datareader`. Incluye 13 series temporales (UNRATE, CPIAUCSL, INDPRO, UMCSENT, DGORDER, ICSA, FEDFUNDS, T10Y2Y, DGS10, BAA10Y, VIXCLS, M2SL, DCOILWTICO). Período: 2002-01-01 a 2024-12-31.

2. **Yahoo Finance**: Precios históricos diarios de 9 activos de renta variable descargados con `yfinance` (AMD, GOOGL, AGG, XOM, JNJ, WMT, CM, DUK, BNS). Período: 2003-01-01 a 2024-12-31. Precios ajustados por dividendos y splits.

3. **Comunicados FOMC** (Reserva Federal de EE.UU.): Obtenidos de dos fuentes complementarias:
   - **HuggingFace** (`vtasca/fomc-statements-minutes`): corpus histórico limpio de statements (2000–2025).
   - **Scraping directo de la FED**: descarga de PDFs desde `federalreserve.gov` para cobertura de actualizaciones mensuales recientes.

---

## Especificación de los scripts para la carga de datos

El pipeline completo de extracción y procesamiento se ejecuta desde el notebook principal:

```
scripts/data_acquisition/Extraccion de datos.ipynb
```

| Celda | Tipo | Operación |
|-------|------|-----------|
| 9 | Código | Instalación de dependencias (`yfinance`, `transformers`, `pandas-datareader`, `datasets`, `pypdf`) |
| 10 | Código | Importación de librerías y configuración de semilla aleatoria (`SEED=42`) |
| 12 | Código | Descarga del corpus FOMC desde HuggingFace + scraping directo de PDFs de la FED; limpieza de texto |
| 14 | Código | Análisis de sentimiento con FinBERT (`ProsusAI/finbert`) sobre chunks de 450 caracteres; cálculo del `hawkish_index` |
| 15 | Código | Generación de features NLP derivadas: `hawkish_delta`, `hawkish_ma3`, `hawkish_vol3`, `hawkish_streak` |
| 17 | Código | Descarga de precios históricos desde Yahoo Finance para los 9 tickers del portafolio |
| 20 | Código | Descarga de 13 series macroeconómicas desde FRED; cálculo de tasas de cambio y deltas mensuales |
| 22–23 | Código | Alineación temporal con `merge_asof`, resampleo mensual, join de retornos + NLP + macro |
| 25 | Código | Normalización con `StandardScaler`; construcción de ventanas deslizantes de 6 meses |
| 29 | Código | Split cronológico: Train (70%) / Validación (15%) / Test (15%) |

**Dependencias principales:** `numpy`, `pandas`, `torch`, `sklearn`, `yfinance`, `pandas_datareader`, `transformers`, `datasets`, `pypdf`, `beautifulsoup4`, `requests`, `matplotlib`.

---

## Referencias a rutas o bases de datos origen y destino

### Rutas de origen de datos

| Fuente | URL / Identificador | Período | Formato |
|--------|---------------------|---------|---------|
| FRED API | `pandas_datareader.data.DataReader(series, "fred", ...)` | 2002–2024 | Series temporal mensual |
| Yahoo Finance | `yfinance.download(ticker, ...)` — tickers: AMD, GOOGL, AGG, XOM, JNJ, WMT, CM, DUK, BNS | 2003–2024 | DataFrame diario |
| HuggingFace FOMC | `load_dataset("vtasca/fomc-statements-minutes", split="train")` | 2000–2025 | Dataset parquet |
| FED Scraping | `https://www.federalreserve.gov/monetarypolicy/fomccalendars.htm` | Histórico | PDFs |

### Estructura de los archivos de origen

- **FRED**: 13 series con frecuencias mixtas (diaria, semanal, mensual); se alinean a frecuencia mensual (`resample("ME").last()`) con forward fill para valores faltantes.
- **Yahoo Finance**: Precios de cierre diarios ajustados (`auto_adjust=True`); se resamplea al último día hábil de cada mes.
- **FOMC**: Texto libre de comunicados (statements); se eliminan las secciones de *implementation notes* y *voting records* antes del análisis de sentimiento.

### Procedimientos de transformación y limpieza

**1. FOMC / NLP:**
- Limpieza de texto: eliminación de notas de implementación y registros de votación mediante búsqueda de palabras clave.
- Análisis de sentimiento: FinBERT (`ProsusAI/finbert`) aplicado sobre chunks de máximo 450 caracteres; promedio ponderado por "extremeness" (`|neg - pos|`).
- Cálculo del Hawkish Index: `hawkish_index = finbert_negative − finbert_positive`.
- Features derivadas: delta mensual (`diff()`), promedio móvil de 3 statements (`rolling(3).mean()`), volatilidad de postura (`rolling(3).std()`), racha consecutiva de dirección.
- Alineación al calendario mensual: `merge_asof` con `direction="backward"` (forward fill lógico — el mercado asume la última postura disponible de la FED).

**2. Precios de mercado:**
- Resampleo mensual: último precio de cierre del mes (`resample("ME").last()`).
- Retorno mensual: `pct_change()` por activo sobre precio de cierre mensual.
- Manejo de NaN: eliminación de filas donde todos los retornos son nulos; forward fill en valores individuales faltantes.

**3. Variables macroeconómicas:**
- Resampleo mensual con `resample("ME").last()` y forward fill.
- Tasas de cambio (`pct_change()`): `Inflation` (CPI), `INDPRO_Growth`, `M2_Growth`, `DGORDER_Growth`.
- Deltas mensuales (`diff()`): UNRATE, FEDFUNDS, T10Y2Y, DGS10, BAA10Y, VIXCLS, UMCSENT, DCOILWTICO, ICSA.

**4. Dataset integrado (`df_combined`):**
- Join secuencial: precios mensuales → NLP (merge_asof) → macro (join por índice fecha).
- Filtro de inicio: fechas ≥ 2003-06-01 para garantizar solapamiento entre todas las fuentes.
- Imputación final: forward fill general, eliminación de filas con NaN residuales.
- Resultado: ~258 meses × (9 retornos + 3 NLP + 22 macro) = 34 features.

**5. Normalización y ventanas deslizantes:**
- `StandardScaler` ajustado sobre las 34 features numéricas del dataset completo.
- Ventana temporal de 6 meses: genera tensores 3D con forma `(muestras, 6, 34)`.
- Variable objetivo (`y`): retornos del mes `t+1` normalizados para los 9 activos.

**6. Split cronológico:**
- Train: primero 70% de las muestras.
- Validación: siguientes 15%.
- Test: últimas 15% (período más reciente, ~Oct 2022 – Dic 2024).

### Base de datos de destino

Los datos procesados se mantienen en memoria durante la ejecución del notebook. Las estructuras principales generadas son:

| Variable | Tipo | Descripción |
|----------|------|-------------|
| `df_fed` | DataFrame | Corpus FOMC con texto limpio y features NLP (hawkish_index y derivados) |
| `df_prices` | DataFrame | Precios diarios de los 9 activos (Yahoo Finance) |
| `df_macro_final` | DataFrame | 22 variables macroeconómicas en frecuencia mensual |
| `df_combined` | DataFrame | Dataset mensual integrado: retornos + NLP + macro (~258 meses × 34 features) |
| `df_scaled` | DataFrame | Dataset normalizado (StandardScaler, μ=0, σ=1) |
| `X` | ndarray 3D | Ventanas de entrada para el LSTM: `(n_muestras, 6, 34)` |
| `y` | ndarray 2D | Retornos objetivo normalizados: `(n_muestras, 9)` |
| `X_train / X_val / X_test` | ndarray | Particiones cronológicas de X |
| `y_train / y_val / y_test` | ndarray | Particiones cronológicas de y |
