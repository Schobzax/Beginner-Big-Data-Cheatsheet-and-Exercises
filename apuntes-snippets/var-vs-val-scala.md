# Diferencias entre Var y Val en Scala

**Var** = **Var**iable.
**Val** = **Va**riable fina**l**.

El objeto asignado a un **val** no puede ser reemplazado.

El objeto asignado a un **var** sí.

Dicho objeto **puede modificarse en su estado interno** en ambos casos.

## Ventajas
* Si un objeto no cambia su estado interno, no tienes que preocuparte de que otra parte de tu código lo cambie. (Muy importante en sistemas multihilo). **Usando val**.
* Sin embargo, la mutabilidad podría ser importante. En ese caso, **usar val**.

En general, **es mejor usar val**, genera un código **más seguro**.