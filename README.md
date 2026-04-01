# Predicción de High Engagement en el VLE Dataset

## Descripción del Proyecto

Este proyecto tiene como objetivo predecir si una video-lectura alcanzará un nivel de **High Engagement** antes de su publicación, utilizando exclusivamente variables disponibles previo a la interacción de los usuarios.

El dataset utilizado proviene del **VLE Dataset v1 (12k)**, un conjunto de datos a gran escala construido a partir de información agregada de consumo de video-lecturas del repositorio [VideoLectures.net](http://videolectures.net/).

Mientras que el dataset original proporciona métricas continuas de engagement (NET, MNET, ANET), en este proyecto el problema se reformula como una tarea de **clasificación binaria**, con el fin de abordar el problema de *item cold-start* en sistemas de recomendación educativa.

## Origen del Dataset

El **VLE Dataset v1**:

- Contiene **11,568 video-lecturas en inglés**
- Incluye datos agregados de más de **1.1 millones de usuarios**
- Proporciona métricas de engagement basadas en **Normalized Engagement Time (NET)**
- Fue diseñado para estudiar el engagement poblacional en video-lecturas educativas
- Está orientado a problemas de recomendación y análisis de calidad educativa

El dataset incluye múltiples tipos de características: metadata, contenido textual, características semánticas basadas en Wikipedia y variables específicas del video.



## Reformulación del Problema

El dataset original ofrece etiquetas continuas relacionadas con:

- MNET (Median Normalized Engagement Time)
- ANET (Average Normalized Engagement Time)
- SMNET / SANET
- Número de visualizaciones
- Calificación promedio por estrellas

En este proyecto:

- Se construyó una variable binaria `engagement`.
- Debido a la ausencia de una ruptura estructural clara en la distribución del engagement, se utilizó una estrategia basada en **cuartiles (75–25)** para definir la clase positiva.
- El dataset resultante presenta un **desbalance de clases (75% – 25%)**, lo cual se considera en la fase de modelado.

Este enfoque simula un escenario realista donde se desea predecir el nivel de engagement antes de que el video reciba interacción de usuarios.


## Grupos de Variables Utilizados

El VLE Dataset contiene cuatro grandes grupos de características. En este proyecto se utilizaron aquellas disponibles antes de la publicación del video.


### Variables basadas en Metadata

- `categories` (dominio agrupado como *stem* o *misc*)
- `type` (tipo de conferencia: lecture, tutorial, invited talk, etc.)

Estas variables fueron anonimizadas y agrupadas en el dataset original para preservar privacidad.


### Variables basadas en Contenido (Transcript)

Extraídas del texto de la transcripción mediante métricas lingüísticas y de NLP:

- `document_entropy`
- `easiness` (Flesch–Kincaid Easiness)
- `fraction_stopword_presence`
- `normalization_rate`
- `pronoun_rate`

Estas variables capturan aspectos relacionados con:

- Cobertura temática
- Comprensibilidad
- Estilo de presentación



### Variables basadas en Wikipedia

Generadas mediante *Entity Linking* entre la transcripción del video y conceptos de Wikipedia.

Se dividen en dos grupos:

1. Autoridad temática (PageRank en grafo semántico)

    - `auth_topic_rank_1_score`
    - `auth_topic_rank_2_score`
    - `auth_topic_rank_3_score`
    - `auth_topic_rank_4_score`
    - `auth_topic_rank_5_score`

2. Cobertura temática (Similitud coseno)

    - `coverage_topic_rank_1_score`
    - `coverage_topic_rank_2_score`
    - `coverage_topic_rank_3_score`
    - `coverage_topic_rank_4_score`
    - `coverage_topic_rank_5_score`

    Estas variables cuantifican la relevancia y profundidad semántica del contenido.



### Variables específicas del Video

- `duration`
- `silent_period_rate`
- `freshness`

Capturan características estructurales y temporales de la conferencia.

# Exploración de Datos (EDA)

Durante el análisis exploratorio se analizaron:

- distribución de variables
- presencia de valores atípicos
- correlaciones entre variables
- balance de la variable objetivo

El análisis mostró que muchas variables presentan **distribuciones altamente sesgadas**, algo esperado en métricas derivadas de texto y comportamiento de usuario.  

Asimismo, se identificaron **niveles moderados de correlación** entre algunas variables semánticas basadas en Wikipedia.

Este análisis permitió orientar las decisiones de preprocesamiento y selección de variables.

# Preprocesamiento de Datos

El pipeline de preprocesamiento incluyó:

### Escalamiento

Se aplicó **StandardScaler** para normalizar variables numéricas en modelos sensibles a escala, como regresión logística.

### Reducción de dimensionalidad

Se evaluaron tres representaciones del dataset:

1. **Dataset completo**
2. **Dataset reducido** eliminando variables altamente correlacionadas
3. **Dataset transformado mediante PCA**

Esto permitió analizar cómo afecta la dimensionalidad al desempeño de los modelos.

# Preprocesamiento de Datos

El pipeline de preprocesamiento incluyó:

### Escalamiento

Se aplicó **StandardScaler** para normalizar variables numéricas en modelos sensibles a escala, como regresión logística.

### Reducción de dimensionalidad

Se evaluaron tres representaciones del dataset:

1. **Dataset completo**
2. **Dataset reducido** eliminando variables altamente correlacionadas
3. **Dataset transformado mediante PCA**

Esto permitió analizar cómo afecta la dimensionalidad al desempeño de los modelos.

# Evaluación del Ajuste

Para analizar la capacidad de generalización de los modelos se comparó el desempeño en **train vs test** mediante la métrica **ROC AUC**.

Se calculó el **AUC gap**, definido como: AUC gap = AUC Train - AUC Test

Los resultados muestran:

- **Logistic Regression**: excelente generalización (gap ≈ 0)
- **Decision Tree**: ligero sobreajuste
- **Random Forest**: mayor gap (~0.11)
- **XGBoost**: buen equilibrio entre capacidad de ajuste y generalización

# Evaluación de Modelos

Los modelos fueron evaluados utilizando múltiples métricas:

- **ROC AUC**
- **Recall**
- **Precision**
- **Accuracy**

También se utilizaron herramientas visuales como:

- Curvas ROC
- Curvas Precision–Recall
- Matrices de confusión

Dado que el objetivo del proyecto es **minimizar falsos negativos**, se priorizó el **Recall**.

# Ajuste del Umbral de Decisión

El umbral de clasificación por defecto (0.5) fue ajustado para cada modelo con el objetivo de alcanzar: Recall >= 0.8

Esto permite identificar la mayor cantidad posible de videos con alto engagement, aceptando un incremento moderado en falsos positivos.


# Selección del Modelo Final

Tras el ajuste del umbral, se compararon los mejores modelos de cada algoritmo.

El modelo final seleccionado fue:

**XGBoost entrenado con el dataset completo**

Resultados:

| Métrica | Valor |
|------|------|
| Recall | 0.80 |
| Precision | 0.52 |
| Accuracy | 0.75 |
| AUC | 0.86 |

Este modelo ofrece el **mejor equilibrio entre detección de videos con alto engagement y control de falsos positivos**.

---

# Conclusiones

Los resultados muestran que es posible **predecir el potencial de engagement de una video-lectura antes de su publicación** utilizando únicamente características del contenido y metadata.

Entre los modelos evaluados:

- los modelos lineales presentan mejor generalización
- los modelos basados en árboles capturan relaciones más complejas
- **XGBoost logra el mejor desempeño global**

Este tipo de modelo puede integrarse en **sistemas de recomendación educativa**, permitiendo priorizar contenidos con mayor probabilidad de generar engagement.

---

# Estructura del Proyecto
├── notebooks

│    └──  data_preprocessing.ipynb

│   └──  modeling.ipynb

├── data

│    └──processed

           └── Datasets for training

    └──raw

           └── vle_dataset

├── README.md


