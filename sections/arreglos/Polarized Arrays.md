Los **Arreglos Polarizados** es el nombre que se le da en la biblioteca `linear-base` al conjunto de los **Arreglos Pull** y **Arreglos Push**.

Estos representan arreglos interesantes por su capacidad de representar la construcción de un arreglo, sin la necesidad de alojar la memoria para este de inmediato; sólo hasta que es requerido.

Es decir, estas estructuras van a representar cómputos y escrituras de forma abstracta, y sólo hasta ser requerido, se evalúan y escriben en un espacio alojado de memoria para bajar al nivel de arreglos concretos.

Veremos que existe una dualidad en estos arreglos, y que su uso tiende a beneficiarse del trabajo en conjunto de los dos tipos de arreglos, junto con los arreglos mutables:

- Arreglos mutables: valores concretos en memoria con capacidades de lectura y escritura.
- Arreglos Push: cómputos que engloban las operaciones necesarias para la construcción y escritura de un arreglo.
- Arreglos Pull: cómputos que engloban las operaciones necesarias para la definición de los valores en las distintas posiciones de un arreglo. 

Usualmente el ciclo de vida que veremos para los arreglos que utilizan la funcionalidad de arreglos polarizados es el siguiente:

```
Arreglo Pull -> Arreglo Push -> Arreglo Mutable
```

Veamos en qué consisten los dos tipos de arreglos polarizados, para eventualmente hablar de cómo se complementan.
## Arreglos Pull

Los Arreglos Pull también se conocen como Arreglos Demorados puesto que sirven para componer la descripción de la construcción de sus elementos, previo al alojamiento de la memoria y escritura.

La representación más popular de Arreglos Pull en Haskell es la propuesta en [[Keller_2010]], donde se utiliza [[Fusión por Atajo]] para optimizar la implementación. Con la introducción de tipos lineales, se ha desarrollado una implementación alternativa en donde ya no es necesario hacer uso de técnicas de fusión.

Al igual que con los [[Destination Arrays]], los tipos lineales garantizan que las funciones de escritura se ejecutan una única vez al alojar la memoria del arreglo.

No obstante, los Arreglos Demorados cuentan con la particularidad de que se pueden construir y operar previo al alojamiento e inicialización del arreglo concreto.

Veámos la definición del tipo de dato:

```haskell
data PullArray a where
  PullArray :: (Int -> a) -> Int -> PullArray a
```

Notemos que se está usando la notación de Tipos de Datos Algebráicos Generalizados (GADT's) para su definición. El motivo detrás de esto es hacer explícita la no-linealidad en las flechas del constructor. Es decir, este tipo de dato permite acceder a sus componentes en forma no lineal, sin la necesidad de hacer explícito el uso de `Unrestricted`.

Un Arreglo Demorado se construye a partir de una función `Int -> a` que sirve para construir el valor en cada posición del arreglo, y su longitud de tipo `Int`.

La forma más general de construir un arreglo demorado es con la siguiente función:

```haskell
fromFunction :: (Int -> a) -> Int -> PullArray a
fromFunction f n = PullArray f' n
	where f' k
			| k < 0  = error "Índice negativo"
			| k >= n = error "Índice fuera de rango"
			| otherwise = f k
```

Su firma es la misma que la del constructor, pero lo envuelve para hacer la verificación de los índices en el arreglo.

A partir de lo anterior se puede definir muy sencillamente la función para construir un arreglo con todas sus posiciones inicializadas en un valor concreto, conocidas como **funciones de consumo**:

```haskell
fromValue :: a -> Int -> PullArray a
fromValue x n = fromFunction (const x) n
```

Algunas de las funciones para operar los arreglos demorados son las siguientes:

```haskell
map :: (a ⊸ b) -> PullArray a ⊸ PullArray b
append :: PullArray a ⊸ PullArray a ⊸ PullArray a
zip :: PullArray a ⊸ PullArray b ⊸ PullArray (a, b)
foldr :: (a ⊸ b ⊸ b) -> b ⊸ PullArray a ⊸ b
```

Con las funciones anteriores, vemos que los arreglos pull pueden operarse como *Functores* y como *Semigrupos*.

Notemos que las funciones para operar arreglos siempre utilizan los arreglos en argumentos de forma lineal, incluso si los componentes de los arreglos mismos se pueden usar sin restricción.

Esto es para garantizar que la composición de tales funciones será lineal en los arreglos. De tal manera, un único alojamiento de memoria al finalizar la composición de las funciones será suficiente, y no es necesario realizar evaluaciones intermedias.

```tikz
\usepackage{amssymb}
\begin{document}
\begin{tikzpicture}
	\draw[draw=black, semithick, solid] (-4.00,2.00) rectangle (0.50,0.00);
	\node[black, anchor=south west] at (-3.06,2.25) {PullArray a};
	\draw[draw=black, semithick, dashed] (-3.00,1.50) rectangle (-0.50,0.50);
	\node[black, anchor=south west] at (-2.06,0.75) {$f$};
	\node[black, anchor=south west] at (0.96,0.61) {$\Huge\multimap$};
	\draw[draw=black, semithick, solid] (3.00,2.00) rectangle (7.50,0.00);
	\node[black, anchor=south west] at (3.94,2.25) {PullArray b};
	\node[black, anchor=south west] at (4.94,0.75) {$f'$};
	\draw[draw=black, semithick, dashed] (4.00,1.50) rectangle (6.50,0.50);
	\node[black, anchor=south west] at (7.96,0.61) {$\Huge\multimap$};
	\node[black, anchor=south west] at (11.94,0.61) {$\Huge\multimap$};
	\node[black, anchor=south west] at (9.93,0.59) {$\Huge\cdots$};
	\draw[draw=black, semithick, solid] (14.00,2.00) rectangle (18.50,0.00);
	\draw[draw=black, semithick, dashed] (15.00,1.50) rectangle (17.50,0.50);
	\node[black, anchor=south west] at (15.94,0.75) {$f^{(n)}$};
	\node[black, anchor=south west] at (14.94,2.25) {PullArray z};
	\node[black, anchor=south west] at (-2.29,-2.78) {$f$};
	\draw[draw=black, -latex, semithick, dashed] (-2.00,-2.00) -- (-2.00,-0.50);
	\node[black, anchor=south west] at (11.94,-3.25) {$\scriptsize{f^{(n)}(0)}$};
	\node[black, anchor=south west] at (12.94,-3.25) {$\scriptsize{f^{(m)}(1)}$};
	\node[black, anchor=south west] at (13.94,-3.25) {$\scriptsize{f^{(m)}(2)}$};
	\node[black, anchor=south west] at (14.94,-3.25) {$\scriptsize{f^{(m)}(3)}$};
	\node[black, anchor=south west] at (18.44,-3.25) {$\scriptsize{f^{(m)}(n-1)}$};
	\node[black, anchor=south west] at (17.14,-3.21) {$\scriptsize\cdots$};
	\draw[draw=black, semithick, solid]  (12.00,-2.50) grid (20.00,-3.50);
	\draw[draw=black, -latex, semithick, solid] (16.50,-0.50) -- (16.50,-2.00);
\end{tikzpicture}
\end{document}
```

![[vis-pullarray.png]]

El nombre "**Pull Array**" viene del hecho de que resulta muy sencillo leer el arreglo en esta representación, sin la necesidad de evaluar el cómputo de todas las posiciones y alojar la memoria. Esto se logra a partir de la siguiente función:

```haskell
index :: PullArray a ⊸ Int -> (a, PullArray a)
index (PullArray f n) ix = (f ix, PullArray f n)
```

La función no está haciendo la validación de los índices, por lo que puede lanzar un error. Esto se resuelve fácilmente retornando un `Maybe`:

```haskell
safeIndex :: PullArray a ⊸ Int -> (Maybe a, PullArray a)
safeIndex (PullArray f n) ix
  | ix < 0    = (Nothing, PullArray f n)
  | ix >= n   = (Nothing, PullArray f n)
  | otherwise = (Just (f ix), PullArray f n)
```

Nótese que el haber definido `PullArray` como un GADT's con flechas no lineales, nos permite acceder a los componentes del arreglo y usar la función `f` más de una vez. Debido a lo anterior, podría parecer inútil la flecha lineal en la firma de `index`/`safeIndex` con respecto al primer arreglo.

Es cierto que resulta inútil la flecha lineal para garantizar que no podemos copiar el arreglo, pues podemos acceder a los componentes del arreglo y duplicarlos, como en el siguiente ejemplo:

```haskell
copyArray :: PullArray a ⊸ (PullArray a, PullArray a)
copyArray (PullArray f n) = (PullArray f n, PullArray f n)
```

No obstante, esto sólo sucede cuando uno tiene acceso a la función constructora de `PullArray` para poder realizar la coincidencia de patrones. De no ser el caso, entonces resulta imposible hacer copias del arreglo. El siguiente código no va a compilar:

```haskell
copyArray' :: PullArray a ⊸ (PullArray a, PullArray a)
copyArray' arr = (arr, arr)
```

Es por ello que el módulo `Data.Array.Polarized.Pull` no exporta el constructor, para evitar el uso inseguro de tales arreglos. Únicamente dentro del módulo es posible acceder al constructor, en cuyo caso los desarrolladores de la biblioteca deben tener el cuidado de mantener las garantías de linealidad necesarias.

Dicho lo anterior, la función `index` resulta de gran ventaja puesto que permite obtener el valor en alguna posición del arreglo sin la necesidad de computarlos todos de antemano. Una posible desventaja de esta función es que realiza el cómputo al vuelo. Es decir, si se consulta dos veces el valor en una misma posición del arreglo, será necesario evaluar dos veces la función `f idx`, al contrario que en un arreglo alojado, donde bastaría leer el valor en memoria.

## Arreglos Push

Los arreglos Push se introdujeron en [[Claessen_2012]] como una estructura de datos complementaria a los arreglos Pull, para facilitar la composición de cómputos para arreglos. Posteriormente se analizaron sus propiedades a la luz de la lógica lineal y los tipos lineales en [[Bernardy_2016]], y se generalizan a listas inductivas y coinductivas en [[Bernardy_2016a]].

Resultan útiles por su enfoque en el uso de objetos de "Escritura" para la construcción de arreglos. Puede resultar útil para desarrolladores de otros lenguajes como Java y C#, hacer la analogía donde los arreglos push son a los arreglos, lo mismo que los String Builders son a las cadenas.

Los arreglos push se encargan de hacer la composición de objetos de escritura, para posteriormente alojar la memoria necesaria y ejecutar las instrucciones de escritura.

Estos objetos de escritura se conocen en la biblioteca estándar para Haskell con tipos lineales, como `ArrayWriter`:

```haskell
data ArrayWriter a where
  ArrayWriter :: (DArray a ⊸ ()) ⊸ Int -> ArrayWriter a
```

Este tipo de dato consiste en una función como estilo de paso de continuaciones que realiza la escritura de datos en un arreglo destino y la longitud que tendrá el arreglo resultante. Esto se encuentra dado con la notación de GADT's.

Es importante destacar que la firma del tipo de `ArrayWriter` utiliza [[Destination Arrays]]. De aquí vemos que los arreglos push (y por tanto también los arreglos pull) funcionan como una abstracción encima de los arreglos destino, heredando las mismas garantías que ya teníamos.

Por otro lado, nótese la similitud con la firma de la función `alloc` de los arreglos destino: es igual, a excepción del orden de los argumentos y del tipo resultante (`ArrayWriter` en vez de `Array`).

Es posible dar una definición como Semigrupo y Monoide al tipo `ArrayWriter a`, como tenemos a continuación:

```haskell
addWriter :: ArrayWriter a ⊸ ArrayWriter a ⊸ ArrayWriter a
addWriter (ArrayWriter f n) (ArrayWriter g m) = 
  ArrayWriter 
    (\arr -> (DArray.split n arr) 
	           & \(arrLeft, arrRight) -> f arrLeft `seq` g arrRight)
    (n + m)

instance Semigroup (ArrayWriter a) where
  (<>) x y = addWriter x y

instance Monoid (ArrayWriter a) where
  mempty = ArrayWriter (DArray.dropEmpty 0)
```

Vamos a entender mejor la función `addWriter` analizándolo más minuciosamente. Inicialmente tenemos dos términos de tipo `ArrayWriter` con sus respectivas continuaciones y longitudes. Para combinarlos, debemos proveer una función que devuelve un `ArrayWriter` que escriba sobre un arreglo de longitud la suma de las longitudes de los argumentos, e idealmente debe realizar la escritura primero de un `ArrayWriter` y luego del otro.

Por convención supondremos que los `ArrayWriter` escriben de izquierda a derecha siguiendo el orden en que están dados los argumentos.

Al combinar dos continuaciones, supondremos que el arreglo destino al cuál estamos escribiendo tiene la longitud adecuada. Entonces basta dividir el arreglo en dos segmentos de longitud `n` y `m` respectivamente, y realizar la escritura. La composición de las continuaciones sigue retornando `()`, para garantizar que el tipo coincide.

Podemos ver que esta definición es muy similar a la que se dio en [[Destination Arrays]] para poblar un arreglo por mitades, con la diferencia de que aquí se divide el arreglo en un punto arbitrario.

Teniendo una forma de operar objetos de escritura, veamos cómo los podemos usar a través de los arreglos push.

El tipo de dato de `PushArray` es el siguiente:

```haskell
data PushArray a where
  PushArray :: (∀ m. (Monoid m) => (a -> m) -> m) ⊸ PushArray a
```

Es decir, tenemos un tipo de rango 2, pues hay 2 niveles de cuantificación, a diferencia del rango 1 por defecto en Haskell, donde todos los cuantificadores se colocan a la izquierda del tipo (sin necesidad de hacerlo explícito en el código). Por lo tanto, el tipo de `PushArray` es:

```haskell
PushArray :: forall a. 
               (forall m. Monoid m => (a -> m) -> m)
               ⊸ PushArray a
```

Una forma de entender el tipo `(a -> m)` es como una función que toma un elemento y construye el objeto de escritura. Luego entonces, `(a -> m) -> m` es una función que toma otra función para generar objetos de escritura, y obtiene la composición de todos estos (lo cuál es posible al `m` ser un monoide).

Es importante notar que, puesto que se está cuantificando sobre `m`, entonces `PushArray a` es en realidad una familia de tipos, donde podemos estar utilizando distintos monoides, cada uno con su técnica de escritura.

En particular, el único provisto por la biblioteca estándar para la escritura de arreglos es el visto anteriormente, `ArrayWriter`. Pero eso no nos debe detener de usar otras instancias, en caso de desearlo. Veremos ejemplos de esto más adelante.

Tenemos las siguientes funciones constructoras:

```haskell
make :: a -> Int -> PushArray a
singleton :: a -> PushArray a
```

para construir a partir de un elemento un arreglo push, de longitud 1 (`singleton`) o arbitraria (`make`).

Y además contamos con las siguientes dos funciones que agregan elementos a un arreglo push:

```haskell
cons :: a -> PushArray a ⊸ PushArray a
snoc :: a -> PushArray a ⊸ PushArray a
```

donde `cons` agrega el elemento a la izquierda, y `snoc` a la derecha.

Y podemos combinar arreglos de la siguiente manera:

```haskell
append :: PushArray a ⊸ PushArray a ⊸ PushArray a
```

Además, resulta fácil ver que `PushArray a` también es una instancia de la clase de semigrupos usando la función `append`. La instanciación para la clase de monoides queda como ejercicio al lector, y se deriva fácilmente de que el argumento de la continuación es un monoide.

La función que sirve como puente entre `PushArray` y `ArrayWriter` es `alloc`.  Esta función proveé al arreglo de una función constructora (instanciando al monoide en particular que se desea utilizar) y ejecuta la construcción del arreglo.

Para la construcción de arreglos, la definición de `alloc` es:

```haskell
alloc :: PushArray a ⊸ Array a
alloc (PushArray c) = allocArray (c singletonWriter)
  where
    singletonWriter :: a -> ArrayWriter a
    singletonWriter a = ArrayWriter (DArray.replicate a) 1

    allocArray :: ArrayWriter a %1 -> Array a
    allocArray (ArrayWriter writer len) = DArray.alloc len writer
```

Lo importante del código anterior son las dos funciones auxiliares:
- `singletonWriter`: función que devuelve un monoide para construir los elementos base del arreglo.
- `allocArray`: función que convierte el monoide con la construcción del arreglo completo, en un arreglo mutable alojado en la memoria. Se utiliza la función provista por los arreglos destino.

En este caso, estamos diciendo que el arreglo (que será de tamaño `n`) se puede construir como la concatenación de `n` arreglos de tamaño 1. Esto puede asemejarse mucho a otras estructuras de datos como las listas, o cadenas. Eso es puesto que en realidad, todos los monoides comparten la propiedad de que pueden construirse a partir de elementos de tamaño 1 (según la definición de tamaño apropiada para cada tipo) que se combinan para generar algo de mayor tamaño.

No obstante, como dijimos anteriormente, al no estar limitados a utilizar `ArrayWriter`, podemos dar una función `alloc` que haga uso de otros monoides. Como ejemplo, podemos construir listas:

```haskell
allocList :: PushArray a ⊸ [a]
allocList (PushArray c) = c singletonList
  where
    singletonList :: a -> [a]
    singletonList a = [a]
```

O incluso podemos realizar operaciones, con los monoides numéricos:

```haskell
allocSum :: PushArray Int ⊸ Sum Int
allocSum (PushArray c) = c singletonSum
  where
    singletonSum :: Int -> Sum Int
    singletonSum = Sum
```

## Flujos Polarizados

En general no será suficiente utilizar los arreglos push o pull por separado. Usualmente, para algoritmos no triviales, conviene utilizar una combinación de ambos, como se mencionó al comienzo.

Para ello tenemos acceso a las siguientes funciones que convierten entre variantes de arreglos:

```haskell
fromArray :: Array a ⊸ PullArray a
transfer :: PullArray a ⊸ PushArray a
alloc :: PushArray a ⊸ Array a -- Analizada anteriormente
```

Los casos de uso más comúnes son en la transformación de arreglos. Usualmente, si se desea operar con un arreglo (alojado en memoria) es necesario transformarlo a una estructura intermedia más flexible - por ejemplo, una lista - y posteriormente reconstruir el arreglo. Todos los pasos en el ejemplo anterior consumen memoria, y en ocasiones puede ser muy ineficiente, especialmente en lenguajes con recolectores de basura donde los datos pueden permanecer en memoria un tiempo indeterminado.

Una alternativa a ello es el uso de arreglos polarizados. Tomemos como ejemplo la siguiente función para aplicar un filtro sobre un arreglo:

```haskell
arrFilter :: Array a -> (a -> Bool) -> Array a
arrFilter arr f = alloc . transfer $ (loop (fromArray arr) f)
  where
    loop :: PullArray a -> (a -> Bool) -> PullArray a
    loop parr f = case PullArray.findLength parr of
      (0,_) -> PullArray.fromFunction (error "empty") 0
      (n,_) -> case PullArray.split 1 parr of
        (head, tail) -> case PullArray.index head 0 of
          (a,_) ->
            if f a
            then PullArray.append (PullArray.singleton a) (loop tail f)
            else loop tail f
```

Podemos ver que el uso de arreglos nos permite realizar la operación sobre el arreglo, sin embargo no estamos alojando memoria para los resultados intermedios. Solamente hasta concluir el proceso se ejecuta la función `alloc` que aloja la memoria y escribe los resultados.

A continuación otro ejemplo:

```haskell
merge :: Ord a => Array a -> Array a -> Array a 
merge v1 v2 = (alloc . transfer) $ merge' (fromArray v1) (fromArray v2)

merge' :: Ord a => PullArray a ⊸ PullArray a ⊸ PullArray a
merge' arr1 arr2 = PullArray.findLength arr1 &
  \(n, arr1') ->
  if n == 0 
    then PullArray.append arr1' arr2
    else PullArray.findLength arr2 &
      \(m, arr2') ->
      if m == 0
        then PullArray.append arr1' arr2'
        else (PullArray.index arr1' 0, PullArray.index arr2' 0) &
          \((x, arr1''), (y, arr2'')) -> 
          if x <= y 
            then PullArray.split 1 arr1'' & \(h, t) ->
              PullArray.append h (merge' t arr2'')
            else PullArray.split 1 arr2'' & \(h, t) ->
              PullArray.append h (merge' arr1'' t)
```

En el código anterior podemos ver cómo el cómputo se realiza al nivel de arreglos Pull. Posteriormente, se utiliza `transfer` para mover las instrucciones de construcción del arreglo a un arreglo Push, que finalmente puede ser alojado en memoria como un arreglo.

## Una analogía para los arreglos polarizados
Una posible forma de entender a los arreglos polarizados es mediante la siguiente analogía. Vamos a entender a los arreglos como edificios.

Los arreglos Pull podemos verlos como planos de construcción, que pueden ser modificados, alterados e incluso desarrollados desde cero. No obstante, el plano nunca es el edificio final, únicamente es una guía para su construcción.

Por otro lado, los arreglos Push serían similar a un obrero, que toma el plano de construcción y construye el edificio concreto. Si bien el obrero suele seguir las instrucciones del arreglo, también puede realizar alteraciones entre el plano y el resultado, y es indispensable que la visión del obrero se alineé al resultado final deseado.

A continuación presento un ejemplo en donde podemos ver que los arreglos push tienen la capacidad de alterar el resultado final en la construcción del arreglo.

```haskell
addWriter :: RevArrayWriter a ⊸ RevArrayWriter a ⊸ RevArrayWriter a
addWriter (RevArrayWriter f n) (RevArrayWriter g m) = 
  RevArrayWriter 
    (\arr -> (DArray.split m arr) 
	           & \(arrLeft, arrRight) -> g arrLeft `seq` f arrRight)
    (n + m)

instance Semigroup (RevArrayWriter a) where
  (<>) x y = addWriter x y

instance Monoid (RevArrayWriter a) where
  mempty = RevArrayWriter (DArray.dropEmpty 0)

alloc :: PushArray a ⊸ Array a
alloc (PushArray c) = allocArray (c singletonWriter)
  where
    singletonWriter :: a -> RevArrayWriter a
    singletonWriter a = RevArrayWriter (DArray.replicate a) 1

    allocArray :: RevArrayWriter a %1 -> Array a
    allocArray (RevArrayWriter writer len) = DArray.alloc len writer
```

En el ejemplo anterior se da una definición para un nuevo monoide `RevArrayWriter` que invierte el orden en que se construyen los arreglos, generando la reversa del arreglo que se construyó.

Esto también puede realizarse con los arreglos Pull, pero tiene la desventaja de que entonces hay que hacer operaciones adicionales para intercambiar índices. Al realizarlo con arreglos Push, no se añaden instrucciones, únicamente se modifica la sección del arreglo a la que se escriben los resultados.