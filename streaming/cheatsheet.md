# Kafka Cheatsheet

## Configuración inicial
Han de estar activos Zookeeper y Kafka Server:
* `cd kafka_2.11-2.1.0/`
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

## Presto
* `bin/launcher start`, `bin/launcher stop` para iniciarlo y pararlo.

### Agregar una tabla
Para agregar una tabla con datos, hay que seguir los siguientes pasos:
1. Se crea un topic, de nombre `topico`, por ejemplo.
2. Se crea un producer asociado y se insertan datos.
3. Se ha de crear un documento y modificar otro:
   1. Modificamos el documento `etc/catalog/kafka-properties` y en la propiedad `kafka.table-names` añadimos el nombre del tópico que queremos consultar.
   2. Creamos un documento en `etc/kafka/` cuyo nombre sea `topico.json`, con el siguiente contenido:

```
{
  "tableName":"topico",
  "topicName":"topico",
  "schemaName":"default", // Esto es por el schema que se está usando en presto.
  "message": {
    "dataFormat": "json",
    "fields": [
      {
        "name":"campo1",
        "mapping":"campo1",
        "type": el tipo que sea el campo
      },
      {
        otro campo, y así sucesivamente
      }
    ]
  }
}
```
Es un fichero JSON con el schema de la tabla. *Nótese que no se pueden poner comentarios, están ahí a modo explicativo.*

### CLI
* El cliente se lanza desde `$HOME` ejecutando `./presto --catalog kafka --schema default`.