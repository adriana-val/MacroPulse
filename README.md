# Proyecto MacroPulse

Este es el repositorio del proyecto MacroPulse desarrollado para el diplomado "Machine Learning & Data Science Avanzado" de la Universidad Nacional de Colombia.

## Resumen del Proyecto

MacroPulse es un sistema de predicción analítica que utiliza una red neuronal recurrente (LSTM multi-output) para la optimización de portafolios de inversión. Su objetivo es predecir los retornos mensuales futuros de activos financieros, combinando dinámicamente tres fuentes principales de información:
1. **Datos de Mercado:** Retornos históricos (Yahoo Finance).
2. **Ciclos Macroeconómicos:** Series y variables económicas reales e históricas en niveles y deltas (FRED).
3. **Sentimiento de la Política Monetaria:** Un "Hawkish Index", elaborado a partir de los comunicados del FOMC de la Reserva Federal aplicando NLP con modelos preentrenados FinBERT.

Estas proyecciones se inyectan a un optimizador de Markowitz para asignar capital eficientemente, ajustándose al contexto económico.

## Estructura del Repositorio

- `docs/`: Documentación estructurada de todas las fases de la metodología del proyecto.
  - `business_understanding/`: Contexto del negocio y estatutos de definición (project charter).
  - `data/`: Información de los conjuntos de datos, diccionarios y reportes consolidados del análisis exploratorio (EDA).
  - `deployment/`: Plan y guías de despliegue lógico del modelo financero.
  - `modeling/`: Detalle sobre diseño de arquitectura, métricas subyacentes, reportes del modelo y modelo base (baselines).
- `scripts/`: Código fuente, rutinas de Python y Jupyter Notebooks utilizados en el desarrollo analítico.
  - `data_acquisition/`: Lógica de Scraping, integración a las APIs financieras (yf, fred) y recolección general del corpus.
  - `eda/`: Exploración a profundidad de los predictores, y limpieza estadística.
  - `preprocessing/`: Transformación (MinMax/Standard Scaler, Sliding Windows), imputación, rezagos aplicados en series temporales.
  - `training/`: Configuración, compilación y rutinas de optimización y entrenamiento completo de la red LSTM.
  - `evaluation/`: Procedimientos de testeo y validación backtesting (portafolio equal-weight vs optimizado).
- `pyproject.toml`: Archivo con la definición del proyecto y dependencias de Python utilizadas.
