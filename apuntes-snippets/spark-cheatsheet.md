# Spark Cheatsheet

* Conexión a Spark Shell: `spark-shell`
* Datos de contexto: `sc`

**Importante: Existe autocompletado (tab)**.

* Uso de operaciones: `val rddresult = rdd.operacion(argumentos)`

## Transformaciones

### Iniciales 
* `sc.parallelize` : transforma una *Scala collection* en un *RDD*.
* `sc.textFile` : lee un fichero de texto desde HDFS o Local y lo transforma en un *RDD[String]*.

### Sobre RDD
* `map()` : genera un nuevo RDD aplicando una función sobre cada línea del RDD.
* `filter()` : genera un nuevo RDD a partir de las líneas del RDD que cumplen un filtro (esencialmente un SQL WHERE)
* `flatMap()` : igual que `map()`, pero en vez de devolver un resultado compuesto (array de arrays) "aplana" las dimensiones y devuelve un solo array con todo.
  * Ejemplo: `("hola mundo","adios mundo")`. Donde map devolvería `[["hola","mundo"]["adios","mundo"]]`; flatMap devolvería `["hola","mundo","adios","mundo"]`.
* `distinct()` : devuelve los valores distintos.
* `union()` : devuelve la unión de dos rdd.
* `intersection()` : devuelve la intersección de dos rdd.
* `subtract()` : devuelve la resta de dos rdd (operaciones de conjuntos).
* `cartesian()` : devuelve el producto cartesiano de dos rdd.
* `sample(withReplacement, fraction)` : devuelve una muestra porcentual (fraction) de un rdd con o sin reemplazo.

## Acciones
* `reduce()` : opera sobre dos elementos de un RDD y devuelve un resultado de igual tipo. Por ejemplo, una suma: `val sum = rdd.reduce((x,y) => x + y)`.
* `foldLeft()` : entrada y salida pueden ser de tipos distintos.
* `count()` : cuenta el número de elementos(filas) de un RDD.
* `take(n)` : toma un array de `n` elementos.
* `countByValue()` : el número de veces que ocurre cada valor en el rdd. Devuelve pares.
* `top(n)` : devuelve los mayores `n` elementos del rdd.
* `takeOrdered(n)(orden)` : devuelve `n` elementos ordenados por orden.
* `takeSample(withReplacement, n)` : devuelve `n` elementos aleatorios.
* `fold(zero)(func)` : igual que reduce pero con el valor cero.
### Guardado
* `collect()` : devuelve el dataset completo, un array de todos los elementos. Es esencialmente un dump. Solo recomendable para datasets pequeños.
* `saveAsSequenceFile()` : guarda el resultado en un archivo en un sistema de almacenamiento. Recomendable para datasets medianos, grandes, muy grandes, etc.
* `saveAsTextFile(file)` : guarda el resultado en un archivo de texto.


## Otros
* `RDD.persist()` : guarda en memoria un conjunto de datos para evitar ejecutar la sucesión de transformaciones que ha llevado hasta él repetidas veces. Recomendable tenerla en mente para persistir datos intermedios de vez en cuando (si la secuencia de transformaciones acaba siendo muy grande).

## Asuntos "avanzados"

### Mostrar un elemento por línea
* `rdd.foreach(println)` pasa por cada elemento imprimiéndolo en una línea separada.
  
### Imprimir un número de líneas
* `rdd.take(n).foreach(println)` ejecuta lo anterior pero solo en un número `n` de líneas.