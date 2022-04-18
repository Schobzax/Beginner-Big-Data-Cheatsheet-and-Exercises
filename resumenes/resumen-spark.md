# Resumen Spark

## Introducción

**Spark** es un motor multilenguaje para computación en clúster, especializándose en análisis de datos. Durante este curso se usará junto al lenguaje **Scala**.

Es entre 10 y 20 veces más rápido que MapReduce.

### El uso de Scala
Se usa Scala con Spark porque Spark como tal está escrito en Scala, facilitando así el desarrollo. Además, Scala ayuda mucho con funciones de paralelismo sin necesidad de librerías (como Thread en Java). Spark distribuye automáticamente el trabajo entre los nodos.

Mejor rendimiento, mejor tiempo de ejecución, mejor interactividad.

Además, los programas se escriben en muchísimas menos líneas.

### RDD
Spark implementa **RDD** (*Resilient Distributed Dataset*), un modelo de datos paralelos distribuidos.

### Hadoop vs. Spark
* Spark es más rápido que Hadoop.
* Hadoop es más rígido que Spark (primero un Map y luego un Reduce, sin poder combinar las funciones)
* Mejor experimentación con los datos (mejor interactividad).

#### Consideraciones de rendimiento

El *shuffle* (la parte intermedia entre el *Map* y el *Reduce* de Hadoop) tiene un coste muy alto en rendimiento: consiste en una copia de los datos intermedios en disco.

* Leer de disco es 100 veces más lento que leer de memoria.
* Leer de red es 1000000 veces más lento que leer de memoria.

Spark almacena en memoria, y Hadoop lo hace en disco.

### Herramientas
* **Spark Core**: Funcionalidad básica (programación de tareas, gestión de memoria, recuperación ante fallos, interacción con sistemas de almacenamiento, etc). Aquí se aloja la API para trabajar con RDD.
* **Spark SQL**: Paquete de Spark orientado a trabajar con SQL o HQL. Soporta distintos tipos de datos, Hive, Parquet, JSON. Permite intercalar comandos de manipulación de RDDs (potente).
* **Spark Streaming**: Procesamiento de streams de datos en tiempo real, como weblogs o serverlogs mientras se van generando.
* **Mllib**: Librería con funcionalidades de Machine Learning. Clasificación, Regresión, Clustering, Filtros Colaborativos, Evaluación de modelos e Importacion de datos.
* **GraphX**: Manipulación de grafos y ejecución de computación paralela sobre grafos.

### Modo de funcionamiento
Existen nodos trabajadores que comparten memoria. Estos nodos trabajadores (`workers`) trabajan en paralelo.

Lo datos se dividen en los distintos nodos, que operan independientemente sobre los datos en paralelo y se combinan los resultados al terminar.

**PROBLEMA**: La latencia del tráfico de red.

* En alto nivel: se trabaja con un *driver*. Lanza operaciones en paralelo en un clúster. Este *driver* contiene la función *main* y define los *datasets*, y les aplica la función.
* Los *drivers* acceden a Spark mediante el objeto *SparkContext (sc)*, la conexión con el clúster. Al trabajar a través del shell, éste se crea automáticamente.
* Para ejecutar las operaciones, el driver usa un grupo de nodos llamados *Executors*.

## Trabajando con RDDs

Un RDD es:
* **Resiliente** (tolerante a fallos): permite la recuperación de datos en memoria.
* **Inmutable**: no podemos realizar modificaciones sobre el RDD. Para modificarlo, crearemos uno nuevo.
* **Distribuido**: Cada RDD se divide en múltiples particiones automáticamente. Puede ejecutarse en diferentes nodos.
* Un **dataset**: Conjunto de datos.