# Predicción del precio del Brent

Analizar si el riesgo geopolítico puede ayudar a predecir el precio promedio mensual del petróleo Brent del mes siguiente.

## Datos

- Precio mensual del Brent.
- Índices GPR, GPRT y GPRA.
- Producción, consumo, inventarios y balance mundial de petróleo.

## Estructura

- `data/raw/`: archivos originales.
- `data/processed/`: datasets limpios y unidos.
- `notebooks/01_preprocesamiento.ipynb`: limpieza, unión y exportación.
- `notebooks/02_exploracion_datos.ipynb`: exploración de Brent y GPR.
- `notebooks/03_exploracion_mercado_mundial.ipynb`: exploración del dataset integrado.

## Info importante
`data/processed/brent_gpr_market_monthly.csv` contiene datos mensuales desde enero de 1998 hasta junio de 2026, incluyendo el precio Brent del mes siguiente. Los datos a partir de julio de 2026 son predicciones del modelo STEO de la EIA, para considerar. 

*** Lo que vamos a presentar en clase es solo el notebook 01_preprocesamiento.ipynb y 02_exploracion_datos.ipynb. El notebook 03_exploracion_mercado_mundial.ipynb es una linea investigativa para que veamos si nos conviene agregar datos de producción, consumo, inventarios y balance mundial. ***

