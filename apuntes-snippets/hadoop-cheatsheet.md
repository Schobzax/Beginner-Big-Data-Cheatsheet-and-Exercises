# Hadoop Cheatsheet

Es importante destacar que HDFS no es el sistema local. Son dos sistemas separados. HDFS puede acceder al sistema local, pero no al revés.

Es interesante también destacar que la carpeta desde la que se empieza tiene como prefijo /user/cloudera, en el caso de la máquina virtual. Esto se puede comprobar con el hecho de que al hacer `hadoop fs -ls` y `hadoop fs -ls /user/cloudera` salen las mismas carpetas, pero con `/user/cloudera` por delante.

## HDFS

* `hadoop fs` - Listado de comandos disponibles para HDFS
  
  En su mayoría, casi todos los comandos son equivalentes a comandos linux: `ls`, `rm`, `cat`, entre otros funcionan de manera similar. Algunos ejemplos:
  * `hadoop fs -ls <directorio linux>` - Lista los archivos en ese directorio *de HDFS*.
  * `hadoop fs -rm <archivo>` - Borra un archivo en HDFS.
  * `hadoop fs -cat <archivo>` - Mira los contenidos de un archivo por la salida estándar (la consola).
  * `hadoop fs -mkdir <ruta>` - Crea un directorio vacío con la ruta seleccionada (si solo se pone el nombre, se asume directorio actual).
  
  Otros comandos:
  * `hadoop fs -put <orig> <dest>` - Coloca un archivo "orig" en el sistema local en una localización "dest" en el hdfs
  * `hadoop fs -get <orig> <dest>` - Coloca un archivo "orig" de HDFS en la localización "dest" del sistema local.

* `hadoop jar <jar-file> <solution-class> <orig> <dest>` - Ejecuta un archivo jar determinado en jar-file y usando la solution-class ejecuta un *MapReduce* en el archivo "orig" y lo escribe en "dest".

## MapReduce

* `mapred job -list` - Muestra la lista de Jobs en ejecución.
* `mapred job -kill <JobID>` - Detiene un Job en ejecución a partir de su JobID.

## Hive

* Entrada: `hive`
* Dentro: Los comandos son como usar SQL desde consola.
* Salida: `hive> exit;`

* Importación de datos a tabla: `LOAD DATA [LOCAL] INPATH '<ruta_archivo_externo>' [OVERWRITE] INTO TABLE <tabla> PARTITION(<campo>='<valor>'[,<otro_campo>=<otro_valor>...])`
  * `LOCAL` indica que es un fichero local. Si no, es un fichero en HDFS.
  * `OVERWRITE` indica sobrescribir la tabla en destino.

***IMPORTANTE***: Usar `LOAD DATA` borra el archivo de HDFS.
* Exportación de datos de tabla: `INSERT OVERWRITE [LOCAL] DIRECTORY '<ruta/archivo>' SELECT campos, varios FROM tabla [WHERE condicion]` Para guardar el resultado de una consulta en un archivo local. Para condiciones, ver importación de datos a tabla.

## Impala

* Entrada: `impala-shell` para entrar a la consola de Impala. El resto es similar a Hive.
* Cabe destacar que también se accede a las bases de datos disponibles en HDFS (me aparece cursohivedb, por ejemplo)
* Definitivamente es más rápido.

## Pig y Pig Latin

Como si de un lenguaje SQL se tratara, Pig también tiene palabras reservadas.

### Comentarios

* Comentario de línea: `--Esto es un comentario`
* Comentario de bloque: `/* Esto es un \n comentario */`

### Operaciones comunes

Similares a SQL.

* Comparaciones: `==` y `!=`

### Carga de datos

La función se llama PigStorage, implícito en la instrucción LOAD.
* Carga de datos: `variable = LOAD 'tabla' [AS (columna, otracolumna)]` - Carga la tabla `tabla` en la variable `variable` con sus atributos llamándose `columna` y `otracolumna`.
  * Nombrar los atributos es opcional y pueden referirse por número: `$0`, `$1`, `$2`...
  * `tabla` es una ruta al directorio de la tabla.
  
* Carga de datos por fichero: `variable = LOAD 'tabla.(csv|txt|otro) USING PigStorage(',') [AS (columna, otracolumna)]` - `USING` permite determinar el delimitador de campo que se va a usar.

### Muestra/Almacenamiento de datos

* `DUMP variable` - Muestra el contenido de la variable en formato de paréntesis: `(Dato,123,xx)`
* `STORE variable INTO 'ruta_guardado'` - Guarda el contenido de la variable en el disco, en HDFS, en la ruta `ruta_guardado` que se ha determinado (nuevamente, puede ser absoluta o relativa).
  * Delimitador por defecto es tab. Puede modificarse usando `USING PigStorage('x')`, siendo `x` el comando delimitador.

**IMPORTANTE:** Al realizar un `STORE`, la ruta de salida (`ruta_guardado`) no debe existir. Será un directorio.

### Tipos de datos

Pig soporta todos los tipos de datos básicos. Los *fields* no identificados se portan como arrays de bytes, llamados `bytearray`.

* Tipos soportados: `int`, `long`, `float`, `double`, `boolean`, `datetime`, `chararray`, `bytearray`.

* Especificación: Es recomendable. `variable = LOAD 'tabla' AS (atributo:int, otroatributo:chararray)`

* Datos inválidos: Pig los sustituye por `NULL`.
* Filtrar datos erróneos: `IS NULL`, `IS NOT NULL`.
```
hasprices = FILTER records BY price IS NOT NULL; --Solo mostrará los récords cuyo precio es válido.
```

Y con esto pasamos al siguiente apartado.

### Filtrado de datos

* `FILTER` es el comando que se usa para extraer tuplas que cumplan una condición. Es una especie de `WHERE` compacto.
* `variable = FILTER otravariable BY condicion`, donde `condicion` puede ser por ejemplo `atributo > 10` o `atributo == 'texto'`. 
* Expresiones regulares: `variable = FILTER otravariable BY atributo MATCES 'regexp` -- donde `regexp` es una expresión regular de toda la vida.

Ejemplo de expresión regular:
```
spammers = FILTER senders BY email_addr MATCHES '.*@example|.com$';
```

### Selección, extracción, generación, eliminación, ordenación
* `FOREACH` itera por cada tupla.
* `GENERATE` trae campos ya existentes o calculados a la nueva variable.
* `variable = FOREACH otravariable GENERATE campo1, campo1 * 2 AS doble:int;` - Esto genera en la `variable` una tabla que toma el `campo1` de `otravariable` y otra columna que es el doble de dicho valor, dándole el nombre `doble` y el tipo `int`.


* `DISTINCT` selecciona los valores únicos de cada consulta. `únicos = DISTINCT variablegrande;`
* `ORDER ... BY` ordena de forma ascendente por defecto. Hay que añadir `DESC` para que sea descendente. `ordenado = ORDER variable BY campo [DESC];`

### Funciones

* `UPPER(campo)` - Pasa a mayúsculas un campo de texto.
* `TRIM(campo)` - Elimina espacios en blanco al principio y al final de un campo de texto.
* `RANDOM()` - Genera un número aleatorio entre 0 y 1.
* `ROUND(price)` - Redondea un número flotante a entero.
* `SUBSTRING(campo, x, y)` - Coge una subcadena empezando en el carácter nº `x` y con una longitud `y` a partir de la cadena en `campo`.

### Salir

* `quit;` para salir.