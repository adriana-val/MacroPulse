# Project Charter - Entendimiento del Negocio

## Nombre del Proyecto

**MacroPulse: Rotación Sectorial con LSTM y Optimización de Portafolio Markowitz**

## Contexto y Antecedentes del Negocio

Los mercados de renta variable exhiben patrones cíclicos vinculados al ciclo macroeconómico
(expansión, contracción, recesión, recuperación). La estrategia de rotación sectorial —
sobreponderar sectores que históricamente lideran en cada fase del ciclo — es ampliamente
utilizada por gestores institucionales (e.g., Fidelity Sector Investing Framework).

Sin embargo, identificar en tiempo real la fase del ciclo y anticipar qué sectores
outperformarán en los próximos 6 meses es un problema complejo que involucra:
- Variables macroeconómicas rezagadas y no lineales (CPI, tasas, desempleo, producción industrial)
- Sentimiento de política monetaria (comunicados FOMC)
- Dinámicas de correlación cambiantes entre activos

Los modelos tradicionales de regresión asumen relaciones estáticas, subestimando la
naturaleza temporal y no-lineal de estos patrones.

---

## Objetivos del Negocio

1. **Principal:** Construir un modelo LSTM capaz de predecir el retorno relativo de 10
   activos a un horizonte de 6 meses, superando un benchmark equal-weight en términos
   de Sharpe Ratio.

2. **Secundario:** Integrar las predicciones del LSTM con un optimizador de portafolio
   Markowitz (covarianza robusta Ledoit-Wolf) para construir asignaciones óptimas con
   control de drawdown máximo.

3. **Terciario:** Incorporar señales de NLP (RoBERTa sobre comunicados FOMC) como
   features adicionales para capturar el régimen de política monetaria.

---

## Alcance del Proyecto

**Incluido:**
- Descarga y preprocesamiento de datos macro desde FRED API
  (CPI, Fed Funds Rate, rendimiento 10Y, desempleo, producción industrial, precio del petróleo)
- Descarga de precios históricos desde Yahoo Finance para los tickers:
  AMD, GOOGL, AGG, XOM, JNJ, WMT, CM, DUK, BNS, SMAX
- Clasificación algorítmica del ciclo macroeconómico (Early/Mid/Late/Recession)
- Análisis de sentimiento de comunicados FOMC con FinBERT (gtfintechlab/FOMC-RoBERTa)
- Entrenamiento de modelo LSTM para predicción de retornos a 6 meses
- Optimización de portafolio Markowitz con Ledoit-Wolf y restricción de 25% por activo
- Evaluación con métricas: Accuracy direccional, Sharpe Ratio, drawdown máximo

**Excluido:**
- Trading en tiempo real o conexión a brokers
- Activos de renta fija individuales (solo AGG como ETF agregado)
- Criptomonedas

---
## Criterios de Éxito

| Criterio | Métrica | Umbral mínimo |
|---|---|---|
| Capacidad predictiva | Accuracy direccional | > 55% |
| Capacidad predictiva | KL Divergence por Activo | < 7 |
| Capacidad predictiva | Cosine Similarity | Media >= 0.086 | 
| Rendimiento del portafolio | Sharpe Ratio (test) | > 0.8 |
| Control de riesgo | Drawdown máximo | < -15% |
| Calidad del modelo | R² en retornos | > 0.3 |

---

## Datos Disponibles

| Fuente | Tipo | Variables |
|---|---|---|
| FRED API | Macro mensual | CPI, FFR, T10Y, Desempleo, IP, WTI |
| Yahoo Finance | Precios diarios | 10 tickers (2003–2026) |
| Fed Reserve | Texto | Comunicados FOMC (sentimiento) |

---

## Metodología

Un LSTM es especialmente adecuado para este problema porque pertenece a la familia de redes neuronales recurrentes diseñadas para datos secuenciales, donde el orden temporal y la historia de las variables son fundamentales. A diferencia de modelos tradicionales (regresiones, redes densas), el LSTM captura:

Dependencias temporales: El comportamiento del mercado depende de cómo evolucionan las variables, no solo de su valor actual. Una inflación alta no tiene el mismo efecto si viene bajando que si viene subiendo. Un tono hawkish de la FED es más relevante si representa un cambio reciente frente a una tendencia estable.

Efectos rezagados: Cambios en la política monetaria afectan al mercado con retraso; la inflación impacta el consumo gradualmente. El LSTM aprende estos patrones de retraso de forma implícita, sin necesidad de especificarlos manualmente. Al trabajar con variables macro y sus rezagos, la ventana de 6 meses permite que las señales tengan mayor varianza de información y potencialmente mayor poder predictivo sobre ciclos de negocio.

Persistencia y cambios de régimen: Los mercados se mueven en fases — períodos de alta o baja volatilidad, entornos monetarios restrictivos o expansivos. El LSTM identifica patrones persistentes y detecta transiciones entre estados, clave para modelar tanto retornos como riesgo.

Interacciones no lineales: El impacto del desempleo puede depender del nivel de inflación; el efecto del Hawkish Index varía según el contexto de tasas de interés. El LSTM captura estas interacciones sin necesidad de definirlas explícitamente.

Integración de señales heterogéneas: El modelo combina datos numéricos (inflación, retornos) con datos derivados de texto (Hawkish Index), procesándolos como una secuencia conjunta donde aprende cómo estas señales evolucionan e interactúan para influir en el mercado.

En este contexto, el modelo no busca "adivinar" el mercado, sino proporcionar mejores insumos para la toma de decisiones en la construcción de portafolios — transformando un enfoque tradicional basado en promedios históricos en un sistema más dinámico, contextual y alineado con el comportamiento real de los mercados.

## Cronograma

| Etapa | Duración Estimada | Fechas |
|------|---------|-------|
| Entendimiento del negocio y carga de datos | 2 semanas | del 1 de mayo al 15 de mayo |
| Preprocesamiento, análisis exploratorio | 1 semanas | del 18 de mayo al 24 de Mayo |
| Modelamiento y extracción de características | 1 semanas | del 25 de Mayo al 31 de Mayo |
| Despliegue | 1 semanas | del 1 de junio al 7 de junio |
| Evaluación y entrega final | 1 semanas | del 8 de Junio al 15 de Junio |

## Equipo del Proyecto

- Adriana Valderrama — adrianavalderrama11@gmail.com
- Luis Felipe Tolosa — ltolosas@unal.edu.co
- Michael Betancourt — msbetancourtge@unal.edu.co

## Diccionario de Datos

El diccionario de datos describe las variables utilizadas en el proyecto MacroPulse, incluyendo su origen, tipo, descripción y uso en el modelo.

### Variables Macroeconómicas (Fuente: FRED API)

| Variable | Tipo | Descripción | Frecuencia | Uso en Modelo |
|----------|------|-------------|------------|---------------|
| CPI | Numérico (float) | Índice de Precios al Consumidor (inflación) | Mensual | Feature para predicción de retornos; indicador de ciclo económico |
| FFR | Numérico (float) | Tasa de Fondos Federales | Mensual | Indicador de política monetaria; afecta expectativas de mercado |
| T10Y | Numérico (float) | Rendimiento del Bono del Tesoro a 10 años | Mensual | Proxy de expectativas de crecimiento e inflación |
| Desempleo | Numérico (float) | Tasa de Desempleo | Mensual | Indicador de salud económica; correlacionado con ciclos |
| IP | Numérico (float) | Producción Industrial | Mensual | Medida de actividad económica real |
| WTI | Numérico (float) | Precio del Petróleo WTI | Mensual | Indicador de energía y presiones inflacionarias |

### Variables de Precios de Activos (Fuente: Yahoo Finance)

| Variable | Tipo | Descripción | Frecuencia | Uso en Modelo |
|----------|------|-------------|------------|---------------|
| Precio de Cierre (Close) | Numérico (float) | Precio de cierre ajustado por dividendos y splits | Diaria | Base para cálculo de retornos |
| Retorno Diario | Numérico (float) | Retorno porcentual diario: (Close_t - Close_{t-1}) / Close_{t-1} | Diaria | Target para predicción; cálculo de volatilidad |
| Retorno a 6 Meses | Numérico (float) | Retorno acumulado en ventana de 180 días | Diaria (agregada) | Variable objetivo principal del modelo LSTM |
| Volatilidad (Rolling Std) | Numérico (float) | Desviación estándar móvil de retornos (30 días) | Diaria | Feature de riesgo; indicador de régimen de mercado |

**Tickers incluidos:** AMD (Tecnología), GOOGL (Tecnología), AGG (Bonos), XOM (Energía), JNJ (Salud), WMT (Consumo), CM (Finanzas), DUK (Utilidades), BNS (Finanzas), SMAX (Salud).

### Variables de Texto y Sentimiento (Fuente: Fed Reserve)

| Variable | Tipo | Descripción | Frecuencia | Uso en Modelo |
|----------|------|-------------|------------|---------------|
| Texto FOMC | Texto (string) | Comunicados oficiales de la Reserva Federal | Irregular (mensual aproximado) | Input para análisis de sentimiento |
| Puntaje de Sentimiento | Numérico (float) | Puntaje hawkish/dovish generado por FinBERT (gtfintechlab/FOMC-RoBERTa) | Por comunicado | Feature adicional; captura tono de política monetaria |

### Variables Derivadas y de Modelo

| Variable | Tipo | Descripción | Origen | Uso en Modelo |
|----------|------|-------------|--------|---------------|
| Fase del Ciclo | Categórico (string) | Clasificación: Early/Mid/Late/Recession | Algoritmo basado en macro vars | Estratificación de datos; feature categórica |
| Predicción LSTM | Numérico (float) | Retorno relativo predicho a 6 meses | Modelo entrenado | Output del modelo; input para optimización |
| Pesos de Portafolio | Numérico (float) | Asignación óptima por activo (0-0.25) | Optimizador Markowitz | Resultado final; sujeto a restricciones |
| Sharpe Ratio | Numérico (float) | Ratio de retorno ajustado por riesgo | Cálculo post-modelo | Métrica de evaluación |
| Drawdown Máximo | Numérico (float) | Pérdida máxima acumulada | Simulación histórica | Métrica de riesgo |
