# Ejercicios SparkSQL

## 5.1 - JSON
1. Creamos un nuevo contexto SQLContext.
```
scala> val ssc = new org.apache.spark.sql.SQLContext(sc)
```
2. Importamos los implicits que permiten convertir RDDs en DataFrames
```
scala> import sqlContext.implicits._
```
3. Cargamos el dataset "zips.json", con el comando "ssc.load" y lo visualizamos.
```
scala> val zips = ssc.load("file:/home/cloudera/BIT/zips.json","json")
scala> zips.show()
--Nos mostrará una tabla ASCII con las 20 primeras filas de la tabla
```
4. Usando la API, tomamos los códigos postales cuya población es superior a 10000.
```
scala> val grandes = zips.select("*").where($"pop" > 10000)
-- otra manera
scala> val grandes = zips.filter()
```
He optado por mostrar todos los datos, pero realmente con seleccionar tan solo el `_id` hubiera bastado.

5. Guarda esta tabla en un fichero temporal para poder ejecutar SQL contra ella: `zips.registerTempTable("zips")`

6. Realizamos el ejercicio 4 pero con SQL a pelo:
```
scala> ssc.sql("SELECT * FROM zips WHERE pop > 10000").show()
```
Nótese que usamos `show()` para no simplificar, pero también podríamos usar `collect().foreach(println)`.

7. Usando SQL, busca la ciudad con más de 100 códigos postales
```
scala> ssc.sql("SELECT city FROM zips GROUP BY city HAVING count(*) > 100").show()
+-------+
|   city|
+-------+
|HOUSTON|
+-------+
```
Como demostración, así se haría con la API: `scala> zips.select("city").groupBy("city").count().where($"count" >= 100.show()`. Esto muestra `city` y `count` con la cuenta. Podemos volver a restringirlo ejecutando un select sobre este resultado para que devuelva solo una de las columnas.

8. Usando SQL, obtén la población del estado de Wisconsin (WI)
```
scala> ssc.sql("SELECT SUM(pop), state WHERE state LIKE 'WI' GROUP BY state)
```
La población de Wisconsin es 4891769. Si no ponemos nombre, el nombre de la columna es `_c0`.

Como demostración, así se haría con la API: `scala> zips.select("pop","state").groupBy($"state").sum($"pop").where($"state" === "WI").show()`

9. Finalmente, usando SQL obtén los 5 estados más poblados.
```
scala> ssc.sql("SELECT state, SUM(pop) FROM zips GROUP BY state ORDER BY SUM(pop) DESC LIMIT 5")
```

Como demostración, así se haría con la API: `scala> zips.select("state","pop").groupBy($"state").sum("pop").orderBy(desc("sum(pop)")).limit(5).show()`
Y efectivamente, la solución es la misma.

## 5.2 - Hive

Tras realizar las configuraciones iniciales dispuestas en los ejercicios, empezamos.

1. Lo primero que tenemos que hacer es crear un HiveContext.
```
scala> val hc = new org.apache.spark.sql.hive.HiveContext(sc)
```
2. A continuación vamos a crear una base de datos y una tabla.
```
scala> hc.sql("CREATE DATABASE IF NOT EXISTS hivespark") -- si queremos que salga la salida por pantalla debemos poner la operación correspondiente, como show() por ejemplo
scala> hc.sql("CREATE TABLE IF NOT EXISTS hivespark.empleados(id INT, name STRING, age INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n'")
```
Podemos hacer distintas comprobaciones que todo está ahí con comandos como SHOW TABLES o SHOW DATABASES o DESCRIBE, que gracias a la función show() se nos muestran en un formato muy legible.

3. Ahora vamos a crear un archivo de texto que tenga unos datos legibles para csv para su posterior importación desde local. Esto se hace en otra terminal mediante nano, vi, o el método que uno prefiera. Una vez hecho eso, subimos esos datos a la tabla que hemos creado con el siguiente método HiveQL:
```
scala> hc.sql("LOAD DATA LOCAL INPATH '/home/cloudera/empleado.txt' INTO TABLE hivespark.empleados")
```
Nuevamente, usando show() podemos ver que los datos se han cargado correctamente:
```
scala> hc.sql("SELECT * FROM empleados").show()
+----+-------+---+
|  id|   name|age|
+----+-------+---+
|1201|nombre1| 25|
|1202|nombre2| 28|
|1203|nombre3| 39|
|1204|nombre4| 23|
|1205|nombre5| 23|
+----+-------+---+
```
4. Comprobando que los datos de Hive también están accesibles desde Hive:
```
hive> select * from hivespark.empleados;
OK
1201    nombre1 25
1202    nombre2 28
1203    nombre3 39
1204    nombre4 23
1205    nombre5 23
Time taken: 0.738 seconds, Fetched: 5 row(s)
```

## 5.3 - DataFrames (Partidos)

Vamos a crear un DataFrame de forma explícita.

1. Lo primero es crear un contexto SQL.
```
scala> val ssc = new org.apache.spark.sql.SQLContext(sc)
```

2. Ahora debemos importar los implicits que nos van a servir para convertir RDDs en DataFrames y Rows.
```
scala> import sqlContext.implicits._
scala> import org.apache.spark.sql._
```

3. Ahora cargamos el dataset "DataSetPartidos" en una variable que nos sirva para generar un esquema.
```
scala> val ruta = "/home/cloudera/BIT/DataSetPartidos.txt"
scala> val dataSetPartidos = sc.textFile(ruta)
```

4. Vamos a crear una variable que contenga el esquema de estos datos, que ya se nos da explícito en el documento de los ejercicios, y creamos un schema a partir de esta string.
```
scala> val esquemaPartidos = "idPartido::temporada::jornada::EquipoLocal::EquipoVisitante::golesLocal::golesVisitante::feha::timestamp"
scala> val schema = StructType(esquemaPartidos.split("::").map(fieldName => StructField(fieldName, StringType, true)))
```

5. Convertimos las filas del RDD a Rows y le aplicamos el esquema finalmente.
```
scala> val rowRDD = dataSetPartidos.map(_.split("::")).map(p => Row(p(0),p(1),p(2),p(3),p(4),p(5),p(6),p(7),p(8).trim))
scala> val dataFramePartidos = sqlContext.createDataFrame(rowRDD,schema)
```

6. Por último, registramos el DataFrame como una Tabla.
```
scala> dataFramePartidos.registerTempTable("partidos")
```

Ya estamos listos para realizar consultas sobre esta tabla.
```
scala> val results = sqlContext.sql("SELECT temporada, jornada FROM partidos").show()
```
Nótese que el resultado es accesible como DataFrame y como RDD normal y corriente:
```
scala> results.map(t => "Name:" + t(0)).take(10)
```

7. Registremos el récord de goles como visitante en una temporada del Oviedo.

*Nota: Es recomendable el uso de la interfaz API de DataFrame en contraposición a usar sentencias SQL en el contexto; pero dado que el esquema provisto es todo Strings, esto solo funciona de la segunda manera en tanto que el propio SQL infiere cuando quieres realizar una suma, por lo visto.*
```
scala> val recordOviedo = sqlContext.sql("select sum(golesVisitante) as goles, temporada from partidos where equipoVisitante LIKE 'Real Oviedo' group by temporada order by goles desc")
```

8. ¿Quién ha estado más temporadas en 1ª División, el Sporting o el Oviedo?
```
scala> val temporadasOviedo = sqlContext.sql("SELECT COUNT(DISTINCT(temporada)) FROM partidos WHERE equipoLocal LIKE '%Oviedo%' OR equipoVisitante LIKE '%Oviedo%'")
scala> val temporadasSporting = sqlContext.sql("SELECT COUNT(DISTINCT(temporada)) FROM partidos WHERE equipoLocal LIKE '%Sporting%' OR equipoVisitante LIKE '%Sporting%'")
```
El Sporting ha estado más temporadas, con 45; frente a las 32 del Oviedo. Realmente no podemos cotejar si los partidos son de 1ª división o de 2ª al haber registros sin distinción de ambas.

Ahora es hora de jugar con este dataset cuanto se quiera. Por ejemplo, cabe destacar que el Real Jaén ha disputado partidos en 1ª o 2ª división entre los años 1976 y 1979, en la temporada 1997-1998, de los años 2000 a 2002, y por último en 2013-2014; un total de 7 temporadas en 1ª o 2ª división.
```
scala> val temporadasJaen = sqlContext.sql("select distinct(temporada) from partidos where equipoLocal LIKE 'Jaen' OR equipoVisitante LIKE 'Jaen' ORDER BY temporada")
scala> temporadasJaen.show() -- Elegimos la vertiente show porque tenía plena confianza en que eran menos de 20 temporadas, y así era.
```

Otros datos interesantes es que el Linares ha estado en 1ª o 2ª división en la temporada 1973-1974 y entre los años 1980 y 1984. *(Este último se deja como ejercicio para el lector)*

## 5.4 - Afianzar (Simpsons)

El objetivo de este ejercicio es dado el dataset sobre puntuaciones de episodios de los simpson, crear un diagrama lineal en el que se vea gráficamente la puntuación media de cada temporada.

1. Lo primero que hacemos tras cargar el archivo en hdfs es cargar la librería que nos permitirá dibujar una gráfica tal. Antes hay que realizar un par de configuraciones externas de compatibiliad:
```
$ echo 'export JAVA_TOOL_OPTIONS="-Dhttp.protocols=TLSv.1.2"' >> ~/.bashrc
$ source ~/.bashrc
$ spark-shell --packages com.databricks:spark-csv_2.10:1.5.0,org.sameersingh.scalaplot:scalaplot:0.0.4 // Importante hacerlo sin espacios.
```

2. A continuación y una vez cargadas las librerías, importamos las bibliotecas que hemos descargado.
```
scala> import org.sameersingh.scalaplot.Implicits._; import org.sameersingh.scalaplot.MemXYSeries; import org.sameersingh.scalaplot.XYData; import org.sameersingh.scalaplot.XYChart; import org.sameersingh.scalaplot.gnuplot.GnuplotPlotter // Como se puede ver estoy usando ; para ejecutar más de un comando en la misma línea. Esto no es del todo recomendable pero me apetecía mostrar que se puede hacer.
```

3. Ahora pasamos el archivo CSV a un DataFrame sobre el que podamos trabajar. Para ello tenemos que usar la biblioteca CSV que hemos importado.
```
scala> val df = sqlContext.read.format("com.databricks.spark.csv").option("header","true").option("inferSchema","true").load("simpsons.csv")
```
Como podemos comprobar, hemos añadido un par de opciones que determinan que hay una fila cabecera, y que queremos que infiera la estructura a partir de la misma.

4. Ahora tenemos que ajustar algunos datos para poder trabajar con ellos:
   * "Season" debería ser INT en vez de STRING.
   * El DF debe estar ordenado de forma ascendente por temporada.
   * Hemos calculado la media de puntuaciones también.
```
scala> val datos = df.withColumn("Season",col("Season").cast("int")).groupBy("Season").avg("imdb_rating").orderBy("Season")
```

5. Para poder trabajar de mejor manera con estos datos vamos a pasar el DF a un pairRDD, siendo la clave la temporada y el valor la puntuación media.
```
scala> val rdd = datos.rdd.map(line => (line.getInt(0), line.getDouble(1)))
```

6. Ahora para poder dibujar la gráfica vamos a separar la clave y el valor en variables x e y.
```
scala> val x = rdd.map({case(key,value) => key.toDouble})
scala> val y = rdd.map({case(key,value) => value})
```

7. Lo siguiente es configurar la salida para obtener una imagen, pero antes de eso tenemos que instalar una herramienta que no está disponible en nuestra distribución.

En este caso, lo que podemos hacer es instalarla manualmente a través de un paquete tar.gz ubicado en la [página de descarga de gnuplot](https://sourceforge.net/projects/gnuplot/files/gnuplot/5.4.2/gnuplot-5.4.2.tar.gz/download).

La secuencia de comandos que debemos ejecutar para configurar e instalar esto (previo guardado del archivo descargable en nuestro sistema de archivos local) es el siguiente: nos dirigmos a la carpeta donde se haya el archivo .tar.gz que nos hemos descargado y lo descomprimimos.

Entramos en la carpeta, y una vez allí, ejecutamos lo siguiente:
```
$ ./configure
$ sudo make
$ sudo make install
```
Una vez realizados estos tres arduos pasos, el programa `gnuplot` estará instalado y dispuesto para su uso.

8. Ahora terminamos de configurar el archivo que nos llevará a la imagen, nuevamente en la consola de spark.
```
scala> val series = new MemXYSeries(x.collect(), y.collect(), "puntuacion")
scala> val data = newXYData(series)
scala> val chart = new XYChart("Media de puntuación de episodios de Los Simpsons durante sus temporadas",data)
scala> output(PNG("docs/","test"),chart)
```

Ahora en la carpeta `/home/cloudera/docs` se ha creado un archivo que usando gnuplot permitirá su visualización. Lo único que nos queda por hacer es configurarlo correctamente, puesto que visualizarlo como PNG requiere una librería adicional.

9. Entraremos al archivo "test.gpl" que acabamos de crear en la carpeta docs, y cambiamos lo siguiente:
```
# Chart settings
set title "Media de puntuación de episodios de Los Simpsons durante sus temporadas"
unset key
set terminal png enhanced

set terminal png enhanced

set output "test.png"
# XYChart settings
```
por
```
# Chart settings
set title "Media de puntuación de episodios de Los Simpsons durante sus temporadas"
unset key
set terminal canvas

set output "test.png"
# XYChart settings
```
Guardamos el archivo.

10. Por último, entramos en gnuplot desde otra terminal usando el comando `gnuplot` y una vez allí cargamos el fichero con `load` y la ruta absoluta:
```
gnuplot> load "/home/cloudera/docs/test.gpl"

Y eso nos mostrará una gráfica muy bonita en ASCII. Lamentablemente no conozco manera de pasarlo a otro formato, pero ahí está.
```