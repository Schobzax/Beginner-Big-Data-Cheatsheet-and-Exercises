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