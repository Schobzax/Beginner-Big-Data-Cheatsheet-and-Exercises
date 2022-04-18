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

### Operaciones
Sobre un RDD se pueden realizar:
* **Transformaciones**: Crean un nuevo RDD.
* **Acciones**: Devuelven un resultado (no otro RDD).

Esta diferencia es muy importante en términos de eficiencia. Se explicará más adelante.

Nos comunicamos con Spark mediante el SparkContext. Tiene diversos métodos para crear RDDs a partir de datos locales.

### Transformaciones
* Son perezosas (no se procesan sobre RDD hasta ejecutar una acción). No se computan inmediatamente.

### Acciones
* No son perezosas, son inmediatamente computadas.

Esta diferencia clave entre transformaciones y acciones es lo que permite a Spark reducir el tráfico de red. Una vez que Spark ve la cadena de transfomaciones, procesa solo los datos necesarios para el resultado de la acción.

Hay que insistir: **No se ejecuta nada hasta que no se realiza una acción**.

#### Una analogía incorrecta pero que nos ayudará a entender:
¿Qué significa procesar solo los datos necesarios? Si por ejemplo, tras ejecutar todas las transformaciones solo se sacan las primeras 10 líneas, solo se guardarán en memoria las primeras 10 líneas. Si Spark ejecutara todas las transformaciones en vez del resultado, tendría que cargar todo el dataset para luego sacar las primeras 10 líneas.

#### Persistencia:
Si se van a ejecutar varias veces sobre los mismos datos puede ser interesante realizar una operación `RDD.persist()`,, para no tener que ejecutar la secuencia de transformaciones repetidas veces. De esta manera los datos se guardan en memoria, particionados a lo largo del clúster para su reutilización.

