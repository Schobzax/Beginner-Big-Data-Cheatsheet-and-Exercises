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