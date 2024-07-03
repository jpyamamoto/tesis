Encima de la construcción básica que son los arreglos mutables, haciendo uso de los tipos lineales, es posible proveer distintas abstracciones más complejas para tener arreglos con propiedades específicas.

Una de las tales abstracciones son los Arreglos Destino. Este tipo de arreglos están restringidos a construirse únicamente mediante operaciones de escritura, funcionando como una estructura intermedia.

Se dice que es una estructura intermedia, puesto que únicamente se utiliza durante la fase de inicialización del arreglo. Al concluir la construcción del arreglo, se convierte en un arreglo de sólo lectura, donde puede ser utilizado de forma segura por el resto del programa.

El nombre de los arreglos destino viene de que usan una versión restringida de lo conocido como *estilo de paso de destino* ([[Destination Passing Style]]).

Al igual que los arreglos mutables simples, los arreglos destino cuentan con la ventaja de no necesitar optimizaciones del tipo [[Fusión por Atajo]] para garantizar su eficiencia. Comparten la característica de que la construcción del arreglo debe suceder dentro de una *continuación*. No obstante, el resultado de tal continuación no será un objeto arbitrario, sino el objeto de tipo unitario `()`, y al salir de la continuación se nos devuelve el arreglo construido en su versión de sólo escritura.

La función principal que caracteriza a los arreglos destino es la de asignación:

```haskell
alloc :: Int -> (DArray a ⊸ ()) ⊸ Array a
```

Como es posible imaginar, la continuación constructora no tiene todo el poder expresivo de los [[Arreglos Mutables]], puesto que los valores que escribe en el arreglo no pueden depender del arreglo más que en su longitud. Esto es porque la interfaz de `DArray a` no cuenta con funciones para lectura, únicamente para obtener la longitud:

```haskell
size :: DArray a ⊸ (Unrestricted Int, DArray a)
```

Además, la única forma de escribir al arreglo es mediante las funciones provistas por el mismo módulo, puesto que no se brinda acceso a la estructura interna de `DArray` como tipo de dato algebráico.

Entre las pocas formas que existen para poblar estos arreglos tenemos el uso de una función que obtiene el valor según el índice al que escribe:

```haskell
fromFunction :: (Int -> b) -> DArray b ⊸ ()
```

Y la función constante, que es fácilmente definible a partir de la anterior:

```haskell
replicate :: a -> DArray a ⊸ ()
replicate x = fromFunction (const x)
```

También tenemos la siguiente función para consumir arreglos vacíos:

```haskell
dropEmpty :: DArray a ⊸ ()
```

Su comportamiento consiste en consumir el arreglo (para garantizar su linealidad) y retornar `()` cuando el arreglo es vacío. Su utilidad puede no ser aparente, pero veremos que es una función necesaria cuando lleguemos a los [[Polarized Arrays]].

Un caso de uso interesante para los arreglos destino es el presentado en [[Spiwack_2020a]] donde utiliza una función para dividir en 2 un arreglo destino, y copiar elementos a cada parte. La firma de tal función es la siguiente:

```haskell
split :: Int -> DArray a ⊸ (DArray a, DArray a)
```

Llevando el ejemplo un poco más lejos, podemos ver aquí una forma de segmentar el arreglo y construirlo por partes. Revisemos el siguiente ejemplo:

```haskell
replicateHalves :: a -> a -> DArray a %1 -> ()
replicateHalves lval rval arr = case size arr of
  (Unrestricted l, arr') ->
    split (l `div` 2) arr' & \case
      (arrLeft, arrRight) ->
        replicate lval arrLeft `lseq` replicate rval arrRight
```

En el ejemplo anterior tenemos una función en que poblamos las dos mitades del arreglo utilizando los valores `lval` y `rval` respectivamente. Cabe resaltar que al dividir el arreglo destino, no se alojan 2 arreglos nuevos, sino que se divide el segmento del arreglo al que se tiene acceso, ahora desde `arrLeft` y `arrRight`.

Esto puede generalizarse y permitir que funciones arbitrarias poblen los segmentos como en el siguiente ejemplo:

```haskell
populateHalves ::
  (DArray Int %1 -> ()) ->
  (DArray Int %1 -> ()) ->
  DArray Int %1 ->
  ()
populateHalves lf rf arr = case size arr of
  (Unrestricted l, arr') ->
    split (l `div` 2) arr' & \case
      (arrLeft, arrRight) -> lf arrLeft `lseq` rf arrRight
```

O incluso generalizarse para poblar por segmentos de distintos tamaños. Esta última idea se deja como ejercicio al lector.

Otra forma de identificar cuándo utilizar un Arreglo Destino en vez de un Arreglo Mutable, es cuando la construcción del arreglo no tiene alguna dependencia de temporalidad. Es decir, es posible calcular el valor en cada posición, no importa en qué orden se escriban.

Si bien los arreglos destino tienen algunas limitantes, sí proveen algunas garantías interesantes.

1. Eficiencia: Al igual que con los arreglos mutables simples, puesto que conocemos de antemano que cada operación es el único consumidor del arreglo, el compilador puede optimizar las instrucciones generadas para trabajar directamente sobre el arreglo *en sitio* de forma segura.
2. Complejidad: Dado que se conoce de antemano que el valor escrito en cada posición del arreglo no puede depender de nada mas que de su propio índice (y a lo mejor, la longitud del arreglo), sabemos que se realiza una única operación de escritura por cada elemento del arreglo. Así, si la complejidad de la función de escritura es $O(f)$, la complejidad de crear el arreglo será $O(nf)$ donde $n$ es la longitud del arreglo.
3. Concurrencia: Ya que los arreglos cuentan con la independencia temporal que se mencionó antes, pueden ser fácilmente construidos en paralelo garantizando estar libres de condiciones de carrera.
