# Ejercicios Spark

## Módulo 1

1. Lo primero que debemos hacer es arrancar Spark y su contexto. El contexto se crea automáticamente al arrancar Spark. Podemos comprobar la existencia de este contexto:
```
$ spark-shell
[datos varios de inicio]
scala> sc
res0: org.apache.spark.SparkContext = org.apache.spark.SparkContext@7c651424
```
Vamos a usar una carpeta compartida en la máquina virtual para la realización de estos ejercicios. En caso de no tenerla configurada, es el momento de hacerlo.

## Módulo 2

2. Vamos a crear un RDD con un archivo llamado "relato.txt" que esá incluido entre los archivos a importar.
3. 