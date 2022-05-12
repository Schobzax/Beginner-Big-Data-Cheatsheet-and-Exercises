# Scala vs. Python: Traducciones

En este documento se verán una serie de traducciones de los distintos comandos y sus diferencias respecto a la otra versión.

## Introducción
A lo largo de la realización de estos ejercicios me he visto en la necesidad de corregirme muchas veces por las diferencias que hay entre un lenguaje y otro, así que he decidido reunir esa serie de diferencias entre los dos lenguajes en un solo documento.

## La tabla

No está ordenado ni agrupado ni organizado, por ahora haz un Ctrl+F.

Por otro lado, hay algunos comandos que pueden usarse en ambos lados pero están puestos porque acostumbro a ponerlo de una manera en un sitio y de otra manera en otro y porque en algún momento me han dado alguna clase de fallo.

| Comando | Python | Spark |
| ------- | ------ | ----- |
| Declaración de variables | `ejemplo = [valor]` | `val ejemplo = [valor]` |
| display [^display] | `df.display()` | `display(df)` |
| print | `print()` | ``println()`` |
| print variable | `print("%d" % [variable/operacion])` | `print(s"$variable")` |
| Llamar a columnas | `col("nom_colum")`, `"nom_colum"`, `df.nom_column` | lo otro además de `$"nom_colum"` |
| Ordenación | `.sort(col("A").desc()/asc())` | `.sort(col("A").desc/asc)` |
| import [^import] | `import org.apache.spark.sql.functions._` | `from pyspark.sql.functions import *` |
| Arrays | `[(valor1.1,valor1.2),(valor2.1,valor2.2)]` | `Array((valor1.1,valor1.2),(valor2.1,valor2.2))` |
| Acceso a columnas | `df.columns[0]/["Id"]` | `df.col(0)/("Id")` |
| Parámetros en funciones | Python puede ponerlos en cualquier orden porque puede hacer `funcion(parametro=valor)` | Spark tiene que ponerlos en orden porque sus funciones son posicionales |
| Comparación, operadores [^comp] | De a dos, `!=` o `==` | De a tres, `=!=` o `===` |
| Aliases | `col("ASDF").alias("ZXVC")` | `col("ASDF") as "ZXVC"` |
| Elementos de una row [^row] | `print(row[0])` | `val row0 = row.getInt(0)` |
| Convencion de nombrado de variables [^conv] | `snake_case` | `camelCase` |
| DataSets, Case Class y demás asuntos relacionados | `Nope` | `Ye` |
| printSchema | `df.printSchema()` | `df.printSchema` |
| Definición de UDF | `def func(param):` y debajo tabulado pues la función con un return | `val func = (param: Type) => { [aquí la función] }` |
| Registro de UDF | `spark.udf.register("nombreFunc", func, [Type()])` | `spark.udf.register("nombreFunc", func)` que como ya tiene el tipo puesto antes pues no hay que ponerlo |
| Pandas | Ye | Nope |
| Import Window | `from pyspark.sql.window import Window` | `import org.apache.spark.sql.expressions.Window` |

[^display]: Por lo visto `display(df)` funciona en python también. No lo sabía. Pero es cierto que lo de python no funciona en scala.
[^import]: La biblioteca concreta podría variar pero la sintaxis general es más o menos esa.
[^comp]: No del todo acertado, seguramente haya otras cuestiones asociadas al por qué son tres pero en mi experiencia personal he usado tres en Scala y dos en Python y por eso está puesto ahí.
[^row]: Basado en una cosa que escribí y que en verdad no está bien, pero en esencia lo que significa es que para Scala tienes que especificar el tipo. O algo.
[^conv]: Esto no tiene nada que ver con nada, pero me resultó curioso verlo así en los ejercicios resueltos así que lo pongo.

## TODO: Ordenar la tabla de alguna manera