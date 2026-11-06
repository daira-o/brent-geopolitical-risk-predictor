# Predicción mensual del precio del Brent con riesgo geopolítico

## Descripción

Proyecto final de Data Science para EnergIA Digital de Fundación YPF. Integra el precio mensual del petróleo Brent, el índice de riesgo geopolítico (GPR) y variables mundiales de producción y consumo para estudiar si esas señales ayudan a anticipar el nivel, la dirección y el retorno del Brent del mes siguiente.

## Pregunta de investigación

¿La incorporación de riesgo geopolítico e información sobre oferta y demanda mundial mejora la predicción mensual respecto de utilizar únicamente la historia del Brent? La pregunta se evalúa en dos problemas distintos: el nivel del precio del mes siguiente y su retorno porcentual.

## Fuentes de datos

El precio Brent proviene de la serie mensual [Europe Brent Spot Price FOB de EIA](https://www.eia.gov/dnav/pet/hist/LeafHandler.ashx?f=m&n=PET&s=RBRTE); GPR, GPRT y GPRA provienen de [Caldara e Iacoviello](https://www.matteoiacoviello.com/gpr.htm); y las series `PAPR_WORLD`, `PATC_WORLD` y `COPR_WORLD` provienen del [STEO Data Browser de EIA](https://www.eia.gov/outlooks/steo/data/browser/). Los archivos originales se conservan en `data/raw/`.

## Dataset

`data/processed/brent_gpr_market_monthly.csv` contiene 336 observaciones mensuales entre enero de 1998 y diciembre de 2025. Incluye Brent, GPR y sus componentes, producción mundial de petróleo y otros líquidos, consumo mundial, producción mundial de crudo y el balance producción-consumo. El último mes no tiene objetivo futuro porque enero de 2026 queda fuera del período elegido. El experimento de retornos recalcula su objetivo desde los precios, sin modificar este archivo.

## Preprocesamiento

El notebook 01 corrige fechas y tipos, controla duplicados y faltantes, valida unidades, transforma la tabla STEO de formato ancho a mensual y calcula el balance únicamente entre categorías compatibles (`PAPR_WORLD - PATC_WORLD`). Los datos 2026-2027 se excluyen; la historia 1998-2025 se documenta como serie revisada del vintage STEO de septiembre de 2026.

## Hallazgos del AED

El Brent muestra alta persistencia en niveles y volatilidad variable. El GPR presenta picos extraordinarios, pero su relación bivariada con el retorno del Brent del mes siguiente es débil. Producción y consumo comparten tendencias de largo plazo; el balance resume mejor los desequilibrios mensuales, aunque tampoco explica por sí solo el precio futuro. Los outliers se conservan porque representan episodios reales.

## Modelos y validación

Se comparan exactamente tres escenarios: A) historia del Brent, B) Brent + GPR y C) Brent + GPR + mercado mundial. Todas las fechas son comunes, las variables externas tienen un mes de retraso, el test corresponde a 66 orígenes entre junio de 2020 y noviembre de 2025 y la selección usa cinco folds de `TimeSeriesSplit` con pipelines.

El notebook 03 predice el nivel con Ridge, Random Forest y Gradient Boosting frente a persistencia, y complementa el análisis con clasificación. El notebook 04 reutiliza los mismos regresores, hiperparámetros, observaciones y folds para predecir:

$$r_{t+1}=100\left(\frac{P_{t+1}}{P_t}-1\right).$$

Sus baselines son retorno cero y repetición del último retorno observado. Imputación o escalado, cuando corresponden, se ajustan únicamente dentro de cada partición de entrenamiento. El test no participa en la selección.

## Resultados: nivel del precio

En regresión, Ridge del escenario A obtuvo MAE 5,014, RMSE 6,238 y R² 0,864 en test; la persistencia obtuvo MAE 4,942 y RMSE 6,243, por lo que la diferencia es despreciable y no constituye una mejora consistente. En clasificación, el mejor modelo entrenado fue Random Forest optimizado del escenario C, con accuracy 0,485, balanced accuracy 0,491, F1 0,370 y ROC-AUC 0,555; no superó el baseline de balanced accuracy 0,500.

El R² alto del nivel refleja en gran medida la persistencia temporal: el modelo aprendido no mejora de manera consistente al baseline aun cuando ambos explican gran parte de la variación del nivel.

## Resultados: retornos mensuales

La configuración elegida solo por validación temporal fue Random Forest en C. Su RMSE CV fue 11,805 puntos porcentuales, peor que 10,119 del baseline de retorno cero. En el test histórico obtuvo MAE 6,580, RMSE 7,828 y R² −0,075; retorno cero obtuvo MAE 6,277, RMSE 7,608 y R² −0,016. Repetir el último retorno fue aún peor: RMSE 10,391 y R² −0,895.

Con Random Forest fijo para aislar el efecto de las variables, GPR redujo el RMSE CV de A a B en 0,351 puntos y el mercado redujo B a C en 0,121. Esa mejora pequeña no venció al baseline y no fue estable en test: B empeoró A en 0,525 puntos, mientras C mejoró B en 0,566. Por tanto, no hay evidencia robusta de aporte predictivo adicional. Un R² negativo es esperable cuando un modelo de retornos rinde peor que una referencia simple; no implica un error de implementación.

Los MAE y RMSE de precios están en USD/barril y los de retornos en puntos porcentuales, por lo que no se comparan directamente.

## Período 2020–2022

El test contiene 30 meses objetivo entre julio de 2020 y diciembre de 2022 y 36 meses posteriores. Para el modelo de retornos seleccionado, el RMSE fue 9,557 puntos porcentuales en 2020–2022 frente a 6,019 en el resto; retorno cero obtuvo 9,515 y 5,538, respectivamente. En nivel, Ridge A obtuvo RMSE 7,820 USD/barril en 2020–2022 y 4,513 después.

La ventana incluye la recuperación posterior al shock inicial de la pandemia y las perturbaciones energéticas de 2022 asociadas con la invasión rusa de Ucrania, pero la comparación es descriptiva: no identifica causalidad ni atribuye todos los errores a esos acontecimientos. La cantidad de observaciones también limita la generalización.

## Cuarta preentrega: clustering

El notebook `05_clustering.ipynb` estudia si las condiciones mensuales del mercado forman grupos descriptivos sin utilizar etiquetas ni objetivos futuros. La muestra contiene 330 meses entre julio de 1998 y diciembre de 2025; los primeros seis meses se excluyen porque todavía no existe una ventana completa para calcular volatilidad histórica.

Las variables son el precio contemporáneo del Brent, su retorno mensual, la volatilidad móvil de seis meses, `log1p(GPR)` y el balance mundial producción-consumo. GPRT y GPRA se excluyen por su alta redundancia con GPR; producción, consumo y producción de crudo tampoco se incorporan simultáneamente porque comparten una fuerte tendencia. Los outliers se conservan como episodios reales, la transformación logarítmica reduce la asimetría de GPR de 4,061 a 1,039 y `StandardScaler` pone todas las variables en una escala comparable.

Se adaptaron las implementaciones de K-Means, codo, silhouette, estandarización y PCA de los materiales del curso. Se evaluaron K=2 a K=8 con `k-means++`, `n_init=50`, `max_iter=300` y `random_state=42`. K=5 obtuvo el mayor silhouette, 0,234, pero creó un grupo de solo 5 meses. Se eligió K=3 como compromiso entre métricas, tamaño e interpretación: silhouette 0,2008, Davies-Bouldin 1,6012 y grupos de 78, 114 y 138 meses. Las cinco semillas verificadas produjeron la misma partición (ARI=1).

| Grupo | Meses | Perfil promedio |
|---|---:|---|
| 0 | 78 | Brent USD 35,72; retorno 6,39%; volatilidad 13,31%; GPR 78,33; balance −0,72. |
| 1 | 114 | Brent USD 53,55; retorno −4,37%; volatilidad 8,23%; GPR 141,26; balance 0,69. |
| 2 | 138 | Brent USD 88,63; retorno 2,42%; volatilidad 5,95%; GPR 92,87; balance −0,77. |

La proyección PCA en dos dimensiones explica 55,24% de la varianza y muestra superposición entre grupos. Hubo 77 cambios en 329 transiciones mensuales; durante 2020–2022 los 36 meses se repartieron entre los tres grupos (10, 11 y 15), por lo que ese período no constituye un régimen único.

El clustering complementa los modelos supervisados al resumir combinaciones históricas recurrentes, pero no demuestra capacidad predictiva, independencia del efecto de GPR ni causalidad. El silhouette bajo, la ausencia de un codo marcado, la sensibilidad de K-Means a outliers y sus supuestos de geometría aproximadamente esférica limitan la interpretación. DBSCAN se revisó en el material del curso, pero no se agregó porque no había evidencia suficiente para fijar sus parámetros de densidad de manera defendible.

## Conclusión

Con este diseño y período no aparece evidencia robusta de que agregar GPR u oferta y demanda mejore la predicción mensual. En nivel, la persistencia explica el R² elevado y compite con Ridge A. En retornos, todos los escenarios quedan detrás del baseline cero y las pequeñas mejoras internas del bosque cambian entre validación y test. El resultado negativo es informativo: las variables adicionales no aportan una señal incremental estable bajo una evaluación temporal comparable.

## Limitaciones

El test fue observado durante el desarrollo y se interpreta como evaluación histórica, no como holdout completamente intacto. STEO contiene historia revisada y el rezago de un mes aproxima la disponibilidad real sin reconstruir calendarios históricos de publicación. La frecuencia mensual deja una muestra pequeña, especialmente para modelos flexibles y para separar 2020–2022. SHAP y LIME se conservan como complementos del experimento de nivel y no sustituyen la comparación contra baselines.

## Estructura

```text
data/raw/                                archivos originales
data/processed/brent_gpr_market_monthly.csv
notebooks/01_preprocesamiento.ipynb      preparación e integración
notebooks/02_exploracion_datos.ipynb     análisis exploratorio
notebooks/03_modelado_supervisado.ipynb  regresión y clasificación
notebooks/04_prediccion_retornos.ipynb   retornos mensuales y análisis 2020-2022
notebooks/05_clustering.ipynb             regímenes descriptivos mediante K-Means
docs/                                    material local del curso, ignorado por Git
```

## Ejecución

Crear y activar un entorno virtual, instalar `requirements.txt` y abrir los notebooks con ese kernel. Ejecutar desde la raíz del repositorio en el orden `01 → 02 → 03 → 04 → 05`; el notebook 01 reconstruye el CSV procesado y los siguientes lo consumen. Los notebooks 04 y 05 son independientes del 03 y pueden ejecutarse directamente si el CSV procesado ya existe.

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m ipykernel install --user --name brent-gpr --display-name "Brent GPR"
```
