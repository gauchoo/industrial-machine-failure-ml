# Informe técnico — Proyecto 01
## Clasificación de fallas industriales mediante variables físicas e inteligencia de datos

**Proyecto:** Industrial Machine Failure Classification Using Physics-Informed Feature Engineering  
**Fuente:** AI4I 2020, réplica en Kaggle  
**Tipo de estudio:** caso exploratorio educativo en datos sintéticos

### Resumen ejecutivo
Se desarrollaron clasificadores de condiciones de falla sobre 10.000 registros sintéticos de equipos. Una variable objetivo derivada de `Failure Type` identificó 348 casos positivos (3,48%). El trabajo comparó Decision Tree, Random Forest y XGBoost; posteriormente construyó potencia mecánica y diferencia térmica como características adicionales. En validación cruzada estratificada sobre entrenamiento, el F1 promedio de Random Forest aumentó de 0,6496 a 0,8031 al incorporar ambas variables físicas.

### 1. Definición del problema
El objetivo es **clasificar registros de operación**, no anticipar fallas futuras en un horizonte definido. El uso comercial como sistema de mantenimiento predictivo requeriría información temporal, monitoreo, contextualización del equipo y validación adicional.

### 2. Datos y preparación
Se empleó la réplica CSV de AI4I alojada en Kaggle (`shivamb/machine-predictive-maintenance-classification`). Variables de entrada: tipo (L/M/H), temperatura de aire y proceso en K, velocidad en rpm, torque en Nm y desgaste en minutos. La variable objetivo se construyó como `Failure Type != "No Failure"`. No se emplearon como predictores identificadores, `Failure Type`, `Target` ni indicadores del modo de falla.

**Diferencia de etiquetas:** el notebook muestra 348 registros con `Failure Type` distinto de `No Failure`, mientras `Target` presenta otra prevalencia (~339). La definición de etiqueta debe declararse siempre y la discrepancia analizarse antes de comparar métricas con fuentes externas.

### 3. Modelos y validación
Un conjunto de 8.000 registros para entrenamiento y 2.000 para prueba se construyó mediante división estratificada. Se evaluaron árboles de decisión, bosques aleatorios y XGBoost. Para comparar características se utilizó validación cruzada estratificada de 5 particiones sobre los 8.000 registros de entrenamiento. Se desarrollaron dos features: potencia `P = torque × 2π × rpm / 60` (convertida a kW) y diferencia `ΔT = temperatura de proceso − temperatura de aire`.

### 4. Resultados

| Características en Random Forest | Precision CV | Recall CV | F1 CV | AP CV |
| --- | ---: | ---: | ---: | ---: |
| Originales | 0,7221 | 0,5940 | 0,6496 | 0,7023 |
| + potencia | 0,8449 | 0,6801 | 0,7523 | 0,8002 |
| + ΔT | 0,8355 | 0,6622 | 0,7384 | 0,7809 |
| + ambas | 0,8949 | 0,7305 | 0,8031 | 0,8325 |

Con umbrales seleccionados sobre probabilidades out-of-fold de desarrollo, Random Forest con variables físicas y umbral 0,45 alcanzó F1 0,8115 (TP 211; FN 67; FP 31; TN 7691). XGBoost con umbral 0,73 alcanzó F1 0,7862 (TP 217; FN 61; FP 57; TN 7665). En una evaluación de demostración posterior, usando una semilla de división distinta pero registros previamente expuestos al desarrollo, Random Forest obtuvo 55 TP, 15 FN, 9 FP y 1921 TN (AP 0,8293; F1 aproximado 0,82).

### 5. Interpretación industrial
La potencia mecánica y el diferencial térmico constituyen representaciones físicamente interpretables. El estudio de ablación mostró mejora con cada una por separado y mayor rendimiento conjunto. Eso evidencia utilidad para **este conjunto sintético**, pero no demuestra una relación causal ni una mejora generalizable a cualquier activo industrial.

### 6. Limitaciones metodológicas
La selección de umbral utilizó las mismas predicciones out-of-fold para optimizar y reportar el F1, lo que introduce optimismo de selección. La partición alternativa no es independiente porque sus registros formaron parte de análisis anteriores. Se recomienda validación anidada, donde la selección completa ocurra solo dentro de cada pliegue interno, o prueba externa inédita. Los datos sintéticos codifican mecanismos físicos de falla y carecen de verdadera dimensión temporal para afirmar anticipación predictiva.

### 7. Conclusiones y próximos pasos
El estudio establece un flujo reproducible de exploración, clasificación desbalanceada, validación, ingeniería de características, ablación y selección de umbrales. El mayor aprendizaje es que las características físicamente fundamentadas mejoraron el desempeño en este experimento. Siguientes acciones: (1) fortalecer evaluación del pipeline, (2) publicar código y README, (3) iniciar un segundo caso con señales temporales y detección de anomalías sobre datos de activos.
