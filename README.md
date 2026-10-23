# Industrial Machine Failure Classification — AI4I 2020

**Proyecto 01 | Applied Machine Learning & Industrial Analytics**

## Objetivo
Clasificar registros sintéticos como condición normal o con falla utilizando variables operativas. **No se predice el tiempo hasta la falla ni una alerta anticipada**.

## Datos
Réplica AI4I 2020 de Kaggle: https://www.kaggle.com/datasets/shivamb/machine-predictive-maintenance-classification

El análisis original utiliza **`Failure Type != "No Failure"`** como etiqueta binaria (348 positivos de 10.000). La columna `Target` del CSV presenta distinta frecuencia (~339 positivos), por lo que es imprescindible especificar cuál se emplea y verificar la diferencia antes de comparar otros trabajos.

Entradas: tipo de producto, temperaturas de aire y proceso, rpm, torque y desgaste. IDs y etiquetas de falla no se incluyen como predictores.

## Métodos
- División estratificada (80/20); datos de entrenamiento con 278 fallas.
- Decision Tree, Random Forest, XGBoost.
- Features físicas: potencia `P_kW = T_Nm × 2π × rpm / (60 × 1000)` y diferencial térmico `ΔT = T_proceso − T_aire`.
- Validación cruzada estratificada de 5 particiones; ablación de características; selección de umbral con predicciones out-of-fold.

## Resultados exploratorios

| Experimento | Precision | Recall | F1 | Average Precision |
| --- | ---: | ---: | ---: | ---: |
| RF original, CV media | 0.7221 | 0.5940 | 0.6496 | 0.7023 |
| RF solo potencia, CV media | 0.8449 | 0.6801 | 0.7523 | 0.8002 |
| RF solo diferencial térmico, CV media | 0.8355 | 0.6622 | 0.7384 | 0.7809 |
| RF potencia + ΔT, CV media | 0.8949 | 0.7305 | 0.8031 | 0.8325 |
| XGBoost + features, CV media (umbral 0.5) | 0.5456 | 0.8312 | 0.6584 | 0.8150 |

Umbrales optimizados *sobre predicciones out-of-fold de desarrollo*: RF 0,45: F1 0,8115 (TP 211, FN 67, FP 31, TN 7691); XGBoost 0,73: F1 0,7862. Estas son métricas optimistas de desarrollo, porque se selecciona el umbral sobre las mismas predicciones.

Un ejercicio adicional con nueva semilla de división dio para RF: TP 55, FN 15, FP 9, TN 1921; AP 0,8293. **Esa división no constituye prueba independiente**, pues los registros ya se usaron en el desarrollo previo.

## Limitaciones
- Dataset **sintético**; mecanismos de falla parcialmente definidos por características físicas próximas a las introducidas en Feature Engineering.
- Sin validación temporal, sin identificación de equipos independientes, sin horizonte predictivo.
- Elección de modelos y umbrales expuesta a sesgo de selección; falta validación anidada del pipeline completo o datos externos nuevos.
- El split alternativo con semilla distinta **no resuelve** la contaminación por análisis previos.
- La columna `Target` y el objetivo construido desde `Failure Type` no son idénticos en esta réplica.

## Reproducibilidad
1. Abrir [`industrial-ml-project-01-predictive-maintenanc.ipynb`](industrial-ml-project-01-predictive-maintenanc.ipynb) en Kaggle.
2. Adjuntar el dataset `shivamb/machine-predictive-maintenance-classification`.
3. Ejecutar secuencialmente las celdas; algunas son experimentos alternativos y pueden requerir tiempo de cómputo.
4. Dependencias: `pandas`, `numpy`, `matplotlib`, `scikit-learn`, `xgboost`.

## Próximos pasos
Implementar validación anidada sin reusar decisiones basadas en el test y avanzar a un dataset temporal para detección de anomalías industrial.
