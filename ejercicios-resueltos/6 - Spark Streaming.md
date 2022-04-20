# Ejercicios Spark Streaming

## 6.1 - Spark Streaming I

1. Lo primero que vamos a hacer es realizar un ejercicio parecido a  "A Quick Example" localizado [aquí](https://spark.apache.org/docs/1.5.2/streaming-programming-guide.html). Para empezar, debemos iniciar `netcat` en una terminal:
```
$ nc -lkv 4444
```
Ahora todo lo que escribamos ahí se enviará por el puerto 4444 de localhost.

2. Lo siguiente que debemos hacer es iniciar una shell de spark con al menos 2 hilos para poder ejecutar el ejercicio:
```
$ spark-shell --master local[2]
```

3. Lo siguiente es realizar los `import` necesarios.
```
scala> import org.apache.spark.streaming.StreamingContext; import org.apache.spark.streaming.StreamingContext._; import org.apache.spark.streaming.Seconds
```

4. A continuación creamos un `StreamingContext` con una duración de 5 segundos.
```
scala> val ssc = new StreamingContext(sc,Seconds(5))
```

5. Ahora creamos un DStream que lea del puerto 4444, el que acabamos de poner, en nuestra máquina.
```
scala> val mystream = ssc.socketTextStream("localhost",4444)
```

6. Por último contamos las palabras.
```
scala> val words = mystream.flatMap(line => line.split("\\W"))
scala> val wordCount = words.map(x => (x,1)).reduceByKey(_+_)
```
7. Por último imprimimos el resultado. Nos dará vacío.
```
scala> wordCount.print()
```

8. Ahora arrancamos el contexto.
```
scala> ssc.start()
```

Ahora, tristemente si se ejecuta desde consola, la única manera de salir es mediante Ctrl+C.
Pero todo lo que se escriba ahora en netcat debería verse reflejado en algo parecido a esto:
``````
--------------------------------------------
Time: 1650451930000 ms
--------------------------------------------
(los,1)
(mucho,1)
(galgo,1)
(acordarme,1)
(cuyo,)
(hidalgo,1)
(corredor,1)
(adarga,1)
(mancha,1)
(la,1)
``````

## 6.2 - Spark Streaming II

