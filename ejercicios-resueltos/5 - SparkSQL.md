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

## 5.3 - DataFrames

## 5.4 - Afianzar