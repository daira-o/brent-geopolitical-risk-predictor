# Predicción mensual del precio del Brent con riesgo geopolítico

## Descripción

Proyecto final de Data Science para EnergIA Digital de Fundación YPF. Se eligió el Brent porque es una referencia internacional del precio del petróleo, está expuesto a perturbaciones geopolíticas y cuenta con una serie mensual extensa y pública. El proyecto integra su precio con el índice de riesgo geopolítico (GPR) y variables mundiales de producción y consumo para estudiar el nivel, la dirección y el retorno del mes siguiente.

## Pregunta de investigación

¿La incorporación de riesgo geopolítico e información sobre oferta y demanda mundial mejora la predicción mensual del Brent respecto de utilizar únicamente su propia historia? La pregunta se evalúa en dos problemas diferentes: el nivel del precio del mes siguiente y su retorno porcentual.

## Fuentes de datos

El precio proviene de la serie mensual [Europe Brent Spot Price FOB de EIA](https://www.eia.gov/dnav/pet/hist/LeafHandler.ashx?f=m&n=PET&s=RBRTE); GPR, GPRT y GPRA provienen de [Caldara e Iacoviello](https://www.matteoiacoviello.com/gpr.htm); y `PAPR_WORLD`, `PATC_WORLD` y `COPR_WORLD` provienen del [STEO Data Browser de EIA](https://www.eia.gov/outlooks/steo/data/browser/). Los archivos originales se conservan sin modificaciones en `data/raw/`.

## Dataset y preprocesamiento

`data/processed/brent_gpr_market_monthly.csv` contiene 336 observaciones entre enero de 1998 y diciembre de 2025: Brent, GPR y sus componentes, producción mundial de petróleo y otros líquidos, consumo mundial, producción mundial de crudo y balance producción-consumo. El notebook 01 corrige fechas y tipos, controla duplicados y faltantes, valida unidades, transforma STEO a frecuencia mensual y calcula el balance entre categorías compatibles (`PAPR_WORLD - PATC_WORLD`); el último mes queda sin objetivo futuro porque enero de 2026 está fuera del período analizado.

## Hallazgos del análisis exploratorio

El Brent presenta alta persistencia en niveles y volatilidad variable, mientras que GPR contiene picos extraordinarios pero mantiene una relación bivariada débil con el retorno del Brent del mes siguiente. Producción y consumo comparten tendencias de largo plazo; el balance resume sus desequilibrios mensuales, aunque tampoco explica por sí solo el precio futuro. Los valores extremos se conservan porque representan episodios reales y las asociaciones se interpretan de forma descriptiva, sin atribuir causalidad.

## Escenarios, modelos y validación

Los experimentos comparan tres escenarios sobre las mismas fechas: A) historia del Brent, B) Brent + GPR y C) Brent + GPR + mercado mundial. Las variables externas ingresan con un mes de rezago; la selección utiliza cinco folds expansivos de `TimeSeriesSplit` sobre el 80 % inicial y el 20 % final se reserva para evaluación histórica. El notebook 03 compara Ridge, Random Forest y Gradient Boosting frente a persistencia para el nivel, y Logistic Regression y Random Forest frente a la clase mayoritaria para la dirección; el notebook 04 reutiliza las fechas, modelos e hiperparámetros para retornos, con retorno cero y último retorno como baselines. El test no participa en la selección.

## Resultados supervisados

En regresión de nivel, Ridge-A obtuvo MAE 5,014, RMSE 6,238 y R² 0,864 en test, frente a MAE 4,942 y RMSE 6,243 de la persistencia; la diferencia es despreciable y no representa una mejora consistente. En clasificación, Random Forest-B fue seleccionado previamente por validación temporal con balanced accuracy CV 0,532, pero obtuvo 0,456 en test, por debajo del baseline 0,500. Random Forest-C alcanzó ROC-AUC 0,555 en test, resultado que se informa como diagnóstico y no permite reemplazar retrospectivamente al escenario B seleccionado mediante validación.

## Resultados de retornos

Random Forest-C fue la configuración elegida solo por validación temporal: obtuvo RMSE CV 11,805 puntos porcentuales frente a 10,119 del baseline cero. En test logró MAE 6,580, RMSE 7,828 y R² −0,075; retorno cero obtuvo MAE 6,277, RMSE 7,608 y R² −0,016, y repetir el último retorno obtuvo RMSE 10,391 y R² −0,895. Con Random Forest fijo, las pequeñas mejoras de A a B y de B a C en validación no fueron estables en test; por lo tanto, no hay evidencia robusta de aporte predictivo adicional. Las métricas de nivel están en USD/barril y las de retornos en puntos porcentuales, por lo que no se comparan directamente.

## Análisis de 2020–2022

El test incluye 30 meses objetivo entre julio de 2020 y diciembre de 2022 y 36 posteriores. En retornos, el modelo seleccionado obtuvo RMSE 9,557 durante 2020–2022 y 6,019 después, frente a 9,515 y 5,538 del baseline cero; en nivel, Ridge-A obtuvo 7,820 USD/barril durante ese período y 4,513 después. La comparación es descriptiva y no atribuye causalidad a la pandemia ni a las perturbaciones energéticas de 2022.

## Clustering

El notebook 05 aplica K-Means sobre 330 meses con precio contemporáneo, retorno mensual, volatilidad móvil de seis meses, `log1p(GPR)` y balance mundial, sin usar objetivos futuros. Se evaluó K=2 a K=8 con `k-means++`, `n_init=50`, `max_iter=300` y `random_state=42`: K=5 obtuvo el mayor silhouette (0,234) pero creó un grupo de solo 5 meses, por lo que se eligió K=3 como compromiso entre separación, tamaño e interpretación. La solución final obtuvo silhouette 0,2008, Davies–Bouldin 1,6012, grupos de 78, 114 y 138 meses y estabilidad ARI=1 en cinco semillas; PCA explicó 55,24 % de la varianza y mostró superposición. El resultado resume combinaciones históricas recurrentes, pero no demuestra predicción, causalidad ni regímenes completamente separados.

## Conclusión

Con este diseño y período no aparece evidencia robusta de que agregar GPR u oferta y demanda mejore la predicción mensual frente a baselines simples. En nivel, la persistencia explica gran parte del desempeño; en dirección, el modelo seleccionado queda debajo de la clase mayoritaria; y en retornos, todos los escenarios quedan detrás del retorno cero. El resultado negativo es informativo: las variables adicionales no aportan una señal incremental estable bajo una evaluación temporal comparable.

## Limitaciones

El test ya fue inspeccionado y se interpreta como evaluación histórica, no como holdout completamente intacto. STEO contiene historia revisada de la edición de septiembre de 2026, por lo que no reproduce exactamente la información disponible en cada fecha; el rezago de un mes aproxima la publicación real sin reconstruir calendarios históricos. La frecuencia mensual deja una muestra pequeña, especialmente para modelos flexibles y para separar 2020–2022. SHAP y LIME son diagnósticos del experimento de nivel y no sustituyen la comparación contra baselines.

## Estructura publicada

```text
README.md
requirements.txt
data/
├── raw/
│   ├── 3a._International_Petroleum_and_Other_Liquids_Production_Consumption_and_Inventories.csv
│   ├── brent_eia.xls
│   └── gpr_monthly.xls
└── processed/
    └── brent_gpr_market_monthly.csv
notebooks/
├── 01_preprocesamiento.ipynb
├── 02_exploracion_datos.ipynb
├── 03_modelado_supervisado.ipynb
├── 04_prediccion_retornos.ipynb
└── 05_clustering.ipynb
```

## Reproducibilidad

El proyecto fue validado con **Python 3.12.14** y las dependencias resueltas desde `requirements.txt`. Para reproducirlo, crear un entorno virtual y ejecutar los notebooks desde la raíz en el orden `01 → 02 → 03 → 04 → 05`; el notebook 01 reconstruye el CSV procesado y los demás lo consumen. Los notebooks 04 y 05 son independientes del 03 si el CSV ya existe.

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m ipykernel install --user --name brent-gpr --display-name "Brent GPR"
```
