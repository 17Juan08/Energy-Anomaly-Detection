# 🏢 Smart Energy Anomaly Detection: Machine Learning para la Eficiencia Energética

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-v1.5+-darkblue.svg)](https://pandas.pydata.org/)
[![LightGBM](https://img.shields.io/badge/LightGBM-Predictive%20Model-green.svg)](https://lightgbm.readthedocs.io/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-Validation-orange.svg)](https://scikit-learn.org/)

Este repositorio contiene la solución completa de Machine Learning para la detección automatizada de anomalías horarias (*point anomalies*) en sistemas de medición inteligente (*Smart Meters*) de energía eléctrica, utilizando datos de más de 400 edificios comerciales (basado en la versión extendida de la competencia internacional *ASHRAE - Great Energy Predictor III*).

---

## 📌 Contexto del Problema y Desafío de Negocio

A nivel global, la infraestructura urbana consume aproximadamente un tercio de la energía del planeta, y **se estima que más del 20% se desperdicia de forma innecesaria** debido a fallas de equipos, configuraciones erróneas o fallas humanas. 

Este proyecto implementa una solución guiada por datos (*Data-Driven*) para identificar anomalías en perfiles de consumo eléctrico en tiempo real. El principal reto de este problema consiste en **entrenar un modelo capaz de generalizar predicciones en un conjunto de prueba para 206 edificios nuevos** que el sistema jamás observó durante su entrenamiento, utilizando como métrica de optimización el área bajo la curva ROC (AUC-ROC).

---

## 🧠 Hacks Técnicos y Decisiones de Arquitectura

Para resolver los retos típicos de las series temporales y optimizar el rendimiento de los modelos basados en árboles (XGBoost/LightGBM), se aplicaron estrategias avanzadas de ingeniería de datos y diseño experimental:

### 1. Validación Estructurada por Grupos (Evitando Data Leakage)
En series temporales, un split aleatorio tradicional (`train_test_split`) mezcla registros del mismo edificio en fechas cercanas, causando que el modelo memorice la firma de consumo de cada infraestructura en lugar de aprender patrones generales.
* **Solución:** Se implementó una división por bloques basada en los identificadores de los edificios utilizando el operador módulo (`building_id % 5`). 
* Esto destina un **80% de los edificios completos** con su historial intacto al entrenamiento, y el **20% restante** para evaluar de forma estricta la capacidad de generalización en instalaciones completamente desconocidas.

### 2. Ingeniería de Características Temporal (Lags y Residuos)
* **Lags y Diferenciación:** Se calcularon desfases temporales clave ($t-1, t+1, t-24, t-168$) para mapear la estacionalidad horaria, diaria y semanal del consumo.
* **Residuos:** En lugar de alimentar los consumos crudos, se computó la diferencia directa entre el valor del desfase y el consumo actual (`lag_value - meter_reading`), creando un indicador de "desviación de patrón habitual".
* **Transformación Parabólica:** Se introdujo la variable cuadrática `meter_reading**2` para facilitar que los árboles detectaran picos exponenciales de consumo anómalo de forma mucho más ágil.

### 3. Mitigación de Desbalance de Clases (Submuestreo Doble)
Las anomalías representan una fracción muy pequeña de los datos reales. Para evitar un modelo sesgado que predijera siempre la clase mayoritaria (consumo normal), se aplicó una técnica híbrida:
* Se extrajeron **dos muestras de control independientes** del dataset normal de tamaño exacto al de anomalías ($n = 37,296$).
* Se ensambló un dataset balanceado al **50% / 50%** duplicando de forma controlada la clase de anomalías reales para consolidar los patrones críticos de falla.

---

## 📊 Resultados Obtenidos

Gracias al diseño experimental estructurado y libre de fuga de datos, el modelo entrenado reportó métricas extraordinariamente estables:

| Conjunto de Datos | Métrica (Accuracy) | Estado |
| :--- | :---: | :--- |
| **Entrenamiento** | `99.59%` | Ajuste Óptimo |
| **Validación (Edificios Nuevos)** | `96.22%` | Alta Capacidad de Generalización |

---

## 🏁 Conclusiones y Valor de Negocio

La culminación de este proyecto demostró que la detección de fallas en redes inteligentes requiere un entendimiento profundo del contexto físico de los datos más allá de la aplicación de algoritmos complejos:

* **Generalización Real para Producción:** El alto porcentaje de acierto del **96.22% en Validación** sobre un bloque de edificios completamente ajenos al entrenamiento valida que el modelo es robusto, escalable y está listo para ser desplegado en nuevas infraestructuras sin requerir datos históricos previos de calibración.
* **El Poder de la Ingeniería de Características:** El uso de diferencias sobre *lags* temporales demostró que la estacionalidad semanal y diaria es el indicador más potente para identificar desvíos energéticos. El modelo aprendió con éxito las "firmas de error" sutiles en los medidores.
* **Control de Falsas Alarmas:** La arquitectura del pipeline y la optimización basada en la robustez de los grupos de validación permiten controlar la tasa de falsos positivos, garantizando un sistema de alertas confiable para los equipos de mantenimiento físico e infraestructura de las edificaciones.
* **Eficiencia Computacional:** La optimización y modularidad del código (vectorización en Pandas y cálculo de lags enfocado puramente en los archivos de características auxiliares) redujeron drásticamente los costos de cómputo y el uso de memoria RAM, haciendo el pipeline viable para entornos de producción en la nube.

---

## 🚀 Siguientes Pasos Recomendados

1. **Modelado Multivariable:** Integrar variables climáticas externas (temperatura, humedad, velocidad del viento) para enriquecer el contexto, ya que el consumo de energía en climatización está fuertemente correlacionado con el clima exterior.
2. **Implementación de Ventanas Móviles (Online Learning):** Alimentar el modelo en producción con datos en tiempo real mediante micro-batches horarios para actualizar las diferencias temporales dinámicamente.
