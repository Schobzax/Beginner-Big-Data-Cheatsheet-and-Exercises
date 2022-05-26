# Kafka Cheatsheet

## Configuración inicial
Han de estar activos Zookeeper y Kafka Server:
* `zookeeper-server-start.sh config/zookeeper.properties`
* `kafka-server-start.sh config/server.properties`

## Tópicos
* Crear tópico: `kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic <nombre> --create --partitions N --replication-factor M`
  * `N` es el número de particiones.
  * `M` es el factor de replicación.
  * La IP es la de localhost, y el puerto *supongo que* está en la configuración.
  * `<nombre>` es el nombre del tópico.
* Listar tópicos: `kafka-topics.sh --zookeeper 127.0.0.1:2181 --list`
* Describir un tópico: `kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic <nombre> --describe`
  * `<nombre>` es el tópico a describir.

## Productores
* Crear productor: `kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic <nombre>`
  * `<nombre>` es el nombre de un tópico existente.

## Consumidores
* Crear consumidor: `kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic <nombre> --group <grupo>`
  * `<nombre>` es el nombre de un tópico existente.
  * `<grupo>` es el nombre de un grupo de consumidores (existente o no).
* Listar grupos de consumidores (si te fijas no le ponemos nombre al consumidor): `kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --list`
* Describir grupo de consumidores: `kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --describe --group <grupo>`
  * `<grupo>` es el nombre de un grupo de consumidores.