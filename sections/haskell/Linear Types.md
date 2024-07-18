`LinearTypes` es el nombre de la extensión del compilador principal de Haskell: GHC; para el uso de tipos lineales.

En su primera versión, se identificó que la implementación afectó únicamente a 1152 líneas, de un total de más de 100 mil líneas que constituyen al compilador. Esto es evidencia de la buena interacción que representa el uso de tipos lineales en conjunto con Haskell sin extensiones.

La extensión introduce el siguiente tipo:

```haskell
data Multiplicity = One | Many
```

que sirve para enumerar las dos posibles multiplicidades. E introduce los siguientes tipos:

```haskell
type a %1 -> b = a %One  -> b
type a    -> b = a %Many -> b
```

Podemos ver que las funciones con argumentos no lineales se continúan escribiendo como `a -> b`, siendo este ahora un alias de tipo. Además, al utilizar el pragma `UnicodeSyntax` es posible escribir `a ⊸ b` en vez de `a %1 -> b`. En adelante, preferiré la sintaxis con ⊸ para evitar el exceso de notación.

También es posible expresar tipos con multiplicidad polimórfica `a %p -> b`. En tales casos, durante la compilación se generan dos funciones: una en su versión lineal y otra no-lineal, donde el código de ambas debe ser el mismo (esto lo reviso en mayor profundidad en [[Polimorfismo de Multiplicidad]]).