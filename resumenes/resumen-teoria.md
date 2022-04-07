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