# Diferencias entre Hive e Impala

La diferencia entre Hive e Impala se reduce a la siguiente:

**Si quieres hacer algo rápido sin importar la calidad, usa Impala.**

**Si quieres hacer algo bien aunque tarde un poco más, usa Hive.**

---
## Hive
Hace las cosas bien y tiene corrección de errores.

Si estás familiarizado con el uso de MySQL en consola, es muy parecido.

Hive funciona como abstracción de Hadoop MapReduce en su esencia.
* Útil para tareas intensivas como consultas, análisis, procesos y visualización.
* Las expresiones de consulta se generan en tiempo de compilación.
* Los procesos y servicios tienen que iniciarse en frío, al hacer la consulta. Esto resulta en una mayor latencia.
* Útil cuando importa la compatibilidad igual o más que la velocidad.

## Impala
Impala provee de baja latencia y buen rendimiento a la hora de hacer consultas, con la condición de que el dato esté guardado en clústers Hadoop.

Otra cuestión interesante es que al guardar los mv
* No requiere que se muevan los datos o se transformen, muy eficiente para trabajar en HDFS.
* El código se genera en bucles usando llvm.
* Los procesos y servicios de Impala se encienden junto con Impala, listos para ejecutar las consultas.
* Fácil lectura de metadatos.
* Respuesta rápida mediante procesamiento en paralelo.

## Coste

En términos monetarios, es importante destacar cómo funciona cada servicio internamente.

* Hive funciona encendiendo sus *daemons* antes de cada consulta. Esto implica que el **tiempo** de ejecución, donde el **coste** será **mayor**, es **menos** e interrumpido. Por lo tanto, será **más barato**.
* Por contra, Impala enciende sus *daemons* durante el proceso de encendido. Esto implica que el **tiempo** de ejecución de estos servicios, donde el **coste** será **mayor** es **constante** y desde el principio. Por lo tanto, será **más caro**.

## Tolerancia a fallos

* Hive tiene tolerancia a fallos. Si se cae un nodo, el trabajo puede ser recuperado por el resto.
* Impala no tiene tolerancia a fallos. Si se cae un nodo, se cae el resto de la ejecución.

## Resumen

**Impala**: Consultas ligeras para buscar velocidad. Si falla, hay que relanzar la query.

**Hive**: Mejor para trabajos pesados de ETL. Se busca robustez frente a velocidad. Tolerancia a fallos.

## Apunte extra: ¿Y Pig qué?
Pig es una herramienta parecida a Hive pero que usa su lenguaje propio, **Pig Latin**. Para más información se puede ver el [Resumen de teoría](../resumenes/resumen-teoria.md#pig). Realmente no tiene mayor relación.