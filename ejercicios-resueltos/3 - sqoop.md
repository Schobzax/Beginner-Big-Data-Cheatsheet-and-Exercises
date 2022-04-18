# Ejercicios de Sqoop
Por convención, las palabras reservadas se escribirán en mayúsculas. Esto es una convención y no es necesario, pero ayuda a distinguirlo de valores variables o identificadores.

1. Lo primero que tenemos que hacer es crear una tabla en MySQL. Veremos por fin cuánto se parece a HQL y programas similares.
    ```
    $ mysql -u root -p
    Enter password: cloudera
    --Se conecta al servicio de MySQL.
    --Comprobemos qué tenemos por aquí.
    mysql> SHOW DATABASES;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | cm                 |
    | firehose           |
    | hue                |
    | ..y otras tantas   |
    +--------------------+
    13 rows in set (0.00 sec)

    --Vemos que hay unas cuantas.
    --Vamos a crear la nuestra propia.
    mysql> CREATE DATABASE pruebadb;
    Query OK, 1 row affected (0.00 sec)
    --Ahora nos metemos y creamos una tabla.
    mysql> USE pruebadb;
    Database changed
    mysql> CREATE TABLE tabla_prueba (nombre varchar(3), edad int);
    Query OK, 0 rows affected (0.04 sec)
    --Por último, comprobamos que se ha creado la tabla.
    mysql> SHOW TABLES;
    +--------------------+
    | Tables_in_pruebadb |
    +--------------------+
    | tabla_prueba       |
    +--------------------+
    1 row in set (0.00 sec)
    --Insertamos unas filas y mostramos los datos de la tabla.
    mysql> INSERT INTO tabla_prueba VALUES ('Alberto',22),('Luis',23),('Pablo',24),('Carlos',25);
    Query OK, 4 rows affected (0.01 sec)
    Records: 4  Duplicates: 0  Warnings: 0
    mysql> SELECT * FROM tabla_prueba;
    +---------+------+
    | nombre  | edad |
    +---------+------+
    | Alberto |   22 |
    | Luis    |   23 |
    | Pablo   |   24 |
    | Carlos  |   25 |
    +---------+------+
    4 rows in set (0.00 sec)
    --Podemos ver que los datos se han insertado correctamente.
    mysql> DESCRIBE tabla_prueba;
    +--------+-------------+------+-----+---------+-------+
    | Field  | Type        | Null | Key | Default | Extra |
    +--------+-------------+------+-----+---------+-------+
    | nombre | varchar(30) | YES  |     | NULL    |       |
    | edad   | int(11)     | YES  |     | NULL    |       |
    +--------+-------------+------+-----+---------+-------+
    2 rows in set (0.01 sec)
    --Comprobamos que la tabla se ha creado correctamente y sus características se corresponden.
    ```

2. Ahora vamos a hacer lo mismo pero en Hive: será la tabla donde importaremos los datos que acabamos de crear en MySQL.
   ```
   mysql> exit
   --Lo primero que hacemos es acceder a hive
   $ hive
   --Y crear y acceder a una base de datos.
   hive> CREATE DATABASE prueba_sqoop_hive;
   OK
   Time taken: 0.079 seconds
   hive> USE prueba_sqoop_hive;
   OK
   Time taken: 0.018 seconds
    --Comprobamos que efectivamente existe en el sistema HDFS (debemos hacerlo fuera de hive. Podemos abrir una segunda consola o salir de hive y volver a entrar a continuación)
    $ hadoop fs -ls /user/hive/warehouse
   ```
Esto nos muestra las bases de datos disponibles en el almacén de hive creado para el usuario. Se deberían mostrar las mismas tablas que se muestran al hacer `show databases;` en hive. Y, efectivamente, aparece la base de datos que hemos creado.

   ```
   [...]
   drwxrwxrwx   - cloudera supergroup        0 2022-04-08 14:23 /user/hive/warehouse/prueba_sqoop_hive.db
   [...]
   ```

Después volvemos a hive para crear la tabla en la que vamos a importar los datos.

   ```
   hive> USE prueba_sqoop_hive;
   OK
   Time taken: 0.267 seconds
   hive> DROP TABLE tabla_prueba_hive; --Borramos la tabla por si existiera antes
   OK
   Time taken: 0.095 seconds
   hive> CREATE TABLE tabla_prueba_hive(
       > nombre STRING,
       > edad   INT,
       > )
       > ROW FORMAT DELIMITED
       > STORED AS TEXTFILE; -- Creamos la tabla con los campos necesarios para la importación.
   ```

Tras comprobar con `show tables` que la tabla existe, pasamos al siguiente paso: importar la tabla de una a otra.

3. Primero hay que configurar la base de datos Accumulo.
```
$ sudo mkdir /var/lib/accumulo -- Creamos la carpeta donde se guardará la base de datos.
$ ACCUMULO_HOME='/var/lib/accumulo' -- Importante que no tenga espacios o nos dará error.
$ export ACCUMULO_HOME -- Guardamos la variable.
```

4. Ahora conectamos Sqoop con MySQL y visualizamos las conexiones.
```
$ sqoop list-databases --connect jdbc:mysql://localhost --username root --password cloudera
22/04/18 09:11:27 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6-cdh5.13.0
22/04/18 09:11:27 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure.Consider using -P instead. -- Se nos advierte sobre poner la contraseña en texto plano en la terminal.
22/04/18 09:11:27 INFO manager.MySQLManager: Preparing to use a MySQL streaming subset.
information_schema -- La primera base de datos.
cm
firehose
[...] -- Hay unas cuantas más que omito.
rman
sentry
```
Aquí estamos ejecutando `list-databases` sobre una conexión establecida por una cadena, un nombre de usuario y una contraseña.

5. Por último, importamos la tabla creada en MySQL a la estructura creada en Hive.

Usaremos el comando `import`y usando la cadena de conexión de nuestra base de datos creada (`--connect`), nos conectaremos a la tabla concreta que queremos (`--table`) mediante usuario y contraseña usando un solo mapper (`-m 1`) e importando directamente a hive (`--hive-import`) a una tabla concreta (`--hive-table`)
```
$ sqoop import --connect jdbc:mysql://localhost/pruebadb --table tabla_prueba --username root --password cloudera -m 1 --hive-import --hive-overwrite --hive-table prueba_sqoop_hive.tabla_prueba_hive;
--Después de una serie de cadenas de advertencia e información (WARN e INFO), comienza un trabajo MapReduce.
```

Por último y como paso adicional, con `hive` podemos comprobar si los datos se han importado correctamente, accediendo a la tabla correspondiente en la base de datos correspondiente.