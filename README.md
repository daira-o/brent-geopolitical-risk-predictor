# Predicción mensual del precio del Brent con riesgo geopolítico

## Descripción

Proyecto de Data Science para la 3er pre-entrega de EnergIA Digital. Integra el precio mensual del petróleo Brent, el índice de riesgo geopolítico (GPR) y variables mundiales de producción y consumo para estudiar si esas señales ayudan a anticipar el precio y la dirección del Brent del mes siguiente.

## Pregunta de investigación

¿La incorporación de riesgo geopolítico e información sobre oferta y demanda mundial permite mejorar la predicción mensual del precio del Brent respecto de utilizar únicamente sus precios históricos?

## Fuentes de datos

El precio Brent proviene de la serie mensual [Europe Brent Spot Price FOB de EIA](https://www.eia.gov/dnav/pet/hist/LeafHandler.ashx?f=m&n=PET&s=RBRTE); GPR, GPRT y GPRA provienen de [Caldara e Iacoviello](https://www.matteoiacoviello.com/gpr.htm); y las series `PAPR_WORLD`, `PATC_WORLD` y `COPR_WORLD` provienen del [STEO Data Browser de EIA](https://www.eia.gov/outlooks/steo/data/browser/). Los archivos originales se conservan en `data/raw/`.

## Dataset

`data/processed/brent_gpr_market_monthly.csv` contiene 336 observaciones mensuales entre enero de 1998 y diciembre de 2025. Incluye Brent, GPR y sus componentes, producción mundial de petróleo y otros líquidos, consumo mundial, producción mundial de crudo y el balance producción-consumo. El último mes no tiene objetivo futuro porque enero de 2026 queda fuera del período elegido.

## Preprocesamiento

El notebook 01 corrige fechas y tipos, controla duplicados y faltantes, valida unidades, transforma la tabla STEO de formato ancho a mensual y calcula el balance únicamente entre categorías compatibles (`PAPR_WORLD - PATC_WORLD`). Los datos 2026-2027 se excluyen; la historia 1998-2025 se documenta como serie revisada del vintage STEO de septiembre de 2026.

## Hallazgos del AED

El Brent muestra alta persistencia en niveles y volatilidad variable. El GPR presenta picos extraordinarios, pero su relación bivariada con el retorno del Brent del mes siguiente es débil. Producción y consumo comparten tendencias de largo plazo; el balance resume mejor los desequilibrios mensuales, aunque tampoco explica por sí solo el precio futuro. Los outliers se conservan porque representan episodios reales.

## Modelos y validación

Se comparan exactamente tres escenarios: A) historia del Brent, B) Brent + GPR y C) Brent + GPR + mercado mundial. La regresión evalúa Ridge, Random Forest y Gradient Boosting frente a persistencia; la clasificación evalúa regresión logística y Random Forest frente a la clase mayoritaria. Todas las fechas son comunes, las variables externas tienen un mes de retraso, el test es el tramo cronológico junio de 2020-noviembre de 2025 y la selección usa cinco folds de `TimeSeriesSplit` con pipelines.

## Resultados

En regresión, Ridge del escenario A obtuvo MAE 5,014, RMSE 6,238 y R² 0,864 en test; la persistencia obtuvo MAE 4,942 y RMSE 6,243, por lo que la diferencia es despreciable y no constituye una mejora consistente. En clasificación, el mejor modelo entrenado fue Random Forest optimizado del escenario C, con accuracy 0,485, balanced accuracy 0,491, F1 0,370 y ROC-AUC 0,555; no superó el baseline de balanced accuracy 0,500.

## Conclusión

Con este diseño y período no aparece evidencia robusta de que agregar GPR u oferta y demanda mejore la predicción mensual. GPR mejora algunas comparaciones internas del mismo Random Forest, y el escenario C logra el mayor ROC-AUC, pero esas diferencias no vencen de forma estable a los baselines y los modelos de árboles muestran brechas de overfitting. El resultado negativo es coherente con el AED y señala que el precio mensual depende de más factores y regímenes de mercado.

## Estructura

```text
data/raw/                                archivos originales
data/processed/brent_gpr_market_monthly.csv
notebooks/01_preprocesamiento.ipynb      preparación e integración
notebooks/02_exploracion_datos.ipynb     análisis exploratorio
notebooks/03_modelado_supervisado.ipynb  regresión y clasificación
docs/                                    material local del curso, ignorado por Git
```

## Ejecución

Crear y activar un entorno virtual, instalar `requirements.txt` y abrir los notebooks con ese kernel. Ejecutar desde la raíz del repositorio en el orden `01 → 02 → 03`; el notebook 01 reconstruye el CSV procesado y los siguientes lo consumen.

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m ipykernel install --user --name brent-gpr --display-name "Brent GPR"
```
