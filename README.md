
# Optimización de hiperparámetros de LightGBM

## Descripción del proyecto

Este proyecto evalúa el efecto de aumentar la cantidad de hiperparámetros optimizados simultáneamente mediante optimización bayesiana.

## Objetivo

Evaluar si aumentar la cantidad de hiperparámetros optimizados simultáneamente mejora la ganancia obtenida por el modelo.


## Experimentos

Se diseñaron tres experimentos para evaluar el efecto de incrementar la cantidad de hiperparámetros optimizados simultáneamente mediante optimización bayesiana (BO).

- **Experimento A (5 HP):** incluye los cinco hiperparámetros del script original: `num_iterations`, `num_leaves`, `learning_rate`, `feature_fraction` y `min_sum_hessian_in_leaf`.

- **Experimento B (11 HP):** incorpora seis hiperparámetros adicionales relacionados con la complejidad del árbol, la regularización y el muestreo de observaciones: `max_depth`, `min_data_in_leaf`, `lambda_l1`, `lambda_l2`, `bagging_fraction` y `bagging_freq`.

- **Experimento C (14 HP):** incorpora tres hiperparámetros adicionales: `scale_pos_weight`, `min_gain_to_split` y `feature_fraction_bynode`, relacionados con el balance de clases, el control de divisiones del árbol y la selección de variables por nodo.

## Herramientas y entorno de ejecución

- **Lenguaje de programación:** R
- **Modelo de machine learning:** LightGBM
- **Método de optimización:** Optimización bayesiana
- **Infraestructura:** Máquina virtual (VM) de Google Cloud
- **Entorno de trabajo:** JupyterLab
- **Evaluación de resultados:** Kaggle

## Metodología

Se utilizó LightGBM como algoritmo de aprendizaje automático y optimización bayesiana (BO) para la búsqueda de las mejores combinaciones de hiperparámetros.

Se estableció un límite de 300 iteraciones de BO por experimento, manteniendo el workflow original y modificando únicamente los componentes necesarios para la optimización.

Los rangos de búsqueda de los hiperparámetros se definieron previamente, manteniendo los mismos rangos para aquellos compartidos entre experimentos. También se evaluó la proximidad de los valores optimizados a los límites establecidos para identificar posibles necesidades de ampliación.

Para automatizar las ejecuciones, se desarrolló un script lanzador de semillas que permite ejecutar secuencialmente los workflows de optimización. Cada integrante del grupo trabajó con cinco semillas diferentes.

Para cada experimento y semilla se realizaron siete envíos a Kaggle, variando la cantidad de clientes seleccionados entre 1800 y 2400, en incrementos de 100. Esto representa un total de 105 envíos por cada conjunto de cinco semillas.

El entrenamiento del modelo se realizó con datos de julio de 2021 y su rendimiento se evaluó sobre septiembre de 2021.

La comparación entre experimentos se realizó principalmente mediante las ganancias obtenidas en el leaderboard público de Kaggle, considerando también la variabilidad de los resultados entre semillas.

## Resultados

Se analizaron las ganancias máximas obtenidas en el leaderboard público de Kaggle para los tres experimentos, utilizando 10 semillas por experimento.

### Comparación de ganancias

| Experimento | HP optimizados | Ganancia media | Mediana | Desvío estándar |
| ----------- | -------------- | -------------- | ------- | --------------- |
| A | 5 | 89.246 | 88.016 | 3.839 |
| B | 11 | 88.506 | 88.640 | 3.542 |
| C | 14 | 85.117 | 84.652 | 3.154 |

Los experimentos A y B obtuvieron ganancias similares. El experimento A presentó la mayor ganancia promedio, mientras que B tuvo una mediana ligeramente superior. Por otro lado, el experimento C fue el que obtuvo las menores ganancias.

### Análisis estadístico

Para comparar las ganancias entre experimentos, se utilizó el test de Wilcoxon para muestras apareadas, con corrección de Holm para comparaciones múltiples (α = 0,05).

| Comparación | Valor p ajustado |
| ----------- | ---------------- |
| A vs. B | 0,9057 |
| A vs. C | 0,2058 |
| B vs. C | 0,0241 |

Se encontraron diferencias estadísticamente significativas solamente entre los experimentos B y C, siendo mayores las ganancias obtenidas en B. Entre A y B, y entre A y C, no se encontraron diferencias significativas.

### Resultados complementarios

También se analizaron los valores de los hiperparámetros obtenidos en las optimizaciones para evaluar si quedaron cerca de los límites de los rangos de búsqueda.

Se observó que `learning_rate` presentó valores próximos al límite inferior en los tres experimentos, mientras que en el experimento A, `num_leaves` presentó varios valores próximos al límite superior.

Esto permitiría considerar la ampliación de algunos rangos de búsqueda en futuras optimizaciones.

### Conclusión

Los resultados obtenidos no permitieron confirmar nuestra hipótesis de que optimizar simultáneamente una mayor cantidad de hiperparámetros permite obtener mayores ganancias en Kaggle.

Si bien el experimento A obtuvo la mayor ganancia promedio, el experimento B presentó una mediana ligeramente superior y un menor tiempo de ejecución. Sin embargo, no encontramos diferencias estadísticamente significativas entre ambos experimentos.

Los experimentos B y C presentaron menores tiempos de ejecución que A. Una posible explicación es la incorporación de hiperparámetros relacionados con el submuestreo de observaciones a partir del experimento B.

Por lo tanto, la elección entre A y B dependerá de qué se quiera priorizar: una búsqueda más sencilla con 5 hiperparámetros (A) o un menor tiempo de ejecución (B).

### Posibles mejoras y futuros experimentos

En futuras optimizaciones se podrían ampliar los rangos de búsqueda de aquellos hiperparámetros cuyos valores quedaron próximos a los límites establecidos. Por falta de tiempo, no pudimos evaluar estas ampliaciones con la cantidad de semillas necesaria para obtener resultados comparables.

También se podría aumentar la cantidad de iteraciones de optimización bayesiana, que en este trabajo se fijó en 300, para evaluar si esto permite encontrar mejores combinaciones de hiperparámetros y obtener mayores ganancias en Kaggle.

## Estructura del repositorio

El repositorio contiene los scripts utilizados para los experimentos de optimización bayesiana, los resultados obtenidos y los archivos correspondientes al análisis de los datos.

### Scripts de los experimentos

**`Script_Experimentos_A-B-C/`**

Contiene los workflows de optimización de los experimentos A, B y C, junto con los scripts lanzadores de semillas correspondientes a cada integrante del grupo.

Los archivos que comienzan con a1_ corresponden al alumno 1, mientras que los restantes corresponden al alumno 2. Cada integrante utilizó cinco semillas diferentes para ejecutar los tres experimentos.

### Resultados de las optimizaciones

**`Resultados/`**

Contiene los archivos generados durante las ejecuciones, organizados por experimento:

- `Experimento A/`
- `Experimento B/`
- `Experimento C/`

Dentro de cada experimento, los resultados se encuentran organizados en subcarpetas según las ejecuciones realizadas.

### Ampliación de envíos a Kaggle

**`Ampliación_Kaggle/`**

Contiene notebooks utilizados para ampliar o modificar las evaluaciones realizadas en Kaggle, identificados según la semilla y el experimento correspondiente.

### Análisis de resultados

- **Notebook de análisis (`.ipynb`):** contiene el procesamiento de los resultados, las estadísticas descriptivas, las comparaciones entre experimentos y las visualizaciones.
- **`Resultados_grupoA.xlsx`:** archivo Excel con los resultados recopilados de los experimentos.

## Requisitos e instrucciones de ejecución

Los experimentos se desarrollaron utilizando notebooks de Jupyter (`.ipynb`), ejecutados en una máquina virtual de Google Cloud.

### Requisitos

- R y Python.
- LightGBM y otros paquetes de R utilizados en los workflows.
- `papermill` para automatizar la ejecución de los notebooks.
- Entorno compatible con Jupyter Notebook.

### Ejecución

Los workflows de los experimentos A, B y C están desarrollados en R, mientras que los scripts lanzadores de semillas están escritos en Python y utilizan `papermill` para ejecutar automáticamente los experimentos con las distintas semillas.

Para ejecutar los notebooks en otro entorno, es necesario contar con el dataset original, instalar los paquetes requeridos, modificar las rutas de los archivos en los scripts lanzadores y workflows, y configurar el acceso a Kaggle.