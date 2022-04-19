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

## Trabajo con pares RDD
Ejemplo de función `func` `rdd.operacion(func) => rdd.operacion(a => a + a)`

### Transformaciones 
* `reduceByKey(func)` combina valores con la misma clave.
* `groupByKey()` agrupa valores con la misma clave. Da `(Clave,[Grupo,De,Valores])`.
* `combineByKey()` combina valores con la misma clave usando un tipo de resultado diferente. Parece complejo.
* `mapValues(func)` aplica una función a cada **valor** de un *pair RDD* sin cambiar la clave.
* `flatMapValues(func)` aplica un flatMap combinado con un mapValues. Devuelve valores clave-valor por cada valor.
* `keys()` devuelve un RDD simple de las claves.
* `subtractByKey(key)` elimina los elementos que tengan una clave dada (operación de conjuntos)
* `join(key)` inner join entre dos RDD. Devuelve claves con grupos (ver groupByKey)
* `rightOuterJoin(key)`, la clave ha de estar en el primer RDD.
* `leftOuterJoin(key)`, la clave ha de estar en el segundo RDD.
* `cogroup(key)`, agrupa datos que compartan clave. Con datos vacíos.
* `values()` devuelve un RDD simple de los valores.
* `sortByKey()` ordena un RDD por clave.

### Acciones
* `countByKey()` cuenta el número de elementos por clave.
* `collectAsMap()` hace un `collect()` como un mapa para que sea más fácil buscarlo.
* `lookup(key)` devuelve los valores asociados con una clave.

## Dependencias y eficiencia
* `toDebugString` muestra el plan de ejecución de una operación. Ahí podremos ver si va a haber mucho shuffle.
* `dependencies` muestra la secuencia de dependencias que va a haber.
  * Narrow: `OneToOneDependency`, `PruneDependency`, `RangeDependency`
  * Wide: `ShuffleDependency`.

### Dependencias Narrow
* `map`
* `mapValues`
* `flatMap`
* `filter`
* `mapPartitions`
* `mapPartitionsWithIndex`
### Dependencias Wide
* `cogroup`
* `groupWith`
* `join`
* `leftOuterJoin`
* `rightOuterJoin`
* `groupByKey`
* `reduceByKey`
* `combineByKey`
* `distinct`
* `intersection`
* `repartition`
* `coalesce`

## Asuntos "avanzados"

### Mostrar un elemento por línea
* `rdd.foreach(println)` pasa por cada elemento imprimiéndolo en una línea separada.
  
### Imprimir un número de líneas
* `rdd.take(n).foreach(println)` ejecuta lo anterior pero solo en un número `n` de líneas.

### Ordenar un Pair RDD por valor
* `rdd.map(field => field.swap).sortByKey([false]).map(field => field.swap)`, siendo `[false]` lo que determina si es orden ascendente.

## Troubleshooting

### ¡Se me ha olvidado pasar el valor a variable!
¡No pasa nada! Si estás trabajando en consola, presta atención a la línea que te devuelve. Normalmente empezará algo así:
```
resXX: org.apache.spark.rdd.RDD[...]
```
Como podrás comprobar, te crea una variable automaticamente con la que puedes trabajar. `res1`, `res12`, `res48`, `res123`... puedes trabajar con esas variables. No son intuitivas y no se recomienda, pero te pueden sacar de un apuro inmediato.