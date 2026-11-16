# Predicción final – Laboratorio 1 | Analista Junior

## 1. Objetivo: 
El objetivo de este trabajo es generar una predicción para septiembre de 2021 en la competencia de Kaggle de Analista Junior,
buscando maximizar la ganancia obtenida mediante un modelo de clasificación LightGBM.

Para ello, se evaluaron distintas combinaciones de las soluciones propuestas por los equipos A y B en los experimentos colaborativos 
de la materia. En la sección `Modificaciones realizadas` se observan que combinacoines se realizaron a partir de las recomendación de
cada equipo, evaluando si se complemetan o no. Luego se muestran los resultados obtendidos para estas pruebas. 

Al haber realizado el problema 12, de cantidad de HP a optimizar silumtaneamente en la BO, y haber recomendado, puetos que no se 
observaron diferencias siganificativas que se podian utilizar 5HP si se deseaba ir por lo simple u 11 HP si la opción era disminuir 
el tiempo de ejecución. Se decidió probar ambas opciones sobre una misma configuración del workflow.

## 2. Organización de pruebas

Se utilizó como punto de partida el workflow `v00_bl_729_final_junior` basado en el notebook Senior, adaptándolo para trabajar con el 
dataset de Analista Junior. Este no se ejecutó, solo se creeó para poder hacer las modificaciones de prueba.

El dataset contiene información mensual de clientes bancarios desde enero de 2019 hasta septiembre de 2021. El workflow original incluye
etapas de corrección de variables, tratamiento del data drifting, ingeniería de variables, selección de atributos, optimización de 
hiperparámetros y entrenamiento final.

Sobre este workflow baseline se incorporaron las distintas modificaciones, modificando solo donde se relaiza la prueba y dejando el resto 
del código sin modificaciones (mismo procedimiento que para Experimentos Colaborativos).

Nota: Ya con varias pruebas realizadas utilizando la versión Jr adaptada desde el Sr, la cátedra comprtió una versión del workflow adaptada
para Analista Junior. Por este motivo, decidí continuar con mi versión adaptada  para mantener la misma base en todos los experimentos.

## 3. Metotodología general
### 3.1. Modelo y optimización

En todas las pruebas se utilizó LightGBM como algoritmo de clasificación y optimización bayesiana para seleccionar los hiperparámetros.
workflow inicial, LightGBM, BO y 300 iteraciones 

-Se establecieron las siguientes condiciones comunes:
-300 iteraciones inteligentes de BO, ampliando las 30 del script original.
-AUC como métrica de optimización, calculada sobre los meses definidos para validación.
-training_pct = 1.0, utilizando todos los registros disponibles de entrenamiento, sin el 
submuestreo de la clase CONTINUA definido por ese parámetro.

Se mantuvieron los mismos rangos de búsqueda para los cinco hiperparámetros originales en las pruebas 1, 2, 3, 4, 6 y 8. 
En el caso del semillerio, se utilizaron 30 semillas para el ensamble final de las primeras 6 pruebas. Esto cambio a partir de la prueba 7, 
donde se utilizaron 100 semillas.
La idea fue partir de una misma configuración para poder comparar los resultados y ver cómo cambiaba la ganancia al incorporar las 
modificaciones propuestas por los distintos equipos.


### 3.2. Predicción final y evaluación

En las primeras seis pruebas se entrenaron 30 modelos LightGBM, utilizando distintas semillas y los hiperparámetros seleccionados 
mediante BO. A partir de la prueba 7, se amplió el semillerio a 100 modelos.

Para cada experimento se obtuvo una predicción final a partir del promedio de las probabilidades generadas por los modelos.

Con esta predicción probabilística, se realizaron distintos envíos a la competencia de Kaggle para Analista Junior, variando la 
cantidad de clientes seleccionados para evaluar cómo cambiaba la ganancia.

La estrategia utilizada cuando la ganancia máxima se encontraba en uno de los extremos del rango evaluado fue probar nuevas
cantidades de clientes para buscar una ganancia mayor.


## 4. Modificaciones evaluadas

Las modificaciones seleccionadas corresponden a diferentes etapas del workflow.

| Problema | Modificación | Descripción |
|---|---|---|
| **2A** | `rank_cero_fijo` intrames | Se utilizó un ranking de las variables monetarias dentro de cada mes, en lugar de la deflación original. |
| **4B** | Ingeniería de variables E2 | Se agregaron nuevas variables para analizar los cambios en los últimos meses: variación, cantidad de caídas y pendiente. Corresponde al experimento E2. |
| **5A** | L1 + C3 + A | Se amplió el período de pandemia de marzo a octubre de 2020, excluyendo esos meses del entrenamiento y modificando los lags y deltas relacionados. Se utilizó como modelo el experimento 5122.
| **6B** | Canaritos | Se modificó el criterio de selección de variables, pasando de 2 a 1 desvío. Correspondiente al experimento 2|
| **8A** | Validación de BO | Se utilizaron junio y julio de 2021 para la validación, en lugar de solamente julio. Correspondiente al experiemnto A2 |
| **16B** | Exclusión de variables | Se excluyeron `numero_de_cliente` y `foto_mes` como variables predictoras del modelo. Correspondiente al experimento Sin Ambos |

Estas modificaciones se combinaron en diferentes pruebas para explorar su comportamiento conjunto y evaluar si las configuraciones 
resultantes mejoraban la ganancia.

## 5. Diseño de las pruebas

Se plantearon nueve pruebas, utilizando distintas combinaciones de las propuestas anteriores.
| Prueba | Experimento  | Modificaciones | HP optimizados |   Semillerio   |  
|--------|------|----------------|----------------|----------------|
| 1      | 9160 | 5A + 8A        | 5              | 30             | 
| 2      | 9161 | 4B + 6B + 16B  | 5              | 30             | 
| 3      | 9162 | 5A + 4B + 2A   | 5              | 30             | 
| 4      | 9163 | 5A + 4B        | 5              | 30             |
| 5      | 9164 | 5A + 8A        | 11             | 30             | 
| 6      | 9165 | 5A             | 5              | 30             | 
| 7      | 9166 | 5A             | 5              | 100            | 
| 8      | 9167 | 5A + 8A        | 5              | 100            | 
| 9      | 9168 | 5A             | 5              | 100            | 

### Prueba 1 – Problemas 5A + 8A

En esta primera prueba se combinaron las modificaciones 5A y 8A.

Por un lado, 5A propone tratar de forma particular los meses afectados por la pandemia. Por otro, 8A modifica la cantidad
de meses utilizados para validar durante la optimización bayesiana.

La idea fue evaluar cómo funcionaban ambas modificaciones juntas.

### Prueba 2 – Problemas 4B + 6B + 16B

En esta prueba se incorporaron las variables históricas de E2 (4B), se modificó el criterio de selección de variables 
mediante canaritos (6B) y se excluyeron numero_de_cliente y foto_mes de los predictores de LightGBM (16B).

### Prueba 3 – Problemas 5A + 4B + 2A

Se incorporó rank_cero_fijo intrames para las variables monetarias, junto con las variables históricas de E2 y el tratamiento
de pandemia ampliada de 5A.

Las variables E2 se calcularon después del tratamiento de drifting y de la modificación de lags correspondiente a 5A.

### Prueba 4 – Problemas 5A + 4B

En esta prueba se decidió probar solamente las modificaciones 5A y 4B, sin incorporar el cambio de drifting de 2A, 
para ver que efecto tenia esta combinación.

### Prueba 5 – Problemas 5A + 8A con 11 hiperparámetros

En esta prueba se decidió utilizar la misma configuración que en la prueba 1, porque hasta ese momento 
había obtenido la mayor ganancia. La idea fue evaluar si al aumentar de 5 a 11 los hiperparámetros optimizados
mediante BO se podía mejorar ese resultado..

Los cinco hiperparámetros originales fueron:

num_iterations, num_leaves, learning_rate, feature_fraction y min_sum_hessian_in_leaf.

A estos se les agregaron otros seis:

max_depth, min_data_in_leaf, lambda_l1, lambda_l2, bagging_fraction y bagging_freq.

Se mantuvieron las 300 iteraciones de BO y las 30 semillas del ensamble.


### Prueba 6 – Modificación 5A con 5 hiperparámetros

Esta prueba busca evaluar cómo funciona el tratamiento de los meses de pandemia propuesto por 5A de manera individual.

Por otro lado, permite comparar su rendimiento con el obtenido al incorporar otras modificaciones, como en los experimentos 
9160 y 9163, y de forma secundaria con el 9162.

### Prueba 7 – Modificación 5A con ampliación de rangos de HP y semillero de 100 modelos

Esta prueba se realizó para evaluar si ampliar los rangos de búsqueda de los hiperparámetros `learning_rate` y `num_iterations`, 
que se encontraban próximos a los límites de búsqueda de la BO, permitía obtener mejores resultados.

Además, por sugerencia del profesor a través de Zulip, se decidió aumentar el semillerio de 30 a 100 modelos.

### Prueba 8 – Problemas 5A + 8A - 5 HP - semillerio 100

Se tomó como referencia la prueba 1 porque había obtenido la mayor ganancia en Kaggle y ninguno de los hiperparámetros 
optimizados había quedado próximo a los límites de búsqueda de la BO.

La idea fue evaluar si aumentando el semillerio de 30 a 100 modelos se podía mejorar ese resultado.

### Prueba 9 – Problemas 5A  - 5 HP con ampliación rango - semillerio 100

Esta prueba se realizó después de la prueba 7, donde se observó una mejora en la ganancia al ampliar 
los rangos de `learning_rate` y `num_iterations`. Sin embargo, `num_leaves` quedó próximo al límite de búsqueda de la BO.

Por este motivo, se decidió realizar una nueva prueba sobre la base del experimento 9166, ampliando también el rango 
de búsqueda de `num_leaves`.


## 6. Consideraciones para la comparación

Durante la BO se utilizó la AUC para optimizar los hiperparámetros, pero para comparar los resultados 
de las distintas pruebas se tomó como referencia la ganancia obtenida en Kaggle.

Hay que tener en cuenta que en algunas pruebas se modificaron varias cosas al mismo tiempo, por lo que
no siempre es posible saber cuál de esos cambios produjo una mejora o disminución de la ganancia.

Por otro lado, se realizaron varios envíos a Kaggle, probando distintas cantidades de clientes. Esto puede generar 
cierto sobreajuste a los resultados del leaderboard público. Por eso, obtener una mayor ganancia en el público no significa
necesariamente que se vaya a obtener el mismo resultado en el privado.

## 7. Resultados obtenidos

| Exp. | Mejor AUC | Ganancia máxima | Mejor envío (clientes) | HP próximos al límite | Semillas | Tiempo aprox. (h) |
|---|---:|---:|---:|---|---:|---:|
| 9160 | 0.93117 | **96864** | 1700 | Ninguno | 30 | 4 |
| 9161 | 0.93413 | 93790 | 2200 | `num_leaves` | 30 | 7 |
| 9162 | 0.93587 | 90882 | 1400 | `num_iterations`, `learning_rate` | 30 | 7 |
| 9163 | 0.93503 | 91547 | 1900 | `num_leaves`, `num_iterations`, `learning_rate` | 30 | 7 |
| 9164 | 0.93113 | 93707 | 1800 | `max_depth`, `bagging_freq` | 30 | 3 |
| 9165 | 0.93542 | 94205 | 1800 | `learning_rate` | 30 | 4 |
| 9166 | 0.93485 | **96697** | 1700 | `num_leaves` | 100 | 8 |
| 9167 | 0.93074 | 88058 | 1900 | `num_iterations`, `learning_rate` | 100 | 6 |
| 9168 | 0.93473 | 93956| 1800| `num_iterations` | 100 | 9 |

Los hiperparámetros señalados son aquellos que quedaron dentro del 10% más cercano a alguno de los límites del rango de búsqueda de la BO. Los tiempos de ejecución son aproximados.

### 7.1 Resultados generales

De las nueve pruebas realizadas, el experimento 9160 obtuvo la mayor ganancia en el leaderboard público de Kaggle. El expeirmento 9166 obtuvo un resultados muy similar pero ligeramente inferior. 
En ambos casos, el máximo se encontró en el envio de 1700 clientes. 

En general, se observó que las prueba ocn mayor AUC no correspondieron a las mayores ganancias en
Kaggle. 

En cuanto a la optimización de hiperparámetros, en varios experimentos los mejores valores quedaron próximos a los límites de búsqueda de la BO. Esto llevó a realizar nuevas pruebas ampliando algunos rangos, como en los experimentos 9166 y 9168.

### 7.2. Comparación de resultados

#### 7.2.1. Comparación de las modificaciones

Este apartado permite observar los resultados que surgen de implementar las recomendaciones porpuestas por los ditintos grupos, explicadas en el apartado `4. Modificaciones evaluadas`. 

Para ello se toma como referencia la prueba 6 (experimento 9165), donde se utilizó unicamente 5A.

#### Prueba 6 (9165) vs prueba 1 (9160)
 Al incorporar 8A a la modificación 5A, se obtuvo una ganancia superior en Kaggle (96864 frente a 94205), con una menor cantidad de clientes seleccionados (1700 frente a 1800) y tiempos de ejecución similares.

En ambos casos fue necesario ampliar los envíos a Kaggle, probando cantidades menores de clientes.

Además, en el experimento 9165, learning_rate quedó próximo a uno de los límites de búsqueda de la BO, mientras que en el 9160 ninguno de los HP presentó esta condición.

La introducción de 8A, en este caso y en esta corrida, permitió obtener una mayor ganancia en Kaggle. 

#### Prueba 6 (9165) vs prueba 4 (9163)

Al incorporar 4B a la modificación 5A, se obtuvo una ganancia inferior en Kaggle (91547 frente a 94205), con una mayor cantidad de clientes seleccionados (1900 frente a 1800) y un mayor tiempo de ejecución aproximado (7 h frente a 4 h).

Solo fue necesario ampliar los envíos a Kaggle en el experimento 9165.

Además, en ambos experimentos, `learning_rate` quedó próximo a uno de los límites de búsqueda de la BO. En el 9163 también ocurrió con `num_iterations` y `num_leaves`.

La introducción de 4B, en este caso y en esta corrida, no mejoró la ganancia obtenida en Kaggle. 

#### Prueba 6 (9165) vs prueba 3 (9162)

Al incorporar 4B y 2A a la modificación 5A, se obtuvo una ganancia inferior en Kaggle (90882 frente a 94205), con una menor cantidad de clientes seleccionados (1400 frente a 1800) y un mayor tiempo de ejecución aproximado (7 h frente a 4 h).

En ambos casos fue necesario ampliar los envíos a Kaggle, probando cantidades menores de clientes.
En el 9162, se requirieron dos ampliaciones para hallar la ganancia máxima.

Además, en ambos experimentos, `learning_rate` quedó próximo a uno de los límites de búsqueda de la BO. En el 9162 también ocurrió con `num_iterations`.

La introducción de 4B y 2A, en este caso y en esta corrida, no mejoró la ganancia obtenida en Kaggle. 

Además, se realizaron las siguientes comparaciones:
#### Prueba 3 (9162) vs prueba 4 (9163)

Al incorporar 2A a la combinación 4B + 5A, se obtuvo una ganancia inferior en Kaggle (90882 frente a 91547), con una menor cantidad de clientes seleccionados (1400 frente a 1900) y tiempos de ejecución similares.

En el 9162, se requirieron dos ampliaciones de envíos a Kaggle para hallar la ganancia máxima.

Además, en ambos experimentos, `learning_rate` y `num_iterations` quedaron próximos a uno de los límites de búsqueda de la BO. En el 9163 también ocurrió con `num_leaves`.

La introducción de 2A, en este caso y en esta corrida, no mejoró la ganancia obtenida en Kaggle. 

#### Prueba 2 (9161) vs general (otras pruebas)

La prueba 2 (9161) combinó las modificaciones 4B, 6B y 16B. En este caso, se decidió comparar su resultado con el resto de las pruebas, ya que las modificaciones 6B y 16B solamente se evaluaron en esta combinación.

Se observa que obtuvo una ganancia máxima en Kaggle (93790) similar a la de los experimentos 9168 (93956) y 9164 (93707). iFue inferior a la de los experimentos 9160 (96864), 9166 (96697) y 9165 (94205), pero superior a la de los experimentos 9162 (90882), 9163 (91547), y 9167 (88058). 

En este caso no fue necesario ampliar los envíos a Kaggle para encontrar la ganancia máxima, que se obtuvo seleccionando 2200 clientes.

Además, `num_leaves` quedó próximo a uno de los límites de búsqueda de la BO, al igual que en los experimentos 9163 y 9166. 

Esta combinación obtuvo una ganancia intermedia respecto de las demás pruebas. Sin embargo, no es posible determinar qué aportó cada una de las modificaciones al resultado obtenido.

#### 7.2.2. Comparación de la optimización de hiperparámetros

#### Prueba 1 (9160) vs. prueba 5 (9164)

En ambas pruebas se utilizaron las modificaciones 5A y 8A. La diferencia fue la cantidad de hiperparámetros optimizados mediante BO, pasando de 5 a 11.

Se observó que la ganancia máxima en Kaggle del experimento 9160 (96864) fue superior a la del 9164 (93707). Sin embargo, el tiempo de ejecución fue menor en el 9164 (3 h frente a 4 h).

En ambos experimentos fue necesario ampliar los envíos a Kaggle, probando cantidades menores de clientes. La ganancia máxima se obtuvo seleccionando 1700 clientes en el 9160 y 1800 en el 9164.

Además, en el experimento 9164, `max_depth` y `bagging_freq` quedaron próximos a los límites de búsqueda de la BO, mientras que en el 9160 ninguno de los HP presentó esta condición.

Sin embargo, al haberse realizado una sola corrida de BO por configuración, aun utilizando un semillerio final de 30 modelos, no alcanza para determinar si esta diferencia se mantendría al repetir las pruebas.

Por lo tanto, en esta comparación y con los resultados obtenidos, aumentar de 5 a 11 los hiperparámetros optimizados simultáneamente mediante BO no permitió mejorar la ganancia en Kaggle, aunque sí se observó un menor tiempo de ejecución.

#### Prueba 6 (9165) vs. prueba 7 (9166)

En el experimento 9165, `learning_rate` quedó próximo a uno de los límites de búsqueda de la BO. Por este motivo, se decidió ampliar su rango y también el de `num_iterations`, por la relación que existe entre ambos hiperparámetros.

En la escala logarítmica utilizada por la BO, `learning_rate` pasó de (-7, -0.1) a (-9, -0.1) y `num_iterations` de (2, 10) a (2, 10.5).

Con estos cambios se obtuvo una ganancia superior en Kaggle (96697 frente a 94205), con una menor cantidad de clientes seleccionados (1700 frente a 1800).

El tiempo de ejecución aproximado del 9166 duplicó al del 9165 (8 h frente a 4 h). En ambos casos fue necesario ampliar los envíos a Kaggle, probando cantidades menores de clientes.

En el experimento 9166, `num_leaves` quedó próximo a uno de los límites de búsqueda de la BO.

En esta corrida, la ampliación de los rangos de búsqueda y el aumento del semillerio permitieron obtener una mayor ganancia en Kaggle. Sin embargo, al realizar ambos cambios al mismo tiempo, no es posible determinar cuánto aportó cada uno a la mejora.

#### Prueba 7 (9166) vs. prueba 9 (9168)

En este caso como en el experimento 9166, `num_leaves` quedó próximo a uno de los límites de búsqueda de la BO, se decidió ampliar el rango de búsqueda de este HP. 

Se utilizó como base el workflow del experimento 9166 y se amplio el rango en escala logarítica de `num_leaves` de (2, 11) a (2, 11.5). 



#### 7.2.3. Comparación del semillero