# Ejercicios Hive

Por convención, las palabras reservadas se escribirán en mayúsculas. Esto se una convención y no es necesario, pero ayuda a distinguirlo de valores variables.

1. El código para entrar a hive es el siguiente:
    ```
    hive
    ```
    Esto cargará *hive* y abrirá la terminal en la propia consola donde se haya ejecutado.
2. Crear una base de datos llamada "cursohivedb"
   
    Lo primero que debemos hacer es asegurarnos de que no existe ya de por sí.
    ```
    hive> SHOW DATABASES; --ojo que este lenguaje usa ;

    OK -- El comando es correcto.
    cursohivedb -- Como ya hemos hecho el ejercicio, vemos que efectivamente existe.
    default
    prueba_sqoop_hive
    Time taken: 0.477 seconds, Fetched: 3 row(s) -- Se nos muestran algunas estadísticas después de mostrarnos todas las bases de datos.
    ```

    Para continuar con la creación deberemos borrar la base de datos. Esto se hace así:

    ```
    hive> DROP DATABASE cursohivedb;
    FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database cursohivedb is not empty. One or more tables exist.)
    ```

    Evidentemente no nos permite borrar una base de datos que no esté vacía. Como todo el tema del borrado se escapa al ámbito de este ejercicio, vamos a optar por una solución alternativa: otro nombre. Dejo esto al menos como curiosidad.

    El código para crear una base de datos es el siguiente:

    ```
    hive> CREATE DATABASE repe_curso;
    OK
    Time taken: 0.127 seconds
    ```
   
3. Situarnos en la base de datos recién creada para trabajar con ella:
    ```
    hive> USE repe_curso;
    OK
    Time taken: 0.021 seconds
    ```

4. Comprobar que la base de datos está vacía
    
    La mejor manera de realizar eso es comprobar que no tiene ninguna tabla asociada.
    ```
    hive > SHOW TABLES;
    OK
    --Aquí irían las tablas, pero al no tener ninguna, pues no hay.
    Time taken: 0.035 seconds

    --Veamos un ejemplo donde sí hay tablas.
    hive> USE default;
    OK
    Time taken: 0.012 seconds
    hive> show tables;
    OK
    tabla_prueba
    tabla_prueba_hive
    Time taken: 0.014 seconds, Fetched: 2 row(s)
    ```
    Pues eso.

5. Crear una tabla llamada "iris" en nuestra base de datos que contenga 5 columnas (s_length float, s_width float, p_length float, p_width float, clase string) cuyos campos estén separados por comas (ROW FORMAT DELIMITED FIELDS TERMINATED BY ',')
    ```
    hive> CREATE TABLE iris( -- Hive permite realizar operaciones multilínea.
        > s_length FLOAT, -- convenientemente se nos han
        > s_width FLOAT, -- dado los tipos de los campos.
        > p_length FLOAT,
        > p_width FLOAT,
        > clase STRING
        > ) -- Aquí termina la tabla.
        > ROW FORMAT DELIMITED -- El formato de las filas es campos delimitados por un separador.
        > FIELDS TERMINATED BY ','; -- En este caso el separador es la coma.
    OK
    Time taken: 0.168 seconds
    ```

6. Comprobar que la tabla se ha creado y el tipado de sus columnas.

    Nuevamente lo mejor que podemos hacer es mostrar las tablas.
    ```
    hive> SHOW TABLES;
    OK -- El OK, como se va comprobando, es la muestra de que el comando se ejecuta sin ningún error de sintaxis.
    iris
    Time taken: 0.011 seconds, Fetched: 1 row(s)
    ```
    Otra cosa que podemos hacer para mostrar el tipado de las columnas es usar el comando DESCRIBE y la tabla en cuestión. Esto nos muestra los campos con sus tipos.
    ```
    hive> DESCRIBE iris;
    OK
    s_length        float
    s_width         float
    p_length        float
    p_width         float
    clase           string
    Time taken: 0.121 seconds, Fetched: 5 row(s)
    ```

7. Importar el fichero "iris_completo.txt" al sistema hdfs. La importación de ficheros ya la hemos visto. Comprueba que el fichero está donde tiene que estar.
    ```
    $hadoop fs -mkdir /user/cloudera/hive -- Creamos una carpeta para el archivo.
    $hadoop fs -put /home/cloudera/ejercicios/ejercicios_HIVE/iris_completo.txt /user/cloudera/hive -- put, recordamos, va de local a hdfs.

    --Para comprobarlo, hacemos un ls.
    $hadoop fs -ls hive
    -rw-r--r--   1 cloudera cloudera       4551 2022-04-05 10:10 hive
    --Estos son los archivos que están dentro de la carpeta hive en el sistema de archivos hdfs.
    ```

8. Importar el fichero que hemos importado a la tabla. Comprobar que la inserción se ha realizado.

    Esto se hace mediante un comando específico.

    ```
    hive> LOAD DATA INPATH '/user/cloudera/hive/iris_completo.txt' INTO TABLE iris; -- Este comando inserta los datos del archivo en una tabla.
    Loading data to table repe_curso.iris
    Table repe_curso.iris stats: [numFiles=1, totalSize=4551]
    OK
    Time taken: 0.374 seconds

    --Es importante destacar que usar LOAD DATA borra el archivo de HDFS.
    --Vamos a comprobar que haya datos.
    hive> SELECT * FROM iris;
    OK
    --Y efectivamente, se nos devuelven datos.
    5.1     3.5     1.4     0.2     Iris-setosa
    4.9     3.0     1.4     0.2     Iris-setosa
    [...]
    6.2     3.4     5.4     2.3     Iris-virginica
    5.9     3.0     5.1     1.8     Iris-virginica
    NULL    NULL    NULL    NULL    NULL
    Time taken: 0.054 seconds, Fetched: 151 row(s)
    ```
9. Mostrar las 5 primeras filas de la tabla iris
    ```
    hive> SELECT * FROM iris LIMIT 5; --LIMIT funciona exactamente igual que en SQL, limitando el nº de filas devueltas a las primeras 5.
    OK
    --Y nos devuelve las cinco primeras filas, que no voy a copiar.
    Time taken: 0.043 seconds, Fetched: 5 row(s)
    ```

10. Una serie de ejercicios de consultas y operaciones sencillas, las cuales funcionan todas exactamente igual que usando SQL.
    ```
    --Filas cuyo s_length sea mayor que 5. Observad que se ejecuta un MapReduce y que el tiempo de ejecución es mayor
    hive> SELECT * FROM iris WHERE s_length > 5;
    OK
    Time taken: 0.156 seconds, Fetched: 118 row(s) --Es bastante mayor.

    --Media de s_width agrupados por clase. El tiempo de ejecución es mucho mayor
    hive> SELECT * AVG(s_width) FROM iris GROUP BY clase;
    Launching Job 1 out of 1
    Number of reduce tasks not specified. Estimated from input data size: 1
    In order to change the average load for a reducer (in bytes):
      set hive.exec.reducers.bytes.per.reducer=<number>
    In order to limit the maximum number of reducers:
      set hive.exec.reducers.max=<number>
    In order to set a constant number of reducers:
      set mapreduce.job.reducers=<number>
    Starting Job = job_1649163935098_0002, Tracking URL = http://quickstart.cloudera:8088/proxy/application_1649163935098_0002/
    Hadoop job information for Sage-1: number of mappers: 1; number of reducers: 1
    2022-04-07 18:17:04,821 Stage-1 map = 0%,  reduce = 0%
    2022-04-07 18:17:12,429 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.87 sec
    2022-04-07 18:17:19,902 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 1.97 sec
    MapReduce Total cumulative CPU time: 1 seconds 970 msec
    Ended Job = job_1649163935098_0002
    MapReduce Jobs Launched:
    Stage-Stage-1: Map: 1  Reduce: 1  Cumulative CPU: 1.97 sec  HDFS Read: 13246 HDFS Write: 57 SUCCESS
    Total MapReduce CPU Time Spent: 1 seconds 970 msec
    OK
    NULL
    3.41800000667572
    2.770000009536743
    2.9739999914169313
    Time taken: 25.172 seconds, Fetched: 4 row(s)
    ```
Hay mucho que desengranar aquí. Vamos paso a paso.

Lo primero es que se ejecut la salida estándar de un Job de Hadoop, algo que no ocurre en los casos anteriores.

Después se nos indica una URL. Como ya ha terminado el trabajo, no podemos acceder a la misma, pero mediante la página principal de nuestras aplicaciones hadoop ((quickstart.cloudera:8088, que nos redirige a /cluster) ahí aparece el trabajo que acabamos de terminar en el historial. Entrando en él, nos aparece su nombre (la consulta que hemos realizado), el usuario que lo ha ejecutado, la fecha y hora de ejecución, comienzo, finalización, el tiempo que ha tardado en total y en sus partes, así como otras cuestiones.