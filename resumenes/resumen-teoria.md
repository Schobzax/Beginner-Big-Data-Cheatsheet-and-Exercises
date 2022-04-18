# Resumen Teoría

## Hadoop

Es un entorno de trabajo para software centrado en el big data y enfocado en trabajo distribuido.

Se divide en varias partes:
1. HDFS: Sistema de archivos
2. MapReduce: Algoritmo que utiliza
3. Herramientas usadas: Hive, Impala, Pig...

### Definiciones

- **Cluster**: Conjunto de nodos (servidores) trabajando en común.
  - Existen nodos maestros (gestionan) y esclavos (procesan)
  - Cada nodo tiene demonios propios.
- **Job**: Trabajo que se realiza en total en todos los nodos, coordinado por un nodo maestro. Se divide en Tasks. Ejecución completa.
- **Task**: Tarea ejecutada en un nodo esclavo. Parte de un Job. Ejecución individual.

### HDFS

Sistema de archivos propio de Hadoop. Distribuye y replica los datos entre los distintos nodos de forma *transparente* al usuario.

**HDFS solo es accesible mediante Hadoop, mediante el comando** `hadoop fs`. **Hadoop, sin embargo, sí puede acceder al sistema de archivos locales.**

Hadoop --> Local

Local --x Hadoop

Existe una versión gráfica para ver los trabajos en proceso y pasados.

### MapReduce

Paradigma de programación que realiza tres etapas:
1. Fase *Map*
2. Fase *Shuffle&Sort*
3. Fase *Reduce*

Sirve para distribuir tareas a lo largo del clúster; con paralelización y distribución automática. Tolerante a fallos.
Abstracción para programadores.

Previo al comienzo, el nodo maestro se encarga de repartir las tareas.

#### 1. Map

Actúa sobre cada registro y está dividido en tasks. Cada Task se realiza en su propio nodo, que almacena el bloque de datos. Del Map sale un par (K,V): clave, valor.

#### 2. Shuffle & Sort

Ordena por clave y agrupa los datos intermedios que salen de los Mappers en formato (K,V).

#### 3. Reduce

Recibe la salida del proceso anterior y realiza operaciones sobre la misma para conseguir la salida final.

### Ecosistema y herramientas

- Hive: Sistema de consulta y manipulación de datos en el HDFS. Funciona en HQL (parecido a SQL). Traduce Queries HQL a MapReduce.
- Pig: Plataforma para analizar datasets grandes en HDFS. Tiene lenguaje propio (Pig Latin). Traduce PigLatin a MapReduce.
- Sqoop: Su función principal es traer datos desde BBDD relacionales a HDFS. Parte configuración, parte SQL. Todo tipo de conectores.
- Flume: Importación de datos en un clúster en tiempo real, pensada para orígenes distintos a BBDD relacionales (IoT, correo, web, logs...). Consta de source, channel y sink.
- Kafka: Plataforma que puede verse como una cola de mensajes publicación-suscripción. Concebida como registro de transacciones.
- Bases de Datos NoSQL: pensadas para datos multiestructurados. Permiten distribución y ejecución paralela. No garantizan ACID, pero muy escalables horizontalmente.
- Neo4j: Para trabajar con grafos.

Expandiremos sobre algunas de estas herramientas más adelante.

## Hive

**Hive** es una infraestructura para almacenaje y consulta de datos basada en Hadoop. Usa un lenguaje similar a SQL-92 llamado HQL.

- +Buena escalabilidad
- +Tolerancia a fallos
- -Alta latencia (procesamiento por lotes)
- -No ofrece consultas en tiempo real.
- -Mal rendimiento con sistemas tradicionales.

*Nota personal*: Si estás familiarizado con el uso de herramientas de bases de datos relacionales en consola, como MySQL, **es extremadamente parecido**.

### Definiciones

* Tablas: Unidades de datos homogéneas que comparten esquema.
* Particiones: Cada tabla puede tener claves de particionado que determinan cómo se almacenan, para optimización de consultas.
* Buckets: Los datos dentro de cada partición pueden dividirse en *buckets* según el valor de una función de dispersión. Optimizan *joins*.

### Creación de tablas

La creación de tablas tiene una serie de consideraciones adicionales:

```
CREATE [EXTERNAL] TABLE ejemplo (
  ejemplo_campo TYPE COMMENT 'Campo de ejemplo' --Descripción del campo
  otro_campo TYPE
) COMMENT 'Tabla de ejemplo' -- Descripción de tabla
PARTITIONED BY(abc STRING) -- Particionada por el campo "abc"
CLUSTERED BY (campo) -- Agrupada en buckets por el campo "ejemplo_campo"
SORTED BY (otro_campo) -- Ordenado dentro del bucket por el campo "otro_campo"
INTO x BUCKETS -- En un número de buckets o clusters.
ROW FORMAT DELIMITED -- El formato de las filas consiste en campos separados por un delimitador
FIELDS TERMINATED BY ',' -- En este caso, dicho delimitador será una coma
COLLECTION ITEMS TERMINATED BY ';' -- El delimitador de colecciones (campos array, campos mapa, campos struct, etcétera) será el punto y coma
MAP KEYS TERMINATED BY '.' -- El delimitador de maps será el punto.
LINES TERMINATED BY '*' -- El delimitador de fin de línea es el asterisco (por defecto es el salto de línea, \n)
STORED AS SEQUENCEFILE; -- El formato de almacenamiento es SEQUENCEFILE
[LOCATION '/dentro/del/hdfs] -- Una tabla externa (EXTERNAL) necesita especificar su localización en el hdfs.
```
El resto de comandos son similares al uso de un motor de base de datos por consola, con la excepción de las cuestiones de carga de datos, que funcionan como un `INSERT INTO tabla SELECT * FROM otratabla`.

Para una información más sucinta, resumida y comparada con Impala, ver [El documento que los compara.](../apuntes-snippets/hive-impala-diff.md)

## Impala

**Impala** es un motor SQL de gran rendimiento gracias al procesamiento en paralelo de grandes volúmenes de datos.

Tiene **muy baja latencia**, gracias a que los servicios que ejecuta se inician con el propio motor, en lugar de antes de cada consulta.

La ventaja de usar Impala, además de las ya mencionadas, es que es más fácil y rápido que usar MapReduce. Tan solo hay que ver ejemplos de código, y vemos que algo que en Java puede tomar docenas de líneas, en SQL apenas es un puñado.

* Hay que destacar que Hive soporta ciertas características que Impala aún no ha implementado, como DATE, JSON, algunas funciones complejas de agregación, sampling, vistas laterales, más de un DISTINCT por consulta, etc.

Impala funciona en el cluster de la siguiente manera: **cada nodo posee un *daemon* Impala**, que se encarga de planificar la query cuando el cliente se conecta con el *daemon* local, "coordinador". Este pide una lista de *daemons* Impala activos, guardada en una lista que se va guardando periodicamente, y distribuye la query entre los que están disponibles.

Pero lo que distingue a Impala de otros sistemas similares es la **caché de metadatos**. Los metadatos se cachean al inicio de la sesión y solo se cambian cuando hay cambios en el *metastore* (cambios en los datos o estructura de la Base de Datos, para entendernos). Esta caché agiliza mucho el trabajo de Impala.

El problema: no registra todos los cambios como cambios en los metadatos. Algunos cambios, como los realizados mediante Hive, Pig, Hue, o datos a pelo en el HDFS, no son registrados, y por lo tanto la caché deja de ser válida. Necesita un **refresco manual** de los datos, para lo cual hay operaciones concretas que se pueden realizar.

### Tolerancia a fallos

Hive tiene tolerancia a fallos porque su procesamiento se basa en MapReduce. Si un nodo falla, otro puede hacerse cargo de esa parte.

Sin embargo, Impala no posee tal tolerancia a fallos. Si un nodo falla durante la ejecución, falla la ejecución completa. Lo único que se puede hacer es volver a lanzar la query.

### Optimización

Hay un comando concreto, llamado `COMPUTE STATS <nombre_tabla>`, que permite informar a Impala de las características de las tablas involucradas en una consulta. Esto optimiza la ejecución de la consulta. Se recomienda su uso justo después de cargar las tablas o cuando se modifique una tabla en más de un 20%.

Esta información puede mostrarse mediante `SHOW TABLE STATS <nombre_tabla>` o `SHOW COLUMN STATS <nombre_tabla>`;

### Comandos

Igual que Hive: similares a SQL.

## Pig

**Pig** es cerdo en inglés. También es una herramienta similar a Hive o Impala, excepto que usa su propio lenguaje: el **Pig Latin**.

**Pig Latin** es un lenguaje de flujo de datos, representado por una secuencia de instrucciones.

Para más información, consultar la chuletilla de cerdo. Perdón, la [cheat sheet de Pig](../apuntes-snippets/hadoop-cheatsheet.md#pig-y-pig-latin).

## Sqoop
Sqoop es una herramienta para transferir datos entre RDBMs y Hadoop. Ese es su objetivo fundamental y su *raison d'être*, su propósito, su alfa y su omega.

Dispone de varias opciones para transferir una, parte de una o más de una tabla, con soporte para cláusula `WHERE`.

Usa MapReduce para la importación de datos. Por defecto usa cuatro Maps pero puede configurarse o determinarse en ejecución. Además, posee paralelización y tolerancia a fallos. Los datos se van importando registro a registro. Utiliza JDBC.

Al importar mediante MapReduce los archivos se guardanen HDFS con el formato `part-*.0*`. Puede usarse para importaciones incrementales:
* La primera vez que se ejecuta el `import` cargas todos los datos de una tabla.
* La segunda y consiguientes veces se cargan los datos creados desde la última importación.

Hay conectores específicos para cada motor. Por otro lado, está el modo **direct**, que conecta con la base de datos sin pasar por JDBC siendo más rápido.

El resto es sintaxis, que básicamente es `sqoop herramienta [opciones]`. Para más información acudir al [cheatsheet](../apuntes-snippets/hadoop-cheatsheet.md#sqoop).

## Flume
Flume es un sistema para mover grandes cantidades de datos de diversas fuentes a un almacén centralizado.
### Características
* Arquitectura sencilla y flexible.
* Robusto
* Tolerante a fallos
### Partes
* Event: Unidad de dato a propagar.
* **Source**: Origen.
* **Sink**: Destino.
* **Channel**: Buffer de almacenamiento intermedio que conecta el *Source* con el *Sink*.

Para que Flume funcione debemos configurar correctamente estos tres últimos.

Esto se realiza configurando un "agente" mediante un fichero local.

Para lanzar el agente es necesario el nombre del agente, el directorio de configuración y el archivo de configuración.

### Conexión única / múltiple

Es fácil configurar varios agentes. Un solo agente debe conectar *source* y *sink* a través del *channel*.

En caso de que haya más de un agente, la diferencia consiste en que el *sink* del primer agente debe coincidir con el *source* del segundo. Ambos han de ser de tipo AVRO (sistema de serialización de datos).

Puede haber un caso en el que un único agente deba mandar la información por múltiples *flows*, de manera que se multiplexan los eventos en función de ciertas condiciones. (Un poco como un switch).

### Definiciones

* Source: Fuente de datos, parametrizable. Algunos tipos incluyen Avro, Exec, Spooling directory, Twitter, Kafka, Netcat, Http, Legacy, Custom, etc.
* Sink: Extraen eventos de un channel y se almacenan en un sistema externo o se envían al siguiente agente. Algunos tipos incluyen HDFS, Hive, Logger, Avro, IRC, FileRll, Null, Hbase, Kafka, Custom, etc.
* Channel: Por dónde pasan los eventos. Algunos tipos incluyen Memoria, JDBC, Kafka, File, Custom Channel, etc.
* Interceptor: Modifica el evento en caliente.  Algunos tipos incluyen Timestamp (inserta una timestamp en la cabecera), Host (añade host o IP), Static (cabecera fija), Search&Replace, Regex, etc.