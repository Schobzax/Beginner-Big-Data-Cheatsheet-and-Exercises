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