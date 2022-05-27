En este hay poco que contestar.

## Pregunta 1

**¿Serías capaz de listar todos los topics existentes?**

Sí. `kafka-topics.sh --zookeeper 127.0.0.1:2181 --list`.

## Pregunta 2

**¿En otra consola serías capaz de revisar el contenido de este topic recién creado?**

Esta pregunta puede interpretarse de dos maneras.

La primera: Describir el topic. Esto se hace con `kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic <nombre> --describe`

La segunda: mostrar el contenido. Esto se puede hacer creando un consumer que muestre todos los datos escritos desde el principio.

```
$ kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic topic_app --from-beginning
```
(Si no se pone grupo parece que lee de todos los grupos)

## Ejercicio 1 o 3

**Crear un nuevo topic que maneje datos en formato JSON con variedad de atributos (que tengan cadenas de caracteres, enteros, y tipos de datos reales/flotantes) inserta varios registros (al menos unos 6-10 registros) y crea una tabla en presto asociada a ese topic y lanza consultas sobre el topic (que incluso podría estar en una continua evolución, asumiendo que sigan insertando elementos) con condiciones (cláusulas WHERE) basadas en algunos de los atributos que contienen dichos JSON.**

```
$ kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic otro_topico --create --partitions 3 --replication-factor 1
$ kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic otro_topico
> {"campo1":"valor1","campo2":"valor2", etcétera} -> De esto se escriben entre 6 y 10 filas. Mucho cuidado con las comillas.
```
Cancelamos con Ctrl+C el producer, y situados en la misma carpeta de kafka:
```
$ nano etc/catalog/kafka.properties
```
Modificamos la lista de tablas añadiendo el nombre del tópico recién creado. (Si la carpeta catalog no está hay que crearla)
```
$ nano etc/kafka/otro_topico.json
```
Este archivo que acabamos de crear tendrá el siguiente contenido.
```
{
  "tableName":"otro_topico",
  "topicName":"otro_topico",
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
Una vez realizado esto, iniciamos el servidor: `~/presto-server-0.213/bin/launcher start`. Si no ha habido ningún problema y todos los archivos están bien escritos (que yo me dejé una coma en el JSON y me cachis en la mar)[^consejo]

[^consejo]: En el vídeo indica que si en vez de poner `bin/launcher start` pones `bin/launcher run`, te lo ejecuta en primer plano y puedes ver los errores. Me imagino que eso me hubiera ayudado pero no lo sabía entonces.

Por último, abrimos el CLI de consola.
```
$ cd $HOME
$ ./presto --catalog kafka --schema default
presto:default> show tables; // Este comando debería mostrar todos los tópicos incluidos en el archivo etc/catalog/kafka.properties.
    Table
-------------
 csv_topic
 json_topic
 otro_topic
 txt_topic
(4 rows)

Query blablabla

presto:default> select * from otro_topic;
```
Este otro comando muestra una tabla (con todos los campos "ocultos" propios de kafka) en formato -less con los datos insertados.

Y bueno, ya a partir de aquí se pueden hacer todas las consultas que se quiera.