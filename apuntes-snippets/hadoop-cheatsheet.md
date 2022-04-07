# Hadoop Cheatsheet

Es importante destacar que HDFS no es el sistema local. Son dos sistemas separados. HDFS puede acceder al sistema local, pero no al revés.

## HDFS

* `hadoop fs` - Listado de comandos disponibles para HDFS
  
  En su mayoría, casi todos los comandos son equivalentes a comandos linux: `ls`, `rm`, `cat`, entre otros funcionan de manera similar. Algunos ejemplos:
  * `hadoop fs -ls <directorio linux>` - Lista los archivos en ese directorio *de HDFS*.
  * `hadoop fs -rm <archivo>` - Borra un archivo en HDFS.
  * `hadoop fs -cat <archivo>` - Mira los contenidos de un archivo por la salida estándar (la consola).
  
  Otros comandos:
  * `hadoop fs -put <orig> <dest>` - Coloca un archivo "orig" en el sistema local en una localización "dest" en el hdfs
  * `hadoop fs -get <orig> <dest>` - Coloca un archivo "orig" de HDFS en la localización "dest" del sistema local.

* `hadoop jar <jar-file> <solution-class> <orig> <dest>` - Ejecuta un archivo jar determinado en jar-file y usando la solution-class ejecuta un *MapReduce* en el archivo "orig" y lo escribe en "dest".

## MapReduce

* `mapred job -list` - Muestra la lista de Jobs en ejecución.
* `mapred job -kill <JobID>` - Detiene un Job en ejecución a partir de su JobID.