# Ejercicios Introducción a Hadoop

Voy a saltarme en mayor parte las partes que no corresponden a código o programación activa.

## HDFS

1. Lo primero que tenemos que hacer es habilitar la transferencia de ficheros entre vuestra máquina virtual y vuestro host.
   ```
   En VirtualBox esto se realiza creando una carpeta compartida en la configuración de carpetas compartidas de la máquina virtual.
   ```
2. Una serie de comandos (ver [Hadoop Cheatsheet](../apuntes-snippets/hadoop-cheatsheet.md) para más información) realizando varias tareas básicas.
   1. `hadoop fs -put` => local -> hdfs
   2. `hadoop fs -get` => hdfs -> local
3. Visualización últimas líneas de un archivo de texto, usando `cat` y `tail`
   - `hadoop fs -cat <hdfs-route: route> | tail -n <nlineas: int>`

## MapReduce

Ejecutamos el mismo proceso que hemos realizado al pasar un archivo de local a hdfs, excepto que esta vez es el contenido de un programa Java para realizar un conteo de palabras. En el paquete se incluyen los archivos .java, los .class compilados, y el paquete .jar para la ejecución.

1. Ejecución del programa
   ```
   hadoop jar wc.jar solution.WordCount shakespeare /user/cloudera/wordcounts

   -- Estamos ejecutando un Job (v. teoría) a partir del archivo wc.jar, usando la clase solution.WordCount en el archivo hdfs de origen (penúltimo archivo) y sacándolo por el archivo de destino (último parámetro)
   ```

   Luego nos pide ejecutarlo otra vez, pero nos dará error, porque el fichero de salida ya existe. Este no puede estar creado para que el programa termine con éxito.

2. Observar el resultado
   ```
   hadoop fs -cat /user/cloudera/wordcounts/part-r-00000

   -- Esto nos muestra por pantalla el resultado de la ejecución.
   -- IMPORTANTE: Al haberse usado solo un reduce, esto conlleva un solo archivo de parte. En un ejercicio posterior observamos que hay más de un archivo.
   ```

3. Lo ejecutamos nuevamente con un archivo de destino distinto e intentamos ver la ejecución con el siguiente comando:
    ```
    mapred job -list --Muestra una lista de los Jobs de MapReduce en proceso.
    ```

4. La única manera de detener un Job es en la terminal con el siguiente comando. Cerrando la terminal este no termina.
   ```
   mapred job -kill <JobID>
   ```