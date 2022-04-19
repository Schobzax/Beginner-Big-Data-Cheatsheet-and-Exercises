# Ejercicios Spark

## Módulo 1

1. Lo primero que debemos hacer es arrancar Spark y su contexto. El contexto se crea automáticamente al arrancar Spark. Podemos comprobar la existencia de este contexto:
```
$ spark-shell
[datos varios de inicio]
scala> sc
res0: org.apache.spark.SparkContext = org.apache.spark.SparkContext@7c651424
```
Vamos a usar una carpeta compartida en la máquina virtual para la realización de estos ejercicios. En caso de no tenerla configurada, es el momento de hacerlo.

## Módulo 2

### A - Fichero plano 1

2. Vamos a crear un RDD con un archivo llamado "relato.txt" que está incluido entre los archivos a importar.
```
scala> val rel = sc.textFile("file:/home/cloudera/BIT/relato.txt")
rel: org.apache.spark.rdd.RDD[String] = file:/home/cloudera/BIT/relato.txtMapPartitionsRDD[3] at textFile at <console>:27
```
Vamos a desgranar este comando.
* `val rel`, estamos declarando una variable llamada `rel` donde guardaremos el resultado en memoria de la operación que vamos a realizar a continuación.
* `sc.textFile`, tomamos los datos de un archivo de texto en el sistema.
* `file:/home/...`, el prefijo `file:/` refiere al sistema local, que es donde se haya el archivo de texto sobre el que vamos a operar.
* Finalmente, en el resultado se ve que lo que hemos creado es un RDD de cadenas.

3. Contamos el número de líneas.
```
scala> val resultado = rel.count()
resultado: Long = 23
```
Al ser `count()` una acción, se ejecuta. La declaración de la variable no es necesaria en este caso, por cierto.

4. Ejecutamos "collect()" sobre el RDD.
```
scala> rel.collect()
res6: Array[String] = Array(Two roads diverged in a yellow wood,, And sorry I could not travel both, And be one traveler, long I stood, [...] , To where it bent in the undergrowt;, "", Then took the other, as just as fair,, [...]Somewhere ages and ages hence:, Two roads diverged in a wood, and I--, I took the one less traveled by,, And that has made all the difference.)
```
Vemos que al ser un dataset pequeño no hay ningún problema, pero teniendo en cuenta la manera en la que muestra los datos, en un dataset más grande sería algo bastante ininteligible.

5. Ahora vamos a usar "foreach" para mostrar las líneas de una manera más legible.
```
scala> rel.foreach(println)
Two roads diverged in a yellow wood,
And sorry I could not travel both
[...]
I took the one less traveled by,
And that has made all the difference.
```

### B - Fichero plano 2

Realicemos operaciones más avanzadas sobre otro archivo.

6. Creamos un archivo a partir de un fichero cualquiera de la carpeta `weblogs`.
```
scala> val log = sc.textFile("file:/home/cloudera/BIT/weblogs/2013-09-15.log")
log: org.apache.spark.rdd.RDD[String] = file:/home/loudera/BIT/weblogs/2013-09-15.log MapPartitionsRDD[7] at textFile at <console>:27
```

7. Filtramos para tomar las líneas que contengan la cadena ".jpg".
```
scala> val jpglog = log.filter(x => x.contains(".jpg")) --toma cada elemento, x, y para cada x comprueba si contiene la cadena ".jpg".
jpglog: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[8] at filter at <console>:29
```

8. Imprimimos las primeras 5 líneas.
```
scala> jpglog.take(5) -- Esto es una acción que imprime.
```

9. Ahora, en una sola línea, imprimir el número de líneas que contienen la cadena de caracteres ".jpg".
```
scala> val jpglogs2 = log.filter(x => x.contains(".jpg")).count() -- Pueden concatenarse operaciones.
jpglogs2: Long = 237
```

10. Ahora vamos a usar map para calcular la longitud de las 5 primeras líneas.
```
scala> log.map(x => x.length).take(5)
res16: Array[Int] = Array(141, 134, 140, 133, 143)
```

11. Ahora imprimimos cada palabra que contiene cada una de las primeras 5 líneas. Para esto se puede usar split().
```
scala> log.map(x => x.split(" ")).take(5)
res21: Array[Array[String]] = Array(Array[...]
```
Devuelve un array de array de palabras. Cada línea es un array con las distintas palabras.

12. En la variable "logwords" guardamos estos arrays de arrays.
```
scala> val logwords = log.map(x => x.split(" "))
```

13. Creamos un RDD de las IP de cada línea (el primer elemento). Esto lo hacemos mapeando el primer elemento de cada línea.
```
scala> val ips = logwords.map(linea => linea(0))
ips: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[22] at map at <console>:31
```
Podemos ahora imprimir las primeras cinco IPs, usar collect(), y usar foreach() para visualizar los datos de distintas maneras.

Como podremos comprobar, collect() es la menos intuitiva.

14. Usando un bucle for vamos a visualizar las primeras 10 líneas.
```
scala> for (x <- ips.take(10)){ println(x) }
```

15. Por último, guardamos el resultado de este bucle en un fichero de texto.
```
scala> ips.saveAsTextFile("file:/home/cloudera/iplist");
```

Vamos a observar los resultados.

Si nos dirigimos a la carpeta donde hemos guardado, `/home/cloudera`, observamos que `iplist`, que se ha guardado, es un directorio, que contiene dos archivos: `part-00000` y `SUCCESS`, que está vacío.

El archivo part contiene las IP, una por línea, del dataset.

### C - Conjunto de ficheros planos

16. Creamos un RDD que contenga las IP (proceso similar al que hemos realizado en el apartado B) pero de TODOS los documentos de la carpeta weblogs.
```
scala> sc.textFile("file:/home/cloudera/BIT/weblogs/*") -- Vemos que es muy sencillo, simplemente con un asterisco tomamos todos los documentos.

scala> sc.textFile("file:/home/cloudera/BIT/weblogs/*").map(linea => linea.split(" ")(0)).saveAsTextFile("file:/home/cloudera/iplistw")
```

Viendo los resultados, son similares a los del apartado anterior; excepto que debido al tamaño del dataset, hay más de un archivo part.

17. Por último, creamos un último RDD llamado htmllogs, que solo contenga la IP junto a la ID de usuario (el tercer campo de cada línea).
```
scala> val htmllogs = log.filter(_.contains(".html")).map(linea => (linea.split(" ")(0),linea.split(" ")(2)))
htmllogs: org.apache.spark.rdd.RDD[(String, String)] = MapPartitionsRDD[39] at map at <console>:29
```

Como vemos, `_` (barra baja) es un capaz sustituto para un "this" en Java, por ejemplo. Si la línea sobre la que estamos trabajando contiene html, pasa el filtro.

Además, mapeamos esas líneas filtradas, tomando la 1ª y 3ª palabras de cada.

## Módulo 4

### A - Weblogs

18. Usando MapReduce vamos a leer el número de peticiones de cada usuario. Empezamos por crear un map para conteo: (ID, 1).
```
scala> val rdd = sc.textFile("file:/home/cloudera/BIT/weblogs/*").map(line => line.split(" ")).map(words => (words(2),1))
```
Esto lo que hace es que primero carga el archivo de texto, y lo divide en un array de palabras por línea; y después toma de cada array el 3er elemento (el id de usuario) y añade el número 1 como segundo elemento para crear el par (ID, 1).

19. Ahora reducimos por clave.
```
scala> val red = rdd.reduceByKey((v1,v2) => v1+v2)
```
Como ya hemos visto en teoría, esto lo que implica es sumar los valores `v1` y `v2` si las claves son iguales.

20. Mostramos los id de usuarios y los accesos para los 10 usuarios con mayor número de accesos. Para esto hay que ordenar por valor.

```
scala> val ordo = red.map(field => field.swap).sortByKey(false).map(field => field.swap)
```
Lo que hemos hecho en esta línea es cambiar el valor por la clave, ordenar por la que ahora es la clave (en realidad el valor) y volver a darles la vuelta. De esta manera, se queda ordenado por valor.

21. Cremos un RDD donde la clave es el ID de usuario y el valor es la lista de IPs asociadas (IPs agrupadas por userID).
```
val idip = sc.textFile("file:/home/cloudera/BIT/weblogs/*").map(line => line.split(" ")).map(words => (words(2), words(0))).groupByKey()
```

22. Vamos a ponerlo bonito usando un bucle for.
```
scala> userips.take(10).foreach{
     | case(x,y) => println("ID:"+x)
     | println("IPS:")
     | y.foreach(println)
     | }
```
Lo que hacemos con esto es lo siguiente:
* Por cada elemento (clave, valores) tomamos `x` como la clave e `y` como el valor.
* Imprimimos "ID:" seguido de la clave.
* Imprimimos "IPS:" en otra línea.
* Imprimimos cada IP asociada a esa clave en una línea separada.

### B - Accounts

23. Vamos a unir los datos del archivo `accounts.csv` con los datos de weblogs usando el userid como clave.

Lo primero que hacemos es montar un RDD con los datos de accounts.
```
scala> val accounts = sc.textFile("file:/home/cloudera/BIT/accounts.csv").map(line => line.split(",")).map(account => (account(0), account))
```
Ahora realizamos la unión de ambos.
```
scala> val union = accounts.join(red) -- Lo unimos con el número de peticiones por usuario.
```
24. Vamos a crear un RDD que tome solo una serie de datos: userid, nº visitas (ambos de `red`), nombre y apellido. Para ello hay que tener en cuenta la posición de cada elemento y donde se haya:
```
scala> for(pair <- union.take(5)){println(pair._1,pair._2._2,pair._2._1(3),pair._2._1(4))}
```
Hay que tener en cuenta la estructura.
* Por un lado, tenemos el primer elemento de la pareja que es la clave, el ID de usuario.
* Por otro lado, tenemos el par con los datos de accounts y con el número de requests por usuario.
  * Dentro de los datos de accounts tenemos que coger el 4º y 5º elemento (los últimos dos)
  * Y el nº de requests es el segundo elemento del segundo elemento de la pareja inicial.

Es más sencillo de entenderlo que de explicarlo, solo hay que intentar visualizarlo.

### C - Más métodos

25. Usando keyBy, crea un RDD con los datos de las cuentas y con el código postal como clave (9º elemento).
```
scala> val accountsByPostalCode = sc.textFile("file:/home/cloudera/BIT/accounts.csv").map(_.split(",")).keyBy(_(8))
```
Aquí lo que hacemos es partir la línea por comas como antes, pero tomamos como clave el 9º elemento tal y como se nos ha indicado.

26. Ahora creamos un Pair RDD con clave el código postal y valor una lista de nombres (Apellido, Nombre) que se correspondan con ese código postal.
```
scala> val namesByPostalCode = accountsByPostalCode.mapValues(values => values(4) + "," + values(3)).groupByKey()
```
Lo que estamos haciendo aquí es operar únicamente con los valores usando mapValues, ya que la clave no cambia. Tomamos el apellido y el nombre y agrupamos por clave.

27. Ordenamos por código postal y lo ponemos en un formato legible (como en el ejercicio 22) para los primeros 5.

Usaremos sortByKey() para ordenar por clave y después ejecutaremos el bucle.
```
scala> namesByPostalCode.sortByKey().take(5).foreach{
     | case(x,y) => println("---" + x)
     | y.foreach(println)
     | }
}
```

## Módulo 4.1

### EJ1
En este ejercicio trabajaremos con el dataset shakespeare, desde HDFS. El efecto de tener el dataset en HDFS es que no es necesario enlazar con el sistema local mediante `file:`.

28. Analizaremos las palabras más repetidas en la obra de Shakespeare, mostrando las 100 palabras que más aparecen. Esto implica ordenar por valor entre otras cosas.
```
scala> val chespirwords = sc.textFile("shakespeare/*").flatMap(line => line.split(" "))
scala> val palabrasContadas = chespirwords.map(palabra => (palabra,1)).reduceByKey(_+_)
scala> val palabrasOrdenadas = palabrasContadas.map(palabra => palabra.swap).sortByKey(false).map(_.swap)
```
Aquí lo que hacemos es tomar los archivos de shakespeare y partirlos por palabras, y después montar un par contador (elemento,1) y reduciendo por clave (mediante suma), para después ordenar.

Usamos flatMap porque no necesitamos ninguna estructura compleja, solo las palabras. También podemos ver mucho uso de las barras bajas (`_`), para ver su utilidad y versatilidad como atajo para el elemento actual.

A continuación vamos a mostrar las 100 palabras que más aparecen.
```
scala> palabrasOrdenadas.take(100).foreach(println)
(,64531)
(the,25069)
(and,18793)
(to,16436)
[...]
(your,6119)
(       And,5955)
(for,5781)
[...]
(or,1792)
(       [Enter,1747)
(more,1676)
(hath,1668)
(you,,1620)
(KING,1600)
[...]
```

En los resultados se observan varios problemas con este análisis: entre las palabras más repetidas encontramos muchas palabras vacías; pronombres, artículos, proposiciones, problemas de mayúsculas, espacios, etcétera. A continuación en los siguientes ejercicios intentaremos resolverlos.

29. Lo primero que podemos hacer es crear un dataset de "stop words", lo que se denominan en inglés palabras vacías, tomando un archivo csv del que ya disponemos en HDFS.
```
scala> val stopwords = sc.textFile("stop-word-list.csv").flatMap(line => line.split()).map(word => word.replace(" ",""))
```
Vemos que hemos tenido que realizar una limpieza también porque hay espacios por medio.
* Al usar `flatMap` evitamos que la lista de palabras vacías se represente en un Array de un Array, lo cual nos ayuda en las instrucciones posteriores.

30. Después vamos a "limpiar" el dataset: no nos interesan espacios en blanco, ni palabras de una sola letra, ni distinguir entre mayúsculas y minúsculas.
```
scala> val chespirlimpio = sc.textFile("shakespeare/*").map(line => line.replaceAll("[^a-zA-Z]+"," ")).flatMap(line => line.split(" ")).map(word => word.toLowerCase)
```
* Tomamos todos los archivos.
* Reemplazamos todo lo que no sean letras mayúsculas o minúsculas por un espacio.
* Separamos por espacios.
* Quitamos las mayúsculas a todas las palabras.

Lo último que tenemos que hacer es eliminar las *stopwords* y los espacios vacíos realizar el conteo y ordenado.
```
scala> val chespirUtil = chespirlimpio.subtract(sc.parallelize(Seq(" "))).subtract(stopwords)
--- Substraemos lo que no nos interesa, incluidos espacios.
scala> val conteo = chespirUtil.map(word => (word,1)).reduceByKey(_+_).map(word => word.swap).sortByKey(false).map(_.swap)
scala> val conteoFinal = conteo.filter(word => word._1.size > 1) -- Eliminamos palabras de longitud 1 y 0
```

1.  Vamos a ver el resultado.
```
scala> conteoFinal.take(100).foreach(println)
(thou,5874)
(thy,4273)
(shall,3738)
(king,3488)
(lord,3391)
(thee,3384)
(sir,3032)
(now,2947)
(good,2929)
(come,2600)
(ll,2490) -- you'll, we'll, they'll, I'll...
(here,2433)
(enter,2427)
[...]
(gloucester,742) -- Me ha parecido curioso.
[...]
```
### EJ2 - Counts
No realizado por inexistencia del dataset.