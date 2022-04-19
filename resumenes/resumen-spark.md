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

## Topología

Algunos problemas pertinentes al uso de Spark tienen que ver con la latencia. Es importante no perder tiempo en operaciones de I/O. En esto Spark es muy eficiente: realiza todas las operaciones en la propia memoria, sin necesidad de guardar los datos intermedios, y después hace la operación de I/O.

Un truco para aumentar la eficiencia (que no hemos estado usando) es persistir los RDD. Eso nos ahorra tener que ejecutar la misma serie de transformaciones todas las veces.

### Caché y persistencia

Hay distintos modos de guardar los datos, que pueden llevar a un almacenamiento ineficiente si no se realizan correctamente: pueden guardarse en memoria, en memoria y disco, o solo en disco; y puede guardarse de forma serializada en los dos primeros.

`Cache()` permite persistir los datos; usa el sistema de persistencia por defecto (memoria); mientras que `persist()` permite escoger el tipo de persistencia:
* `MEMORY_ONLY` - Desecha lo que no entre.
* `MEMORY_ONLY_SER` - Almacena representación serializada
* `MEMORY_AND_DISK` - Si el RDD es demasiado grande cae a disco.
* `MEMORY_AND_DISK_SER` - Almacena representación serializada en memoria. Si el RDD es demasiado grande cae a disco.
* `DISK_ONLY` - Solo en disco.

### Funcionamiento interno

Hay **nodos *worker*** y **nodo *master***. El nodo master controla el **cluster**.

Spark usa una **arquitectura maestro/esclavo**. Durante la ejecución, crea un **gráfico lógico de operaciones** que al iniciar el programa se convierte en un **plan físico de ejecución**.

Cada etapa está formada de tareas (*tasks*)

**Task**: Unidad de trabajo más pequeña de Spark. Todas las tareas van al clúster, donde se dividen entre los ejecutores (workers) disponibles.

**Driver**: Proceso donde se ejecuta el main(), se crea el sparkContext, RDDs y se ejecutan transformaciones y acciones.

Al iniciar el trabajo, los nodos *worker* se registran en el driver para que este sepa lo que tiene disponible. El driver programa los *tasks* en el nodo adecuado, y los *worker* las ejecutan mediante el *executor*.

**NOTA IMPORTANTE**: Los *nodos* son *worker* y *driver*, mientras que los programas son *context* y *executor* (?).

El **driver** y el **worker** se comunican mediante el **cluster manager** (CM).

#### ***Proceso de ejecución***
1. Se lanza una aplicación mediante *spark-submit* (para lanzar scripts)
2. Éste lanza el driver e invoca el *main()*.
3. Se conecta al cluster manager y pide recursos para lanzar los *Executors*.
4. El CM lanza los *Executors*.
5. El driver ejecuta la aplicación del usuario y va enviando las tareas a los *Executors* en forma de *Tasks* según las transformaciones y acciones que vaya habiendo.
6. Los *Executors* ejecutan las *Tasks*.
7. Termina el *main()* y los *Executors*, liberando los recursos.

### Funcionamiento externo

***spark-submit*** funciona de una manera tal que así:

```
bin/spark-submit [options] <app jar | python file> [app options]
```
Y de esta manera se despliegan aplicaciones mediante ***spark-submit***.

Hay varios CM disponibles: *standalone*, *Hadoop Yarn*, *Apache Mesos*...

#### Standalone
El más sencillo, solo el CM. Un *master* y varios *workers*, cada uno con su RAM y CPU asignadas al lanzar la aplicación. En cada nodo hay que instalar *Spark* e indicar qué nodos son *Master* y cuales son *Slave* junto con la configuración. Es el recomendado para ejecutar Spark en local por su sencillez.

#### Yarn
Introducido en Hadoop 2.0, muy útil porque permite acceso a los datos en HDFS. Fácil de usar: los administradores del cluster configuran el uso de Yarn y el usuario final solo tiene que ejecutar las aplicaciones. A través del shell solo tenemos que indicar `--master yarn`. Si usamos YARN disponemos de un número fijo de *Executors*, aunque se pueden modificar por parámetro. Lo mismo ocurre con memoria y núcleos.

#### Mesos
Ofrece dos modos para compartir recursos entre ejecutres:
* "*Fine-Grained*", que asigna de forma dinámica el número de CPUs a cada ejecutor (más de una aplicación)
* "*Coarse-Grained*", un número fijo de CPUs a cada ejecutar y hast que termina no se desasignan aunque el ejecutor esté parado.
Se indica mediante `--master mesos`.

#### Recomendaciones
* Se recomienda empezar con Spark Standalone para hacer pruebas, es más sencillo y rápido de configurar.
* YARN viene preinstalado por defecto con Hadoop.
* Mesos es recomendable para aplicaciones interactivas multiusuario.

### WebUI
El Standalone Cluster Manager de Spark tiene una interfaz web para ver la ejecución en proceso, localizada en spark://``<url>``:7077. La URL suele ser localhost si la ejecución es local.

## Pair RDD
Consiste en trabajar con pares clave-valor.

En Spark, los pares clave-valor se llaman *Pair RDDs*. Permiten operar por clave en paralelo y reagrupar los datos en base a las mismas.

### Creación
La forma más sencilla de crear un *Pair RDD* es mapeando una clave y un valor.
```
val pairRDD = rdd.map(line => (line.key, line.value))
```

A partir de ahí podremos operar con él y usar las operaciones de un *Pair RDD*.

**Importante: Las operaciones *Pair RDD* no sirven para *RDD* regulares, pero sí al contrario.

En muchos sentidos, las operaciones por clave son "atajos" o "abreviaturas" de una o más operaciones con RDD regulares, pero que nos dan mucha potencia.

## Pair RDDs: Joins y Shuffling

### Join

Un *join* solo es válido para *Pair RDD* porque requiere del uso de claves. Es muy parecido al uso que se le da en BBDD.

Los tipos son los mismos que con operaciones de conjuntos clásicas: inner join, outer join, substract, union, intersection, etcétera. Todas las operaciones de conjuntos.

**Importante: Las claves han de ser del mismo tipo. Los valores pueden ser de cualquier tipo.**

### Shuffling

Esta fase intermedia entre el `Map` y el `Reduce` no es 100% evitable. Consiste en el movimiento de datos de un nodo a otro y **aumenta** significativamente la **latencia**.

Algunas maneras de evitarlo consisten en usar operaciones como **`reduceByKey`**, que es más eficiente que agrupar y después reducir.

Esencialmente esto tiene como objetivo reducir la cantidad de datos que se envían a través de la red, para reducir la latencia de la fase *Shuffle*, mejorando así la eficiencia.

## Partitioners

¿Cómo sabe Spark dónde poner los datos? La respuesta está en los *Partitioners*.

Un Partition siempre se queda en un solo nodo, y cada máquina del cluster tiene una o más particiones. El número total es configurable: normalmente refiere al número total de núcleos de ejecución (cores * nodos)

El partitioning solo se realiza en Pair RDDs, porque dependen de las claves para realizarse correctamente.

Hay dos maneras de particionar: *Hash partitioning* y *Range partitioning*.

### Hash partitioning
Usa una función hash para determinar dónde ponerlo. La clave aquí está en tener una buena función hash. Sin embargo, debido a que dependemos del valor de la clave, este método puede no ser del todo eficiente o equitativo.

### Range partitioning
Divide las Partition según el valor ordenado de las claves. Las claves en el mismo rango estarán en la misma máquina.

Requiere hacer ciertas restricciones sobre las claves (p. ej.asumimos que es positiva y que su valor está entre 0 y 100). De esta manera, determinamos una serie de rangos que equilibren.

### Ejecución
Esto se hace mediante el método **`partitionBy`** sobre un RDD, pasándole un Partitioner explícito.

Ejemplos:
```
val pairs = rdd.map(p => (p.clave, p.valor))

val rangepart = new RangePartitioner(8,pairs) --se indica el número y sobre qué se va a ejecutar
val partitioned = pairs.partitionBy(rangepart).persist()
```

Es importante persistir el objeto para no reejecutar el Partitioner en cada iteración.

Range Partitioner requiere especificar el número de particiones y un PairRDD con Keys ordenadas.

#### Operaciones
Hay ciertas operaciones que nos devuelven y propagan un partitioner (`cogroup`, `join` y otras de conjuntos, `foldByKey`, `combineByKey`, `groupByKey` y otras por clave, `mapValues`, `flatMapValues`, etcétera)

El resto no produce partitioners porque dan la posibilidad de modificar la clave (a propósito o por accidente), por ejemplo la operación `map`.

### Dependencias Wide vs.Narrow
Una RDD consta de cuatro partes:
* Particiones: partes atómicas de un dataset.
* Dependencias: modelan las relaciones entre RDDs y particiones.
* Funciones: Función que le pasamos a una operación para transformar el dataset del padre al hijo.
* Metadatos: schema de particionado y almacenamiento de los datos.

* **Dependencias *narrow***: Cuando cada partición del padre es usada como mucho por una partición del hijo (no requieren shuffling). Ejemplos: `map, mapValues, flatMap, filter, mapPartitions, mapPartitionsWithIndex`
* **Dependencias *wide**: Cuando una partición del padre es usada por >1 particiones del hijo. (requieren shuffling). Ejemplos: `cogroup, groupWith, join, leftOuterJoin, rightOuterJoin, groupByKey, reduceByKey, combineByKey, distinct, intersection, repartition, coalesce`

**A más *wide*, más tráfico de red.**

## SparkSQL

SparkSQL es una interfaz pensada para trabajar con datos estructurados (con schema) y semiestructurados.

Permite la carga de datos estructurados, el procesamiento de consultas relacionales desde dentro de Spark y desde aplicaciones externas con conexiones a Spark y la integración con Java/Python/Scala y la posibilidad de hacer joins con RDDs y tablas y otras cosas.

**Es una especie de "librería"**.

Sirve para optimizar las consultas a datos estructurados. De normal, Spark no conoce dicha estructura, por lo que no es posible optimizar. Esto nos permite, entre otras cosas, filtrar antes de agrupar y evitar los productos cartesianos.

**DataFrame**: RDDs de registros distribuidos con esquema conocido. Es una **abstracción** que representa una **tabla** de une esquema de bases de datos.

No están tipados (`RDD[String]` vs. `DataFrame`).

### Trabajo

Para iniciar el trabajo con Spark SQL necesitamos una **Sesión Spark (Spark Session)**, que es esencialmente un SparkContext para SparkSQL.

Para ello hay que ejecutar `import org.apache.spark.sql.SparkSession`.

### Creación

Los DataFrames pueden crearse de dos maneras: a partir de un RDD creando un esquema explícito o infiriendo uno implícito; o leyendo datos desde un origen con esquema (bases de datos Hive, JSON, XML...)

#### **Inferencia**
Dado un PairRDD, puede crearse un DataFrame a partir de un esquema inferido automáticamente con el método **`toDF`**. Sin argumentos asigna `_1`, `_2`,... a las columnas.

#### **Explícita**
1. Creamos un RDD con las filas del RDD original. (Rows)
2. Creamos un esquema representado por un *StructType* que refleja la estructura del RDD *Rows*.
3. Aplicamos el esquema a *Rows* a través del método `createDataFrame`, provisto por el SparkSession.
4. Registramos el DF como tabla para operarla con SQL tradicional con el método `registerTempTable()`.

#### **Lectura de datos**
Mediante el propio objeto SparkSession podemos leer datos [semi]estructurados mediante el método `read`, infiriendo su esquema en el proceso.
`val df = spark.read.json("asdf.json")`

### Uso
Para usar SQL en SparkSQL, lo primero que hay que hacer es declarar el DataFrme como vista temporal SQL. Esto se hace mediante el método `createOrReplaceTempView()`, que nos sirve además para darle un nombre en SQL.
```
exampleDF.createOrReplaceTempView("examplename")
val result = spark.sql("SELECT * FROM examplename WHERE condicion")
```
Siendo `spark` una `SparkSession`.

Por otro lado, SparkSQL tiene su propia API relacional para usar los DataFrames, que permiten optimizar las consultas.

Los tipos de datos deben ser importados: `import.org.apache.spark.sql.types._`.

Como estamos usando Spark, las operaciones relacionales se optimizan, por ejemplo, filtrando antes de un join.

La API tiene métodos similares a `SELECT`,`WHERE`,`LIMIT`,`GROUP BY`,`ORDER BY`, y `JOIN`.

La API puede verse en el [cheatsheet](../apuntes-snippets/spark-cheatsheet.md#sparksql).

### Errores
Al no ser tipados el código compilará; pero si hacemos una consulta sobre una columna que no existe, el error será en tiempo de ejecución (*runtime exception*).

Una posible solución para esto son los **DataSets**.

### DataSets

Facilitan el tipado y dan acceso a operaciones con RDDs. Un Dataset es esencialmente una "colección distribuida de datos tipados", siendo un Dataframe un tipo específico de Dataset.

Un DS es la unión de RDD y DF.

**Nos da la posibilidad de mezclar ambas APIs**.

**Importante**: Las operaciones tipadas han de realizarse sobre columnas tipadas, y no al revés. En conclusión: al transformar un DF a un DS tenemos que pasar información explícita de tipos. Para más información sobre qué operaciones son tipadas y cuáles no, ver el [cheatsheet](../apuntes-snippets/spark-cheatsheet.md#datasets).

### Resumiendo
* **¿Cuándo usar RDD?** Cuando tengamos datos desestructurados, con operaciones de muy bajo nivel o con datos complejos no serializables con Encoders.
* **¿Cuándo usar DataFrames?** Cuando tengamos datos estructurados o semiestructurados y queremos el mejor rendimiento y optimización por parte de Spark.
* **¿Cuándo usar DataSets?** Cuando tengamos datos estructurados o semiestructurados y queremos tipado total de datos, trabajar con una API funcional y un buen rendimiento pero sin necesitar que sea el mejor.

### Hive
Puede usarse SparkSQL con Hive, accediendo a las tablas Hive, a UDF (User Defined Functions), a SerDes (formatos de serialización y deserialización) y a HiveQL.

## SparkStreaming