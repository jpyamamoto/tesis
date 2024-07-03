A continuación presentamos la solución que se propone en [[Bernardy_2018]] para incorporar arreglos de una forma segura y eficiente en lenguajes puramente funcionales con tipos lineales, particularmente en Haskell. Las firmas de funciones están adecuadas para coincidir con las firmas de la implementación de la biblioteca `linear-base`, aunque los nombres de funciones y constructores difieren.

Recordemos que los problemas para incorporar arreglos en lenguajes con transparencia referencial eran:
- Realizar copias de arreglos es ineficiente. Se pierde su garantía más útil que es operaciones de lectura/escritura con complejidad $O(1)$.
- Si no se realizan copias de los arreglos, hay que recurrir al uso de funciones inseguras, dejando en el programador la responsabilidad de escribir en arreglos marcados como inmutables.

Los tipos lineales resuelven ambos problemas. Al garantizar que únicamente se consume el arreglo una vez por cada operación, pueden realizarse tales operaciones de forma segura, y sin necesidad de realizar copias.

De esta forma tenemos garantías de memoria, no debido a la fase de optimización del compilador, sino por la misma semántica del programa escrito.

La API propuesta es la siguiente.

```haskell
type MArray a
type Array a

newMArray :: Int -> (MArray a ⊸ Unrestricted b) ⊸ b
write :: MArray a ⊸ Int -> a -> MArray a
read :: MArray a ⊸ Int -> (MArray a, Unrestricted a)
freeze :: MArray a ⊸ Unrestricted (Array a)
```

Contamos con 2 tipos:
- `MArray a`: arreglos mutables de tipo `a`
- `Array a`: arreglos inmutables de tipo `a`

Y 4 funciones para operaciones sobre arreglos mutables. Recordemos que únicamente los arreglos mutables presentan problemas de inseguridad o ineficiencia, por lo tanto la única operación para arreglos inmutables `read :: Array a -> Int -> a` no requiere mayor cuidado.

Revisamos a continuación una por una las funciones anteriores.

La función `newMArray` sirve para definir un nuevo arreglo, asignando su espacio en memoria. Su primer argumento es el tamaño que tendrá y el segundo es una **continuación** que transforma un arreglo mutable en algún resultado sin restricciones de linealidad. El resultado de `newMArray` es el resultado de la continuación. Resulta importante notar que la continuación impone una condición de linealidad sobre el arreglo; es decir, únicamente se puede hacer uso del arreglo linealmente, al igual que con la continuación misma.

La función `write` escribe en el arreglo recibido un elemento de tipo `a` en la posición dada por un entero, devolviendo un nuevo arreglo.

La función `read` para arreglos mutables, a diferencia de la función para `Array a` consume su primer argumento linealmente y un índice entero. Como resultado devuelve un arreglo mutable (que debería ser igual al dado como primer argumento) y el elemento que se encontró en la posición dada, envuelto en `Unrestricted` para indicar que puede hacerse uso de este sin condiciones de linealidad.

Finalmente, la función `freeze` transforma un arreglo mutable en uno inmutable, haciendo explícito el hecho de que no tiene restricciones de linealidad. Puesto que `freeze` consume el arreglo mutable, no existe el problema de que una copia del arreglo mutable sea modificada.

## ¿Por qué `Unrestricted`?

Recordando que Haskell implementó linealidad a nivel de funciones, no es posible indicar cuántas veces se va a consumir un recurso a lo largo del programa. Lo que debemos hacer es indicar cuántas veces se va a consumir un recurso relativo a la función que lo recibe como parámetro.

Para entender el uso de `Unrestricted`en las firmas anteriores, basta entender el caso de `read`.

```haskell
read :: MArray a ⊸ Int -> (MArray a, Unrestricted a)
readWrong :: MArray a ⊸ Int -> (MArray a, a)
```

La única diferencia entre `read` y `readWrong` está en que una retorna el elemento en el índice del arreglo como `Unrestricted` y la otra no.

El problema con `readWrong` yace en que, si deseamos utilizar más de una vez el resultado en el par que retorna la función, debemos consumir el par tantas veces como deseemos acceder a su segundo elemento. En ello, habremos consumido varias veces el arreglo mutable en la primera posición del par, lo cuál es indeseable. Ese problema desaparece si el segundo elemento en el par está envuelto por `Unrestricted`, permitiendo tantas copias como sean necesarias.

Para los demás casos, es análogo el argumento para la necesidad de `Unrestricted`.

## ¿Por qué estilo de Paso de Continuaciones?

Como podemos ver en la firma de `newMArray`, se está haciendo uso del estilo de programación conocido como **Paso de Continuaciones**, en donde los argumentos capturan las operaciones a realizarse.

Esto es nuevamente resultado de la linealidad sobre funciones, haciendo imposible garantizar que un recurso se utilice sólo una vez durante la ejecución del programa.

Únicamente podemos garantizar que un recurso se usa linealmente relativo a una función. Por lo tanto, `newMArray` hace obligatorio que el recurso sólo exista dentro de una función (continuación) en donde el nuevo arreglo debe usarse linealmente.