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

## Sqoop

* `sqoop help [<comando>]` - Por defecto mostrará una lista de comandos. Si pones un comando te mostrará la ayuda, opciones y explicación de ese comando.

### Importación
* `sqoop import --username user --password pass --connect jdbc:mysql://database.example.com/personal [--table empleados]/[--where "condicion"]/[--query "SELECT * FROM tabla"] -m x --target-dir XXX`
  * `--username` para poner el usuario de acceso a la base de datos.
  * `--password` para poner la contraseña de acceso a la base de datos en texto plano.
    * Puede hacerse `--username user -p`, que te pedirá la contraseña por consola en lugar de escribirla en texto plano (es más seguro).
    * También puede usarse `--password-alias`, señalando un archivo donde está guardada la contraseña. 
  * `--connect` señala la cadena de conexión, el motor y la base de datos donde conectarse.
  * `--table` señala la tabla de la base de datos conectada de donde sacar los datos a importar.
  * `--where` es una condición SQL a cumplir por los datos que se importarán. Por ejemplo, `where "edad>35"` o cosas así.
  * `--query` permite hacer una consulta SQL más compleja (usando JOIN y cosas así) no limitada por los confines de un WHERE.
  * O se usa `table` y opcionalmente `where` o se usa `query`.
  * `--target-dir` señala el directorio objetivo donde se guardará el archivo `part-*.0*` donde estarán guardados los datos.
  * `-m` configura un número `x` de Mappers para este `import.`
El resto de opciones tratan con temas de fuente, configuración de NULL, particiones, etcétera.

### Exportación
* `sqoop export` - Permite exportar datos de HDFS e insertarlos en una tabla existente de una RDBMS

### Compatibilidad con Hive
Sqoop facilita importar directamente a Hive sin pasar por HDFS.
* `sqoop import <argumentos> --hive-import` importa directamente a hive.
  * Si la tabla ya existe se puede añadir la opción `-hive-overwrite` para sobreescribirla.
  * Sqoop a continuación genera un script HQL para crear la tabla si no exite.
  * Por último se genera uan instrucción de carga para mover los datos al warehouse de hive.

Las opciones son similares y tratan con diversas cuestiones de configuración, particiones, reemplazos de carácteres especiales, etcétera.

## Flume
Sobre todo temas de configuración más usuales.
### Sources por defecto
* Avro: Escucha de un puerto Avro y recibe eventos desde streams de clientes externos Avro.
* Thrift: Igual pero con Thrift, puede autenticarse con Kerberos
* Exec: Ejecuta comandos Unix al inicializar la fuente. Si es comando continuo (cat, tail) se irán recogiendo eventos según un límite (tiempo, nlíneas). Si es concreto (date, ls) solo se recoge un evento.
* JMS: Se leen mensajes de una cola.
* Spooling directory: Se lee desde ficheros movidos a un directorio concreto. Se va leyendo el fichero y enviando el contenido al channel.
* Twitter: Conecta a la API de Twitter con las credenciales de tu usuario.
* Kafka: Mensajes almacenados en Kafka.
* Netcat: Se lee desde un puerto. Cada línea de texto es un evento.
* Sequence Generator: Generador secuencial de eventos.
* Syslog: El syslog de la máquina.
* Http: Eventos desde petición HTTP GET o POST.
* Stress: Simula un test de carga.
* Legacy: Eventos de agentes Flume más antiguos.
* Custom: Tiene que configurarse mediante una clase Java propia implementando las interfaces base.
* Scribe: Ingesta propia, utilizable *junto a* Flume.
### Sinks por defecto
* HDFS: Almacena eventos el sistema de archivos de Hadoop, en formato text y sequenceFile. Permite compresión.
* Hive: Almacena en texto o JSON en tablas o particiones de Hive. Transaccionalidad de Hive.
* Logger: Log INFO para guardar los eventos.
* Avro: Host/port de Avro.
* Thrift: Lo mismo.
* IRC: Usa un chat IRC.
* FileRoll: Sistema de ficheros local.
* Null: Se tiran.
* Hbase: Se almacenan en una base de datos Hbase, usando un serializer específico. Autenticable mediante Kerberos.
* MorphlineSolr: Transforma los eventos y los almacena en un motor de búsqueda Solr.
* ElasticSearch: Se almacenan en ElasticSearch
* Kite Dataset: Se almacenan en Kite (una capa de Hadoop)
* Kafka: En un topic de Kafka.
* Custom: Lo mismo que los sources, tienen que configurarse específicamente.
### Channels por defecto
* Memoria: Los eventos se almacenan en memoria de tamaño predefinido.
* JDBC: Persistidos en una base de datos, hay que definir driver, url, etc.
* Kafka: Clúster de Kafka. Alta disponibiilidad y replicación.
* File: En un fichero en el sistema local.
* Spillable Memory: En una cola en memoria. Si se sobrecarga la misma se pueden guardar en disco.
* Pseudo Transaction: Testing.
* Custom Channel: Pues eso mismo.
### Interceptores por defecto
* Timestamp: Agrega una timestamp en la cabecera.
* Host: Añade Host o IP al evento.
* Static: Cabecera fija.
* UUID: Identificador único.
* Morphline: Transformación predefinida en un fichero de configuración de la transformación.
* Search&Replace: Busca y reemplaza una cadena en el evento.
* Regex: Lo mismo pero con expresiones regulares.

Para más información mirar los ejercicios resueltos de flume.