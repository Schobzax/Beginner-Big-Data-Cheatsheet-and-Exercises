# Bloque 1 - Ejercicios parte práctica

## Ejercicio 1
**¿Qué pasaría si cancelamos (utilizando Ctrl+C) uno de los consumidores (quedando 2) y seguimos enviando mensajes por el producer?**

Pasa lo que pasaba antes: al haber un factor de reparto de tres (no sé muy bien como explicarlo), uno de los consumidores se acaba asignando el trabajo de dos consumidores (el cancelado y el de ahora). Pero esto también estaba antes.
Por poner un ejemplo, tanto antes de tener tres como ahora, si insertamos los mensajes 1, 2, 3, 4, 5, 6, 7, 8 y 9; el primer nodo consumiría los mensajes 2, 3, 5, 6, 8 y 9; mientras que el segundo se llevaría los que les corresponde, el 1, 4 y 7. Más o menos la mitad.

## Ejercicio 2
**¿Qué pasaría si cancelamos otro de los consumidores (quedando ya solo 1) y seguimos enviando mensajes por el producer?**

Que este único consumidor acaba llevándose todo lo que se produce.

## Ejercicio 3
**¿Qué sucede si lanzamos otro consumidor pero esta vez de un grupo llamado my-second-application leyendo el topic desde el principio (from-beginning)?**

Aparecen todos los mensajes anteriores. Al añadir nuevos mensajes, cada aplicación (o cada grupo) lee por separado; con lo que los mensajes aparecen en ambos grupos, que al tener un solo consumidor, ese es en el que aparece. Que aparece en los dos consumidores, vaya.

## Ejercicio 4
**Cancela el consumidor y ¿Qué sucede si lanzamos de nuevo el consumidor pero formando parte del grupo my-second-application?¿Aparecen los mensajes desde el principio?**

A ver, si no lo he entendido mal, no, porque no hemos puesto from-beginning. No aparece ningún mensaje, de hecho, solo los que escribimos nuevos.

Sin embargo, si se refiere a cancelar el primer consumidor (con --from-beginning) para que ambos estén en el grupo my-second-application, no imprime absolutamente nada. Por cierto, se repite la replicación 2 para el primero (que es el que acabamos de poner) y 1 para el otro, curiosamente.

## Ejercicio 5
**Cancela el consumer, a su vez aprovecha de enviar más mensajes utilizando el producer y de nuevo lanza el consumidor formando parte del grupo my-second-application ¿Cuál fue el resultado?**

Cancelando este último que hemos generado (el primero), lo que ocurre si adjuntamos --from-beginning es que sigue sin imprimirse el backlog por así decirlo. No sé muy bien por qué, pero sucede.

Que por otro lado, el flujo de mensajes se intercambia: al venir de nuevas, ahora en vez de recibir 2 de cada 3 mensajes, solo recibe 1 de cada 3 pasando los 2 restantes al otro consumidor.

## Ejercicio Final

Lo primero, teniendo activos los dos servicios necesarios:
1. `zookeeper-server-start.sh config/zookeeper.properties`
2. `kafka-server-start.sh config/server.properties`

* **Crear topic llamado "topic_app" con 3 particiones y replication-factor = 1.**
```
$ kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic topic_app --create --partitions 3 --replication-factor 1
```
* **Crear un producer que inserte mensajes en el topic recién creado (topic_app).**
```
$ kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic topic_app
```
* **Crear 2 consumer que forman parte de un grupo de consumo llamado "my_app".**
En una segunda y tercera terminales:
```
$ kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic topic_app --group my_app
```
* **Interactuar con los 3 elementos creando mensajes en el producer y visualizar como estos son consumidos por los consumer.**

En conclusión, lo que ocurre con esto es que el primer consumidor que creamos ha leído los mensajes: 1, 3, 4, 6, 7, 9, 10, 12, 13, 15, 16, 18, 19...

Mientras tanto, el segundo consumidor ha leído los mensajes 2, 5, 6, 11, 14, 17, 20...

* **Aplicar los comandos necesarios para listar los topics, grupos de consumo, así como describir cada uno de estos.**
```
# Lista de topics
$ kafka-topics.sh --zookeeper 127.0.0.1:2181 --list
__consumer_offsets
first_topic
new_topic
topic_app
# Lista de grupos de consumo
$ kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --list
my_app
my-first-application
my-second-application
# Descripción de topics
$ kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic topic_app --describe
Topic:topic_app PartitionCount:3        ReplicationFactor:1     Configs:
        Topic: topic_app        Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_app        Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: topic_app        Partition: 2    Leader: 0       Replicas: 0     Isr: 0
# Descripción de grupos de consumo
$ kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --describe --group my_app
TOPIC       PARTITION   CURRENT-OFFSET  LOG-END-OFFSET  LAG     CONSUMER-ID                                         HOST            CLIENT-ID
topic_app   0           8               8               0       consumer-1-a6fd5a53-c858-413a-b780-5909918ec512     /127.0.0.1          consumer-1
topic_app   1           7               7               0       consumer-1-a6fd5a53-c858-413a-b780-5909918ec512     /127.0.0.1          consumer-1
topic_app   2           7               7               0       consumer-1-ad47ce7a-a545-445d-92da-213dfdee5d1b     /127.0.0.1          consumer-1
```