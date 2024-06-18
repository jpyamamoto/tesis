Visto como tipo de dato abstracto, se puede definir mediante la siguiente interfaz.

**Definición:** ([[Black_2016]])
```haskell
new :: Int -> Array a n
get :: Array a n -> Idx -> a
set :: Array a n -> Idx -> a -> Array a n
```

donde `Idx` es un tipo para representar los índices con los que se accede al arreglo. Su semántica es:

```haskell
new(s) devuelve un arreglo de tamaño s
get(set(A, i, v), i) = v
get(set(A, j, v), i) = get(A, i) cuando i != j
```

La definición de los arreglos suele ser poco satisfactoria puesto que no presenta muchas de las garantías que se esperan usualmente de los arreglos. Mediante el uso de tipos dependientes se puede dar una definición recursiva, a modo de *lista de tamaño fijo*. Sin embargo, no presento tal definición pues también carece de expresividad en cuanto a las garantías que se esperan de un arreglo.

Desde un punto de vista más general,  un arreglo se entiende como segmentos continuos y finitos de memoria, cuyos elementos se acceden a través de un polinomio de redireccionamiento para conocer la dirección de memoria esperada. ([[Pelaez_2018]]) Esto implica que la lectura y escritura de elementos se realiza en tiempo $O(1)$. Esta es su principal garantía de interés.

Los arreglos no son redimensionables, aunque pueden definirse interfaces que envuelven a los arreglos para permitir su redimensión. Esto se suele conocer como un *Vector*.

## Arreglos Mutables y Transparencia Referencial

El concepto de acceso directo a la memoria es ajeno al paradigma de programación declarativo.

Esto es debido a que existe un problema intrínseco en el uso de arreglos en un lenguaje puramente funcional, pues aparentemente se reduce a uno de los siguientes dos casos:
- Se mantiene la transparencia referencial, siendo imposible realizar modificaciones *en sitio*, ó
- Se permite la escritura en memoria para preservar las operaciones de arreglos, pero se pierde la propiedad de transparencia referencial en el subconjunto de programas que utilizan esta estructura de datos.

La solución que se dio a tal situación es el uso de mónadas para secuencializar las operaciones. Esto permite que la transparencia referencial se mantenga bajo una interfaz aparentemente "imperativa", permitiendo el acceso a memoria.

GHC brinda tal interfaz para realizar operaciones "imperativas" a través del tipo `Array#`. Se pueden utilizar la mónada `ST` o la mónada `IO` para secuencializar las operaciones (siendo `ST` la más popular).

No revisaré los detalles de implementación puesto que escapa al tema pertinente. No obstante, cabe destacar que las operaciones para escritura asociadas a `Array#` son "inseguras" (de ahí que los nombres de tales funciones se nombran con el prefijo `unsafe`).

El motivo de que las operaciones de escritura para arreglos (en general, no solamente para `Array#`) son inseguras, es puesto que el resultado va a depender de las optimizaciones que realiza el compilador y la forma de evaluación que decide utilizar.

```haskell
import Control.Monad.ST
import Data.Array.IArray
import Data.Array.ST
import GHC.Arr (unsafeFreezeSTArray)

testFreeze :: (Int, Int)
testFreeze = runST $ do
  marr <- newArray (0, 10) 0 :: ST s (STArray s Int Int)
  arr <- unsafeFreezeSTArray marr
  let x = arr ! 0
  writeArray marr 0 99
  let y = arr ! 0
  return (x, y)
```

Si se ejecuta el código anterior, obtenemos el resultado `(99,99)`. Es decir, incluso después de haberse ejecutado el *freeze* del arreglo (después del cuál se debería tener una copia inmutable del arreglo original), sigue siendo posible modificar los valores en cualquier posición del arreglo, y esto afecta a los accesos que aparentemente suceden previos a la modificación, como es el caso de `x` tomando el valor actualizado.

Esto es debido a que la función que realiza el *freeze* únicamente retorna el apuntador al arreglo original. De ahí se rescata que el `unsafe` en la operación de "congelamiento" no es más que un indicador para que el programador sea consciente de evitar realizar modificaciones a tal arreglo. Sin embargo, no hay ninguna restricción para ello a nivel del compilador.

Podemos ver que si forzamos al compilador a utilizar una evaluación distinta, cambia el resultado. Por ejemplo, al compilar con la extensión `BangPatterns` para forzar la evaluación estricta en algún punto del programa, podemos indicar a Haskell que evalúe `x` lo más pronto posible, y luego el compilador va a deducir que (por la transparencia referencial) `y` debe tener el mismo valor.

```haskell
{-# LANGUAGE BangPatterns #-}

testFreeze' =
  ...
  let !x = arr ! 0
  ...
```

Al hacer la modificación anterior, el resultado evalúa a `(0,0)`. Por otro lado, si además evitamos las optimizaciones por transparencia referencial, el resultado será `(0,99)`. Para ello, basta compilar el código anterior con la bandera `-O0` de GHC.

En todo compilador es importante que las optimizaciones no alteren el comportamiento del programa. Sin embargo, esta propiedad de seguridad se puede perder al momento de utilizar funciones marcadas como `unsafe`.

De aquí surge la pregunta, ¿existe una forma de tener arreglos seguros en un lenguaje con transparencia referencial?