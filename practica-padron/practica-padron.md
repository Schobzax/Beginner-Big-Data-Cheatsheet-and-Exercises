# 1. Práctica Padrón

Aquí se va a ver paso a paso la realización de la práctica del padrón de Madrid resuelta.

## Consideraciones iniciales
Los ejercicios se realizan a partir del .csv provisto [en esta dirección](https://datos.madrid.es/egob/catalogo/200076-1-padron.csv).

## 1. Tablas Hive
Esta parte se está realizando en la máquina virtual de Cloudera provista en cursos anteriores (de hecho, se usa en el resto de este repositorio)-

1. **Crear base de datos "datos_padron"**.

Lo primero es abrir hive con el comando `hive` y dentro ejecutamos lo siguiente:
```
hive> create database datos_padron;
OK
Time taken: 0.054 seconds
```

2. **Crear la tabla de datos padron_txt con todos los datos del fichero CSV y cargar los datos mediante el comando LOAD DATA LOCAL INPATH. La tabla tendrá formato texto y tendrá como delimitador de campo el caracter ';' y los campos que en el documento original están encerrados en comillas dobles '"' no deben estar envueltos en estos caracteres en la tabla de Hive (es importante indicar esto utilizando el serde de OpenCSV, si no la importación de las variables que hemos indicado como numéricas fracasará ya que al estar envueltos en comillas los toma como strings) y se deberá omitir la cabecera del fichero de datos al crear la tabla.**

```
hive> CREATE EXTERNAL TABLE IF NOT EXISTS padron_txt (
    cod_distrito INT,
    desc_distrito STRING,
    cod_dist_barrio INT,
    desc_barrio STRING,
    cod_barrio INT,
    cod_dist_seccion INT,
    cod_seccion INT,
    cod_edad_int INT,
    EspanolesHombres INT,
    EspanolesMujeres INT,
    ExtranjerosHombres INT,
    ExtranjerosMujeres INT
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde' -- Esta propiedad hace que al describir la tabla todo sea string
WITH SERDEPROPERTIES ('separatorChar' = '\073', 'quoteChar' = '\"', "escapeChar = '\\') -- \073 es el código de ';'
STORED AS TEXTFILE
TBLPROPERTIES ("skip.header.line.count"="1");

hive> LOAD DATA LOCAL INPATH "/home/cloudera/share/Rango_Edades_Seccion_202204.csv" INTO TABLE padron_txt;
```

1. **Hacer trim sobre los datos para eliminar los espacios innecesarios guardando la tabla resultado como padron_txt_2. (Este apartado se puede hacer creando la tabla con una sentencia CTAS).**

```
hive> CREATE TABLE padron_txt_2 AS SELECT
    > cod_distrito,
    > trim(desc_distrito) as desc_distrito,
    > cod_dist_barrio,
    > trim(desc_barrio) as desc_barrio,
    > cod_barrio,
    > cod_dist_seccion,
    > cod_seccion,
    > cod_edad_int,
    > espanoleshombres,
    > espanolesmujeres,
    > extranjeroshombres,
    > extranjerosmujeres
    > FROM padron_txt;
```
En realidad no es necesario hacer los trim de los valores enteros, pero me apetecía.

4. **Investigar y entender la diferencia de incluir la palabra LOCAL en el comando DATA.**

La palabra `LOCAL` refiere a que carguemos un archivo del sistema local, mientras que si no lo ponemos, el archivo se encontrará en el sistema hdfs.

5. **En este momento te habrás dado cuenta de un aspecto importante, los datos nulos de nuestras tablas vienen representados por un espacio vacío y no por un identificador de nulos comprensible para la tabla. Esto puede ser un problema para el tratamiento posterior de los datos. Podrías solucionar esto creando una nueva tabla utilizando sentencias case when que sustituyan espacios en blanco en las variables numéricas correspondientes a las últimas 4 variables de nuestra tabla (podemos hacerlo con alguna sentencia de HiveQL) y luego aplicaremos las sentencias case when para sustituir por 0 los espacios en blanco. (Pista: es útil darse cuenta de que un espacio vacío es un campo con longitud 0). Haz esto solo para la tabla padron_txt.**

Para esto lo que vamos a hacer es un sencillo intercambio de nombres.
```
hive> CREATE TABLE padron_txt_3 AS SELECT cod_distrito, desc_distrito, cod_dist_barrio, desc_barrio, cod_barrio, cod_dist_seccion, cod_seccion, cod_edad_int,
    > CASE WHEN length(espanoleshombres) == 0 then 0 else espanoleshombres end espanoleshombres,
    > CASE WHEN length(espanolesmujeres) == 0 then 0 else espanolesmujeres end espanolesmujeres,
    > CASE WHEN length(extranjeroshombres) == 0 then 0 else extranjeroshombres end extranjeroshombres,
    > CASE WHEN length(extranjerosmujeres) == 0 then 0 else extranjerosmujeres end extranjerosmujeres
    > FROM padron_txt;

hive> ALTER TABLE padron_txt RENAME TO padron_txt_borrar;
hive> ALTER TABLE padron_txt_3 RENAME TO padron_txt; -- Es posible que si da error haya que entrar físicamente al HDFS y borrar el directorio que nos indica manualmente.
hive> DROP TABLE padron_txt_borrar;
```
Y esto nos dejaría con la tabla padron_txt con los datos que hemos solicitado.

6. **Una manera tremendamente potente de solucionar todos los problemas previos (tanto las comillas como los campos vacíos que no son catalogados como null y los espacios innecesarios) es utilizar expresiones regulares (regex) que nos proporciona OpenCSV. Para ello utilizamos `ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe' WITH SERDEPROPERTIES ('input.regex'='XXXXXXX')` donde XXXXXX representa una expresion regular que debes completar y que identifique el formato exacto con el que debemos interpretar cada una de las filas de nuestro CSV de entrada. Para ello puede ser útil el portal "regex101". Utiliza este método para crear de nuevo la tabla padron_txt_2.**

Ausencia por falta de expresión regular.

**Una vez finalizados todos estos apartados deberíamos tener una tabla padron_txt que conserve los espacios innecesarios, no tenga comillas envolviendo los campos y los campos nulos sean tratados como valor 0 y otra tabla padron_txt_2 sin espacios innecesarios, sin comillas envolviendo los campos y con los campos nulos como valor 0. Idealmente esta tabla ha sido creada con las regex de OpenCSV.**

## 2. Investigamos el formato columnar parquet.

1. **¿Qué es CTAS?**

Pese a lo que los recién graduados en márketing quieran hacernos creer, CTAS no es *Call to Action*s, sino `CREATE TABLE AS SELECT`. Es una manera de crear una tabla y rellenarla a partir de los datos obtenidos de una consulta.

2. **Crear tabla Hive padron_parquet (cuyos datos serán almacenados en el formato columnar parquet) a través de la tabla padron_txt mediante un CTAS.**

```
hive> CREATE TABLE padron_parquet STORED AS PARQUET
    > AS SELECT * FROM padron_txt;
```

3. **Crear tabla Hive padron_parquet_2 a través de la tabla padron_txt_2 mediante un CTAS. En este punto deberíamos tener 4 tablas, 2 en txt (padron_txt y padron_txt_2, la primera con espacios innecesarios y la segunda sin espacios innecesarios) y otras dos tablas en formato parquet (padron_parquet y padron_parquet_2, la primera con espacios y la segunda sin ellos).**

```
hive> CREATE TABLE padron_parquet_2 STORED AS PARQUET
    > AS SELECT * FROM padron_txt_2;
```

4. **Opcionalmente también se pueden crear las tablas directamente desde 0 (en lugar de mediante CTAS) en formato parquet igual que lo hicimos para el formato txt incluyendo la sentencia STORED AS PARQUET. Es importante para comparaciones posteriores que la tabla padron_parquet conserve los espacios innecesarios y la tabla padron_parquet_2 no los tenga. Dejo a tu elección cómo hacerlo.**

Lo único que se tiene que hacer aquí es ejecutar las consultas creadas anteriormente con la salvedad que donde ponemos `STORED AS TEXTFILE` debemos poner `STORED AS PARQUET`. Esa es la única diferencia.

5. **Investigar en qué consiste el formato columnar parquet y las ventajas de trabajar con este tipo de formatos.**

Apache Parquet es un archivo de datos de código abierto diseñado para un almacenamiento y obtención de datos eficiente, con compresión de datos y encoding. Se almacena en formato columnar en lugar de por filas (como CSV). También almacena metadatos.

Algunas de sus ventajas son: soporte multiplataforma, eficiencia en su almacenamiento columnar (la compresión es más sencilla), y al almacenar metadatos (que aumenta la velocidad a la hora de tomar el schema).

Por dar un ejemplo, si tenemos una columna ventas y queremos todos los registros de ventas mayores que 500 para un mes, es mucho más eficiente buscar en una sola columna (algo que Parquet hace sencillo) que buscar registro por registro la columna correspondiente.

6. **Comparar el tamaño de los ficheros de los datos de las tablas padron_txt (txt), padron_txt_2 (txt pero no incluye los espacios innecesarios), padron_parquet y padron_parquet_2 (alojados en hdfs cuya ruta se puede obtener de la propiedad location de cada tabla por ejemplo haciendo "show create table"**

Usando el comando `describe extended <nombre_tabla>` nos salen una serie de campos. Buscando entre dichos campos encontramos uno llamado `rawDataSize` (está dentro de `parameters`), que contiene el tamaño de la tabla.

Los tamaños obtenidos son los siguientes:
* `padron_txt`: `rawDataSize=16732854` unidades.
* `padron_txt_2`: `rawDataSize=12217245`(es menos porque no tiene espacios)
* `padron_parquet`: `rawDataSize=2859648`
* `padron_parquet_2`: `rawDataSize=2859648`

Puede deducirse que Parquet es mucho más eficiente en tamaño; y también que el número de espacios no influye en el tamaño del almacenamiento.

Nota: Igual no es la manera correcta de hacerlo.

## 3. Juguemos con Impala

1. **¿Qué es Impala?**

**Impala** es un motor SQL de gran rendimiento gracias al procesamiento en paralelo de grandes volúmenes de datos. Tiene muy baja latencia porque los servicios que ejecuta se inician con el propio motor, además de por la **caché de metadatos**, que se cachean al inicio de la sesión y solo se cambian cuando hay cambios en el *meatstore*.

2. **¿En qué se diferencia de Hive?**

* **Impala**: Consultas ligeras para buscar velocidad. Si falla, hay que relanzar la query.
* **Hive**: Mejor para trabajos pesados de ETL. Se busca robustez frente a velocidad. Tolerancia a fallos.

3. **Comando INVALIDATE METADATA, ¿en qué consiste?**

El comando `INVALIDATE METADATA` marca los metadatos de una o más tablas como obsoleto, de manera que la próxima vez que Impala ejecute una consulta en esa(s) tabla(s) tenga que recargar los metadatos. Suele preferirse `REFRESH`, que es incremental y más ligera.

Debe ejecutarse cuando cambien los metadatos de la tabla, cuando se añadan tablas nuevas (que vaya a usar Impala), cuando se cambien los privilegios, cuando se bloqueen metadatos, cambien los jars de UDF o ya no se consulten algunas tablas y se quieran eliminar sus metadatos.

4. **Hacer invalidate metadata en Impala de la base de datos datos_padron.**

```
$ impala-shell
> use datos_padron;
> INVALIDATE METADATA; -- por defecto lo hace en todas la bases de datos. No he averiguado cómo hacerlo solo en una.
```
Debe decirse que no se mostraban tablas antes de invalidar los metadatos, y después sí aparecieron las cuatro.

5. **Calcular el total de EspanolesHombres, EspanolesMujeres, ExtranjerosHombres, ExtranjerosMujeres agrupado por DESC_DISTRITO y DESC_BARRIO.**

```
> SELECT desc_distrito, desc_barrio, SUM(CAST(EspanolesHombres AS INT)) AS Sum_EspH, SUM(CAST(EspanolesMujeres AS INT)) AS Sum_EspM, SUM(CAST(ExtranjerosHombres AS INT)) AS Sum_ExtH, SUM(CAST(ExtranjerosMujeres AS INT)) AS Sum_ExtM
> FROM padron_txt
> GROUP BY desc_distrito, desc_barrio
> ORDER BY desc_distrito, desc_barrio;
```
Esto nos devuelve, tras la notificación de Query submitted, una tabla con la suma de las distintas poblaciones, agrupadas por distrito y barrio.

6. **Llevar a cabo consultas en Hive en las tablas padron_txt_2 y padron_parquet_2 (No deberían incluir espacios innecesarios). ¿Alguna conclusión?**

He realizado la consulta anterior y estos han sido los resultados:
* hive-padron_txt_2: 2 trabajos, el primero tardó un total de 2.73 segundos, y el segundo 1.8 segundos. Un total de 4.53 segundos de ejecución. Se imprime mal; tabula cada fila individualmente, por lo que los barrios de nombre más corto tienen los datos más compactos que los barrios de nombre más largo; esto conlleva que algunas columnas no coincidan con las superiores. Ejecución total: 42.871 segundos
* hive-padron_parquet_2: 2 trabajos también; el primero un total de 3.09 segundos y el segundo 1.79 segundos. Un total de 4.880 segundos. Tiene el mismo problema de formateo que el anterior. Ejecución total: 41.724

No se aprecian realmente mayores diferencias en eficiencia.

7. **Llevar a cabo la misma consulta sobre las mismas tablas en Impala. ¿Alguna conclusión?**

Para empezar, quiero dar nota que Impala permite la exploración y edición de consultas multilínea lo cual se agredece mucho.

* impala-padron_txt_2: No se muestran muchos datos, pero la ejecución completa (incluyendo impresión de resultados) han sido 0.66 segundos.
* impala-padron_parquet_2: Lo mismo que lo anterior. Este ha tardado 0.97 segundos.

La conclusión a la que puedo llegar es exactamente la misma que la anterior. No se aprecian especiales diferencias en eficiencia.

8. **¿Se percibe alguna diferencia de rendimiento entre Hive e Impala?**

La ejecución de Impala es aproximadamente 40 veces más rápida que la ejecución en Hive para esta consulta en particular. En general, gracias a las características de rendimiento de Impala, se puede apreciar un gran aumento en el rendimiento (de varias órdenes de magnitud) a la hora de hacer consultas con respecto a Hive.

## 4. Sobre tablas particionadas.

1. **Crear tabla (Hive) padron_particionado particionada por campos DESC_DISTRITO y DESC_BARRIO cuyos dato estén en formato parquet.**

Por una serie de fallos, he tenido que recrear las tablas, y usando `CAST()` he conseguido crearlas exitosamente en cuanto al tipo de los contenidos. (Esto implicaría que en el punto anterior no sería necesario hacer cast en los ejercicios correspondientes).
```
hive> CREATE TABLE padron_particionado (cod_distrito INT, desc_distrito STRING, cod_dist_barrio INT, desc_barrio STRING, cod_barrio INT, cod_dist_seccion INT, cod_seccion INT, cod_edad_int INT, espanoleshombres INT, espanolesmujeres INT, extranjeroshombres INT, extranjerosmujeres INT) PARTITIONED BY (desc_distrito STRING, desc_barrio STRING) STORED AS PARQUET;
```

Antes de continuar, hay que mencionar que la máquina virtual en la que estamos trabajando no está correctamente configurada. Para conseguir que los datos del apartado siguiente se inserten correctamente, debe realizarse la siguiente configuración:

```
hive> set hive.exec.dynamic.partition = true;
hive> set hive.exec.dynamic.partition.mode = non-strict;
hive> set hive.exec.max.dynamic.partitions = 10000;
hive> set hive.exec.max.dynamic.partitions.pernode = 1000;
hive> set mapreduce.map.memory.mb = 2048;
hive> set mapreduce.reduce.memory.mb = 2048;
hive> set mapreduce.map.java.opts = -Xmx1800m;
```

2. **Insertar datos (en cada partición) dinámicamente (con Hive) en la tabla recién creada a partir de un select de la tabla padron_parquet_2.**

```
hive> INSERT INTO padron_particionado PARTITION (desc_distrito, desc_barrio) SELECT cod_distrito, desc_distrito, cod_dist_barrio, desc_barrio, cod_barrio, cod_dist_seccion, cod_seccion, cod_edad_int, espanoleshombres, espanolesmujeres, extranjeroshombres, extranjerosmujeres, desc_distrito, desc_barrio (los campos) FROM padron_parquet_2;
```


3. **Hacer invalidate metadata en Impala de la base de datos patron_particionado.**

Hecho.

4. **Calcular el total de EspanolesHombres, EspanolesMujeres, ExtranjerosHombres y ExtranjerosMujeres agrupado por DESC_DISTRITO y DESC_BARRIO para los distritos CENTRO, LATINA, CHAMARTIN, TETUAN, VICALVARO y BARAJAS.**

```
> SELECT desc_distrito, desc_barrio, sum(espanoleshombres), sum(espanolesmujeres), sum(extranjeroshombres), sum(extranjerosmujeres) from padron_particionado where desc_distrito IN ('CENTRO', 'LATINA', 'CHAMARTIN', 'TETUAN', 'VICALVARO', 'BARAJAS') GROUP BY desc_distrito, desc_barrio ORDER BY desc_distrito, desc_barrio;
3.75s
```

5. **Llevar a cabo la consulta en Hive en las tablas padron_parquet y padron_particionado. ¿Alguna conclusión?**

* Hive-Padron_Parquet: Este es el que tiene los espacios en blanco, lo cual complica las cosas (%CENTRO%, %LATINA% etc). 2 jobs, 3.18 y 1.46 respectivamente, 4.64 en total. Ejecución total: 42.68 segundos.
* Hive-Padron_Particionado: 2 jobs, de 2.06 y 1.44 segundos respectivamente y un total de 3.5 segundos; ejecución completa 39.679 segundos.

La conclusión es que quizá el particionado es algo más rápido, pero no se corresponde con lo que uno piensa que sería en términos de eficiencia.

6. **Llevar a cabo la consulta en Impala en las tablas padron_parquet y padron_particionado. ¿Alguna conclusión?**

En ninguno de los dos sale ninguna row. Eso sí, lo hace increíblemente rápido, pero es preocupante.

7. **Hacer consultas de agregación (Max, Min, Avg, Count) tal cual el ejemplo anterior con las 3 tablas (padron_txt_2, padron_parquet_2 y padron_particionado) y comparar rendimientos tanto en Hive como en Impala y sacar conclusiones.**

Por lo que veo la eficiencia es varias magnitudes mayor en Impala que en Hive. Extrañamente, debido a la manera que tiene impala de trabajar, no se genera ninguna fila frente a Hive que... tampoco. No sé, algo pasa con el comando IN de la consulta que lo trastoca.

## 5. Trabajando con tablas en HDFS.
A continuación vamos a hacer una inspección de las tablas, tanto externas (no gestionadas) como internas (gestionadas). Este apartado se hará si se tiene acceso y conocimiento previo sobre cómo insertar datos en HDFS.

1. **Crear un documento de texto en el almacenamiento local que contenga una secuencia de números distribuidos en filas y separados por columnas, llámalo datos1 y que sea por ejemplo: [1,2,3][4,5,6][7,8,9]**

En realidad, por corregir, sería algo como:
```
$ cat datos1.txt
1,2,3
4,5,6
7,8,9
```

2. **Crear un segundo documento (datos2) con otros números pero la misma estructura.**
Hecho.

3. **Crear un directorio en HDFS con un nombre a placer, por ejemplo, /test. Si estás en una máquina Cloudera tienes que asegurarte que el servicio HDFS está activo ya que puede no iniciarse al encender la máquina (puedes hacerlo desde el Cloudera Manager). A su vez, enlas máquinas Cloudera es posible (dependiendo de si usamos Hive desde consola o desde Hue) que no tengamos permisos para crear directorios en HDFS salvo en el directorio /user/cloudera.**

```
$ hadoop fs -mkdir /test
$ hadoop fs -ls /
...
drwxr-xr-x    - cloudera supergroup       0 2022-05-09 18:17 /test
```

4. **Mueve tu fichero datos1 al directorio que has creado en HDFS con un comando desde consola.**

```
$ hadoop fs -put datos1.txt /test
```

5. **Desde Hive, crea una nueva database por ejemplo con el nombre numeros. Crea una tabla que no sea externa y sin argumento location con tres columnas numéricas, campos separados por coma y delimitada por filas. La llamaremos por ejemplo numeros_tbl.**

```
hive> CREATE DATABASE numeros;
hive> CREATE TABLE numeros_tbl (n1 INT, n2 INT, n3 INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
```

6. **Carga los datos de nuestro fichero de texto datos1 almacenado en HDFS en la tabla de Hive. Consulta la localización donde estaban anteriormente los datos almacenados. ¿Siguen estando ahí? ¿Dónde están? Borra la tabla, ¿qué ocurre con los datos almacenados en HDFS?**

```
hive> LOAD DATA INPATH '/test/datos1.txt' INTO TABLE numeros_tbl;
```
Datos cargados con éxito.

```
$ hadoop fs -ls /test
```
No está.

Si se borra la tabla (con `DROP TABLE numeros_tbl`), los datos siguen estando borrados.

7. **Vuelve a mover el fichero de texto datos1 desde el almacenamiento local al directorio anterior en HDFS.**

```
$ hadoop fs -put datos1.txt /test
```
Hecho.

8. **Desde Hive, crea una tabla externa sin el argumento location. Y carga datos1 (desde HDFS) en ella. ¿A dónde han ido los datos en HDFS? Borra la tabla ¿Qué ocurre con los datos en hdfs?**

```
hive> CREATE EXTERNAL TABLE numeros_tbl (n1 INT, n2 INT, n3 INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
hive> LOAD DATA INPATH '/test/datos1.txt' INTO TABLE numeros_tbl;
```

En este caso ocurre lo mismo; se borran los datos y al borrar la tabla estos permanecen borrados.

9.  **Borra el fichero datos1 del directorio en el que estén. Vuelve a insertarlos en el directorio que creamos inicialmente (/test). Vuelve a crear la tabla numeros desde hive pero ahora de manera externa y con un argumento location que haga referencia al directorio donde los hayas situado en HDFS (/test). No cargues los datos de ninguna manera explícita. Haz una consulta sobre la tabla que acabamos de crear que muestre todos los registros. ¿Tiene algún contenido?**

Como no estaban en ningún directorio, no hace falta borrarlos. Pasamos a la creación de la tabla.
```
hive> CREATE EXTERNAL TABLE numeros_tbl (n1 INT, n2 INT, n3 INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LOCATION '/test';
```

Ahora, sin cargar los datos de manera explícita, podemos ver que los datos que hemos incluido en la carpeta test aparecen al realizar por ejemplo un `SELECT`, con el contenido que aparecería si cargásemos los datos de forma explícita.

10. **Inserta el fichero de datos creado al principio, "datos2" en el mismo directorio de HDFS que "datos1". Vuelve a hacer la consulta anterior sobre la misma tabla. ¿Qué salida muestra?**

```
$ hadoop fs -put datos2.txt /test
```

Ahora al realizar la misma consulta que antes (un select all) la salida que se muestra se corresponde con una concatenación, por así decirlo.
```
> select * from numeros_tbl;
OK
1   2   3
4   5   6
7   8   9 -- Hasta aquí sería el contenido de datos1.txt
9   7   1 -- Inmediatamente se junta el contenido de datos2.txt
8   8   4
6   2   3
Time taken: 0.496 seconds, Fetched: 6 row(s)
```

11. **Extrae conclusiones de todos los apartados anteriores.**
    
Se llega a la conclusión que la diferencia entre `CREATE TABLE` y `CREATE EXTERNAL TABLE` consiste en la localización de los datos; siempre y cuando se indique la localización de los mismos.

Es decir, `CREATE TABLE` almacena los datos en la propia tabla (y si no se indica `LOCATION`, `CREATE EXTERNAL TABLE` también) y por lo tanto desaparecen de donde estaban. De igual manera, al borrar la tabla se borran los datos también.

Por otro lado, `CREATE EXTERNAL TABLE` si se indica `LOCATION` almacena los datos en una localización externa a la tabla, en la proporcionada por la propiedad `LOCATION`. De esta manera, estos datos no se borran.

## 6. Un poquito de Spark
La siguiente sección de la práctica se abordará si ya se tienen suficientes conocimientos de Spark, en concreto en el manejo de DataFrames, y el manejo de tablas de Hive a través de Spark.sql.

*Esta sección de la cosa esta se va a realizar en local

1. **Comenzamos realizando la misma práctica que hicimos en Hive en Spark, importando el csv. Sería recomendable intentarlo con opciones que quiten las "" de los campos, que ignoren los espacios innecesarios en los campos, que sustituyan los valores vacíos por 0 y que infiera el esquema.**

```
scala> val padron = spark.read.format("csv").option("quote","\"").option("sep",";").option("header","true").option("inferSchema","true").option("ignoreTrailingWhiteSpace","true").load("Rango_Edades_Seccion_202204.csv").na.fill(0).withColumn("DESC_DISTRITO", trim($"DESC_DISTRITO")).withColumn("DESC_BARRIO", trim($"DESC_BARRIO"))
```

*He hecho un poco de trampa, porque he tomado el DataFrame resultante y le he aplicado una operación para que los números vacíos se muestren como 0 a posteriori, además de la modificación de espacios en blanco.*.

2. **De manera alternativa también se puede importar el csv con menos tratamiento en la importación y hacer todas las modificaciones para alcanzar el mismo estado de limpieza de los datos con funciones de Spark.**

Sí, de manera alternativa podría hacerse, pero sería más complicado. Además, ya lo he hecho a medias.

3. **Enumera todos los barrios diferentes.**

Con scala tenemos una operación, `dropDuplicates()`, que nos genera un DF a partir de otro.
```
scala> val distintos = padron.select($"DESC_DISTRITO", $"DESC_BARRIO").dropDuplicates("DESC_BARRIO").sort($"DESC_BARRIO").sort($"DESC_DISTRITO")
```

4. **Crea una vista temporal de nombre "padron" y a través de ella cuenta el número de barrios diferentes que hay.**

```
scala> padron.createOrReplaceTempView("padron")
scala> spark.sql("SELECT COUNT(DISTINCT DESC_BARRIO) FROM padron")
``` 

5. **Crea una nueva columna que muestre la longitud de los campos de la columna DESC_DISTRITO y que se llame "longitud".**

```
scala> val longitudes = padron.withColumn("longitud", length($"DESC_DISTRITO"))
```

6. **Crea una nueva columna que muestre el valor 5 para cada uno de los registros de la tabla.**

```
scala> val cinco = longitudes.withColumn("cinco", expr(5))
```

7. **Borra esta columna.**

```
scala> val sincinco = cinco.drop($"cinco")
```

8. **Particiona el DataFrame por las variables DESC_DISTRITO y DESC_BARRIO.**

```
scala> val partido = sincinco.repartitionByRange($"DESC_DISTRITO", $"DESC_BARRIO")
```

9.  **Almacénalo en caché. Consulta en el puerto 4040 (UI de Spark) de tu usuario local el estado de los rdds almacenados.**

Para que se muestre en el almacenamiento, primero hay que realizar una acción.
```
scala> partido.cache()
scala> partido.count()
```
Una vez ahí, nos podemos dirigir a `localhost:4040/storage` en un navegador y ahí veremos la siguiente tabla:

| ID  | RDD Name | Storage Level | Cached Partitions | Fraction Cached | Size in Memory | Size on Disk |
| --- | -------- | ------------- | ----------------- | --------------- | -------------- | ------------ |
| 302 | Exchange rangepartitioning(DESC_DI[... this is way too long ...] #128 8, COD_SE...) | Disk Memory Deserialized 1x Replicated | 132 | 100% | 1436.1 KiB | 0.0 B |

Y si entramos en el RDD se nos muestra el nombre muy largo y las particiones cacheadas (132 en total, correspondiente con el nº de barrios distintos), además de otros datos de interés como la distribución de los datos y las distintas distribuciones, con nombres tan intuitivos como rdd_302_81; y su tamaño en memoria.

*Solo por curiosidad, el nombre completo del RDD es `RDD Storage Info for Exchange rangepartitioning(DESC_DISTRITO#1341 ASC NULLS FIRST, DESC_BARRIO#1354 ASC NULLS FIRST, 200), REPARTITION, [id=#827] +- ^(1) Project [coalesce(COD_DISTRITO#1283, 0) AS COD_DISTRITO#1319, trim(DESC_DISTRITO#1284, None) AS DESC_DISTRITO#1341, coalesce(COD_DIST_BARRIO#1285, 0) AS COD_DIST_BARRIO#1320, trim(DESC_BARRIO#1286, None) AS DESC_BARRIO#1354, coalesce(COD_BARRIO#1287, 0) AS COD_BARRIO#1321, coalesce(COD_DIST_SECCION#1288, 0) AS COD_DIST_SECCION#1322, coalesce(COD_SECCION#1289, 0) AS COD_SECCION#1323, coalesce(COD_EDAD_INT#1290, 0) AS COD_EDAD_INT#1324, coalesce(EspanolesHombres#1291, 0) AS EspanolesHombres#1325, coalesce(EspanolesMujeres#1292, 0) AS EspanolesMujeres#1326, coalesce(ExtranjerosHombres#1293, 0) AS ExtranjerosHombres#1327, coalesce(ExtranjerosMujeres#1294, 0) AS ExtranjerosMujeres#1328, length(trim(DESC_DISTRITO#1284, None)) AS longitud#1674] +- FileScan csv [COD_DISTRITO#1283,DESC_DISTRITO#1284,COD_DIST_BARRIO#1285,DESC_BARRIO#1286,COD_BARRIO#1287,COD_DIST_SECCION#1288,COD_SE...`*

*Es demasiado largo hasta para mostrarlo tal cual.*

10.  **Lanza una consulta contra el DF resultante en la que muestre el número total de "espanoleshombres", "espanolesmujeres", "extranjeroshombres" y "extranjerosmujeres" para cada barrio de cada distrito. Las columnas distrito y barrio deben ser las primeras en aparecer en el show. Los resultados deben estar ordenados en orden de más a menos según la columna "extranjerosmujeres" y desempatarán por la columna "extranjeroshombres".**

```
scala> val partidoOrdenado = partido.withColumn("distrito", col("DESC_DISTRITO")).withColumn("barrio", col("DESC_BARRIO")).select("distrito", "barrio", "espanoleshombres", "espanolesmujeres", "extranjeroshombres", "extranjerosmujeres").groupBy("distrito", "barrio").agg(sum("espanoleshombres") as "SumEspHom", sum("espanolesmujeres") as "SumEspMuj", sum("extranjeroshombres") as "SumExtHom", sum("extranjerosmujeres") as "SumExtMuj").sort(desc("SumExtHom")).sort(desc("SumExtMuj"))
```
Es importante destacar que el ordenado por varias columnas se realiza en orden inverso a SQL. El desempate se indica el primero. A no ser que se haga en el mismo comando, que sería en orden normal: `sort(desc("SumExtMuj"), desc("SumExtHom"))`.

11.  **Elimina el registro en caché.**

```
scala> partido.unpersist()
```
, por lo visto. Alternativamente se puede realizar un limpiado completo con `sqlContext.clearCache()` o algo similar.

12.  **Crea un nuevo DataFrame a partir del original que muestre únicamente una columna con DESC_BARRIO, otra con DESC_DISTRITO y otra con el número total de "espanoleshombres" residentes en cada distrito de cada barrio. Únelo (con un join) con el DataFrame original a través de las columnas en común.**

```
scala> val nuevo = padron.select("DESC_BARRIO", "DESC_DISTRITO", "espanoleshombres").groupBy("DESC_BARRIO", "DESC_DISTRITO").agg(sum("espanoleshombres"))
scala> val juntado = nuevo.join(padron, nuevo("DESC_BARRIO") === padron("DESC_BARRIO") && nuevo("DESC_DISTRITO") === padron("DESC_DISTRITO"), "inner")
```

13.  **Repite la función anterior utilizando funciones de ventana. (over(Window.partitionBy.....)).**

```
scala> val resul = padron.withColumn("TotalEspHom", sum(col("espanoleshombres")).over(Window.partitionBy("DESC_DISTRITO, "DESC_BARRIO")))
```
Añadiendo un filtro puede verse que la columna que estamos añadiendo se corresponde con la suma total de Españoles Hombres para ese barrio.

14.  **Mediante una función Pivot muestra una tabla (que va a ser una tabla de contingencia) que contenga los valores totales (la suma de valores) de espanolesmujeres para cada distrito y en cada rango de edad (COD_EDAD_INT). Los distritos incluidos deben ser únicamente CENTRO, BARAJAS y RETIRO y deben figurar como columnas. El aspecto debe ser similar a este:** COD_EDAD_INT, BARAJAS, CENTRO, RETIRO - y datos.

```
scala> val pivotDF = padron.filter($"DESC_DISTRITO".isin("CENTRO","BARAJAS","RETIRO")).groupBy("COD_EDAD_INT").pivot($"DESC_DISTRITO").agg(sum("EspanolesMujeres")).sort("COD_EDAD_INT")
```

15.  **Utilizando este nuevo DF, crea 3 columnas nuevas que hagan referencia a qué porcentaje de la suma de "espanolesmujeres" en los tres distritos para cada rango de edad representa cada uno de los tres distritos. Debe estar redondeada a 2 decimales. Puedes imponerte la condición extra de no apoyarte en ninguna columna auxiliar creada para el caso.**

```
scala> pivotDF.withColumn("PorcentajeBarajas",(round(((col("BARAJAS"))/(col("BARAJAS")+col("CENTRO")+col("RETIRO")))*100, 2)))
.withColumn("PorcentajeCentro",(round(((col("CENTRO"))/(col("BARAJAS")+col("CENTRO")+col("RETIRO")))*100, 2)))
.withColumn("PorcentajeRetiro",(round(((col("RETIRO"))/(col("BARAJAS")+col("CENTRO")+col("RETIRO")))*100, 2))).show()
```

16.  **Guarda el archivo csv original particionado por distrito y por barrio (en ese orden) en un directorio local. Consulta el directorio para ver la estructura de los ficheros y comprueba que es la esperada.**

```
scala> padron.write.format("csv").partitionBy("DESC_DISTRITO","DESC_BARRIO").save("csv_thing/arquivo.csv")
```
De esta manera se guarda en una carpeta llamada `csv_thing` dentro de la carpeta donde estemos. El árbol resultante es el siguiente (`(d)` determina que es un directorio):
```
csv_thing
|arquivo.csv (d)
 |DESC_DISTRITO=ARGANZUELA (d)
 ||DESC_BARRIO=ACACIAS (d)
 |||_commited_1517545717796882501...
 |||_started_151754717796882501...
 |||_SUCCESS
 |||part-00000-tid-151754571779688...
 ||DESC_BARRIO=ATOCHA (d)
 || [etcétera]
 ||DESC_BARRIO=PALOS DE MOGUER (d)
 |DESC_DISTRITO=BARAJAS (d)
 |DESC_DISTRITO=CARABANCHEL (d)
 |[etcétera]
 |DESC_DISTRITO=VILLAVERDE (d)
 |_SUCCESS
```
Más o menos así, por dotar de un resumen. El archivo `part-00000-etcétera` es el que contiene el csv correspondiente a ese barrio. Importante destacar que no aparecen las columnas DESC_DISTRITO ni DESC_BARRIO; ni por supuesto la cabecera.

17.  **Haz el mismo guardado pero en formato parquet. Compara el peso del archivo con el resultado anterior.**

```
scala> padron.write.format("parquet").partitionBy("DESC_DISTRITO","DESC_BARRIO").save("parquet_thing/arquivo.parquet")
```
El árbol de ficheros y directorios generado es exactamente igual, excepto que se generan archivos parquet en lugar de archivos csv.

## 7. ¿Y si juntamos Spark y Hive?

1. **Por último, prueba a hacer los ejercicios sugeridos en la parte de Hive con el csv "Datos Padrón" (incluyendo la importación con Regex) utilizando desde Spark EXCLUSIVAMENTE sentencias spark.sql, es decir, importar los archivos desde local directamente como tablas de Hive y haciendo todas las consultas sobre estas tablas sin transformarlas en ningún momento en DataFrames ni DataSets.**

Este ejercicio puede realizarse simplemente añadiendo `spark.sql()` y metiendo cada consulta realizada entre los paréntesis.