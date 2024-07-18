Comencemos por la definición de un término que he estado utilizando constantemente.

**Definición.** *Uso restringido.* Entendemos por el uso restringido de un argumento como el caso en que un argumento debe ser utilizado linealmente. El caso en que no se tiene tal condición, se puede usar no-linealmente, es decir, tiene *uso no restringido*.

Recordemos que en lógica lineal tenemos la siguiente equivalencia:
$$ !A \multimap B \equiv A \rightarrow B $$
En Haskell vamos a poder simular el uso del exponencial $!$ mediante el tipo de dato `Ur` (por *Unrestricted*). De manera que se cumple la siguiente equivalencia:

```haskell
(Ur a) ⊸ b ≡ a -> b
```

Para entender el tipo de dato y constructor `Ur`, veamos su definición:

```haskell
data Ur a where
	Ur :: a -> Ur a
```

Resalta inmediatamente que estamos utilizando la sintaxis de GADT's, pues es necesario para indicar que el constructor consume su argumento de forma no-lineal.

Esta pequeña definición nos da las siguientes garantías:
1. No podemos envolver un argumento marcado como lineal, con el constructor `Ur`. Esto es porque como ya vimos, el constructor tiene una flecha no-lineal. La excepción a esto será la clase `Movable` que reviso más adelante.
2. Podemos utilizar el contenido del dato de forma no-lineal, sin importar la multiplicidad del dato `Ur`. Esto es porque podemos usar emparejamiento de patrones para obtener el valor envuelto en `Ur`, y al este haber sido marcado como no-lineal en el constructor, se puede utilizar sin restricciones.

No debemos confundir consumir el dato `Ur a` con consumir el contenido de `Ur`. La diferencia es evidente con el siguiente ejemplo:

```haskell
badFunction :: (Ur a) ⊸ (Ur a, Ur a)
badFunction x = (x, x)

goodFunction :: (Ur a) ⊸ (a, a)
goodFunction (Ur a) = (a, a)
```

Dado un argumento lineal de tipo `Ur a`, podemos dar un uso sin restricción al valor envuelto por `Ur`, más no podemos usar el argumento dato de tipo `Ur a` no-linealmente.

El tipo `Ur a` cuenta con algunas propiedades interesantes, como el hecho de ser instancia de las clases `Functor`, `Applicative`, `Monad`:

```haskell
instance Functor Ur where
	fmap f (Ur a) = Ur (f a)

instance Applicative Ur where
	pure = Ur
	(Ur f) <*> (Ur x) = Ur (f x)

instance Monad Ur where
	(Ur x) >>= f = f x
```

Además, podemos dar una función para transformar un dato de tipo `Ur a` en un dato de tipo `a`:

```haskell
unur :: Ur a ⊸ a
unur (Ur x) = x
```

Este tipo de dato nos va a permitir dar un uso no-restringido a datos en contextos lineales. Para ello, tenemos las siguientes clases.

## Consumable
La clase `Consumable` nos permite retomar la regla estructural de Contracción que elimina la lógica lineal, y la retoma mediante el uso de exponenciales.

Su definición es:

```haskell
class Consumable a where
	consume :: a ⊸ ()
```

Es decir, los datos con tipos instancia de `Consumable` pueden ser no utilizados, incluso en contextos lineales. O en otras palabras, pueden ser consumidos por una función `consume` que va a descartarlos.

Análogo a los exponenciales en la lógica lineal, `Ur` en particular es un tipo que puede ser consumido. No obstante, la definición en Haskell es un tanto más débil puesto que existen más tipos que son instancia de la clase.

En particular, todos los tipos primitivos son instancia de `Consumable`.

## Dupable
La clase `Dupable` rescata una versión de la regla estructural de Debilitamiento restringido únicamente a duplicar datos ya existentes (a diferencia del Debilitamiento usual que permitiría asumir datos sin su previa construcción).

La definición de la clase es:

```haskell
class Consumable a => Dupable a where
	{-# MINIMAL dup2 | dupR #-}
	dup2 :: a ⊸ (a, a)
	dupR :: a ⊸ Replicator a
```

Como podemos ver, lo que la clase nos da es una manera de consumir un argumento en un contexto lineal, y duplicarlo (con la función `dup2`). Por supuesto, si podemos duplicar un dato de forma lineal, entonces podemos replicarlo cuantas veces lo deseemos (mediante la función `dupR`).

En primera instancia puede no ser evidente por qué es necesario que las instancias de `Dupable` sean instancias de `Consumable`. Como hemos visto, existen lógicas subestructurales que tienen sólo una de las dos reglas que estamos rescatando.

Sin embargo, en este caso se debe a una conveniencia en la forma de replicar los datos.

Al momento de generar un dato de tipo `Replicator a` (que es necesario para la segunda función en la definición de la clase), lo que hacemos es crear un tipo de dato coinductivo que podemos consumir hasta donde resulta de utilidad para el programa. Más allá de ese punto, el resto de la estructura debe ser consumida (pues los programas con tipos lineales requieren consumir el dato completo) y entonces la única solución es utilizar la función `consume` de la clase `Consumable`.

Se espera que las instancias de `Dupable` cumplan las siguientes leyes básicas:

*Neutro:* `first consume (dup2 x) ≃ x ≃ second consume (dup2 x)`
*Asociatividad:* `first (dup2 (dup2 x)) ≃ second (dup2 (dup2 x))`

## Movable
Se puede decir que la clase `Movable` rescata la regla de *Introducción del exponencial* $$ \dfrac{\vdash ?\Gamma, A}{\vdash ?\Gamma, !A} $$, salvo pequeños detalles técnicos; en particular que no tenemos un análogo al exponencial $?$. Una versión de la regla más cercana a lo que tenemos en la extensión de Haskell sería la regla $$ \dfrac{\vdash A}{\vdash !A} $$

La definición de la clase `Movable` es la siguiente:

```haskell
class Dupable a => Movable a where
	move :: a ⊸ Ur a
```

Esta clase brinda una forma de escapar la linealidad, y utilizar un dato de forma no restringida. Requerimos la clase `Dupable`, pues sólo un elemento que puede ser copiado debería poder ser utilizado sin restricciones.

La función `move` consume un elemento de forma lineal para construir la forma no-restringida de tal dato.

Se espera que las instancias de `Movable` cumplan con las siguientes leyes básicas:

*Inverso:* `unur (move x) = x`
*Coherencia:* `(move :\: Ur a ⊸ Ur (Ur a)) $ (move :\: a ⊸ Ur a) x = fmap (move :\: a ⊸ Ur a) $ (move :\: a ⊸ Ur a) x`

## Uso
Puede parecer que las 3 clases anteriores nos hacen perder todas las propiedades interesantes que obtenemos de los tipos lineales, más no es así. En la práctica resulta de gran utilidad tener un mecanismo para escapar la linealidad, de la misma forma que los exponenciales resultan de utilidad en la lógica.

Las clases anteriores se implementan por defecto únicamente para tipos primitivos, y otros pocos de los cuáles se esperaría que sean instancias de las clases, como lo es `Ur`.

Sin embargo, para los tipos de los cuáles no se espera ese comportamiento, por ejemplo `Array` o `IO`, no se dan tales instancias. Más allá, las clases han necesario el uso explícito de las funciones `consume`, `dup2` o `move` para los casos donde se desea usar un dato de forma no restringida. La biblioteca estándar no las utiliza, más que en los lugares donde se espera explícitamente ese comportamiento.

Las clases anteriores no suelen definirse explícitamente por el desarrollador (aunque es posible), a excepción de `Consumable`.

Otra ventaja de la cadena de dependencias `Consumable a => Dupable a => Movable a` es la posibilidad de definir instancias genéricas a partir de la definición de `Consumable`, para que Haskell genere `Dupable` y `Movable`.

Para un tipo `ExampleType` que se desea sea instancia de estas clases, se daría explícitamente el código de `Consumable` y se instruye al compilador a generar las demás instancias.

```haskell
instance Consumable ExampleType where
	consume = ...

deriving via Generically ExampleType
  instance Movable ExampleType

deriving via Generically ExampleType
  instance Movable ExampleType
```

Usualmente la instanciación de un tipo como `Consumable` requerirá el uso de código inseguro, pues es la única forma de escapar el uso restringido del dato. De ahí que el programador debe ser explícito en su intención de definir la instancia.