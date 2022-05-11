# Scala vs. Python: Traducciones

En este documento se verán una serie de traducciones de los distintos comandos y sus diferencias respecto a la otra versión.

## Introducción
A lo largo de la realización de estos ejercicios me he visto en la necesidad de corregirme muchas veces por las diferencias que hay entre un lenguaje y otro, así que he decidido reunir esa serie de diferencias entre los dos lenguajes en un solo documento.

## La tabla

No está ordenado ni agrupado ni organizado, por ahora haz un Ctrl+F.

| Comando | Python | Spark |
| ------- | ------ | ----- |
| Declaración de variables | `ejemplo = [valor]` | `val ejemplo = [valor]` |
| display | `df.display()` | `display(df)` |
| print | `print()` | ``println()`` |
| print variable | `print("%d" % [variable/operacion])` | `print(s"$variable")` |
| Llamar a columnas | `col("nom_colum")`, `"nom_colum"` | lo otro además de `$"nom_colum"` |
| Ordenación | `.sort(col("A").desc()/asc())` | `.sort(col("A").desc/asc)` |
| import [^import] | `import org.apache.spark.sql.functions._` | `from pyspark.sql.functions import *` |

[^import]: La biblioteca concreta podría variar pero la sintaxis general es más o menos esa.

## TODO: Ordenar la tabla de alguna manera