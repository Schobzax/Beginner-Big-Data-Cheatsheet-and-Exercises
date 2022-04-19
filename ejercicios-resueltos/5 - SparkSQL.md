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

## 5.3 - DataFrames