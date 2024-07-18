Para la implementación de tipos lineales en Haskell, se optó por aplicar la linealidad al nivel de funciones. Como se vió en el [[Cálculo λ𐞥→]], las flechas se decoran con un parámetro que expresa la multiplicidad (o como expresan los desarrolladores de GHC, linealidad) de las funciones con respecto a sus parámetros.

Si bien las reglas están dadas de manera formal por la semántica estática vista anteriormente, la noción intuitiva detrás de las flechas lineales es:

**Definición.** *Noción de la flecha lineal.*
Una flecha lineal $f :: s \multimap t$ garantiza que si $(f \; u)$ es consumido exactamente una vez, entonces el argumento $u$ se consumió exactamente una vez.

Luego entonces, para entender lo que significa "consumir exactamente una vez" los autores proponen la siguiente definición:

**Definición.** *Consumir exactamente una vez (o linealmente).*
1. Para consumir un término de un tipo base atómico una vez, hay que evaluarlo. Esto incluye tipos como Int, Char, Float.
2. Para consumir una función linealmente, aplica la función a un argumento, y consume su resultado exactamente una vez.
3. Para consumir un par (tupla de dos entradas), realiza la coincidencia de patrones y consume cada entrada exactamente una vez.
4. En general, para consumir un valor de un dato algebráico linealmente, realiza la coincidencia de patrones y consume cada entrada lineal exactamente una vez.

Nótese que la regla 3 es una instancia particular de la regla 4.

Con las reglas anteriores queda claro que los datos no son propiamente lineales o no-lineales. Por el contrario, un dato puede ser *consumido* linealmente o no-linealmente. Esto resulta en que un mismo dato puede ser alimentado a una función lineal $f$ y a otra no-lineal $g$. Si bien $f$ no puede garantizar que está consumiendo la única referencia al dato, sí puede garantizar que dentro de $f$ el dato se usa linealmente, mientras que en $g$ no hay garantías de su uso.

Para eludir los problemas que representa la imposibilidad de garantizar la existencia de un único apuntador a un dato, los desarrolladores de Haskell que hacen uso de la extensión `LinearHaskell`, y en particular de la biblioteca `base-linear` hacen uso del estilo de programación [[Destination Passing Style]], garantizando que el dato únicamente existe en un contexto donde su uso es lineal. Ejemplos de esto, más adelante en el caso de uso.

## Datos
Los tipos de datos en Haskell permiten definir un tipo de dato como uno o más constructores con múltiples campos. Por defecto, cuando un dato se utiliza en un contexto lineal, todos los campos de tal dato deberán ser utilizados linealmente. De otra manera, Haskell identificará un uso incorrecto.

Para modificar tal comportamiento, hay una alternativa utilizando la extensión `GADTs`.

Como se vio anteriormente, los tipos de datos algebráicos generalizados permiten expresar a los constructores de los tipos de datos explícitamente como funciones. Por lo tanto, tales funciones también deben tener su anotación de multiplicidad.

Mediante el uso de GADTs, el desarrollador tiene la posibilidad de definir un tipo de dato cuyo constructor sea explícitamente no-lineal en alguno de sus campos. Por lo tanto, al utilizar el dato en un contexto lineal, no se impondrán restricciones sobre tal campo.

```haskell
data WeightedPair a b where
	WPair :: a -> b ⊸ WeightedPair a b

sndWeighted :: WeightedPair a b ⊸ b
sndWeighted (WPair _ b) = b
```

En el ejemplo anterior tenemos un tipo de dato definido con la notación de GADTs, que representa un par no-lineal en su primera entrada, y lineal en la segunda. Debido a la linealidad explícita que tiene, al utilizar un dato de este tipo en una función lineal, únicamente es necesario ser lineal en su segunda entrada.

Por ejemplo, podemos escribir la función `sndWeighted` que no utiliza el primer campo, sin problemas en la compilación. Por otro lado, la siguiente función no pasará la verificación:

```haskell
fstWeighted :: WeightedPair a b ⊸ a
fstWeighted (WPair a _) = a
```

debido a que no se está usando el segundo campo linealmente.

Otros ejemplos de la utilidad de esta extensión en conjunto con los tipos lineales, se verán más adelante.

## Linealidad en los kinds
Una alternativa a la linealidad sobre las flechas que ha sido propuesta por otros autores, es la linealidad sobre los kinds.

La linealidad sobre los kinds permite garantizar que un dato existe de forma única, y no hay copias o múltiples referencias a este. A continuación enuncio algunas ventajas de cada enfoque:

**Ventajas de linealidad sobre kinds:**
- Facilita modelar tipos de datos que deben tener una única referencia, como por ejemplo, descriptores de archivos y arreglos.
- Facilita la implementación de sistemas de propiedad (*ownership*) sobre los datos, y sistemas de tipos únicos. Este es el caso de otros lenguajes como Rust.

Ambas ventajas tienen solución con linealidad sobre las flechas, pero requiere tomar algunos caminos no tan directos. Para el primer punto se ha mencionado el uso de Destination Passing Style, y para el segundo se presentó anteriormente una multiplicidad intermedia entre lineal y no-restringida para definir la función `borrow`.

**Ventajas de linealidad sobre flechas:**
- La interpretación bajo la correspondencia "*tipos como proposiciones*" se pierde, puesto que no existen las "*proposiciones lineales*" que sería lo correspondiente a linealidad sobre kinds.
- Facilita la reutilización del código mediante el polimorfismo de multiplicidad.
- Facilita la interacción con tipos dependientes. Este es un aspecto que se investiga para su implementación a futuro.

Una alternativa que se plantea y ha estado siento utilizada en algunas bibliotecas de Haskell por algún tiempo, es la linealidad mediante mónadas, en especial la mónada `ST`.

Si bien es una alternativa útil, y se han implementado optimizaciones en el compilador para optimizar los fragmentos monádicos como referencias únicas, presenta el problema de necesitar un mayor nivel de abstracción, además de que la linealidad deja de ser una propiedad semántica explícita en el código: se confía en el compilador para detectar la linealidad, no depende del desarrollador.

## Funciones divergentes
Veamos el siguiente código que pasa la verificación del compilador:

```haskell
repeat :: a ⊸ [a]
repeat x = xs
	where xs = x : xs
```

Es una función que diverge repitiendo un mismo elemento. En principio puede parecer que es un error en el compilador que esta función pase la verificación de tipos, puesto que estamos repitiendo un mismo elemento una cantidad infinita de veces, lo cuál definitivamente no es lineal.

Tomemos la siguiente función:

```haskell
ignore :: a ⊸ a ⊸ [a]
ignore x y = repeat x ++ [y]
```

Esta función también aparenta que no debería pasar la verificación de tipos puesto que al ser divergente `repeat x`, nunca se va a consumir el programa `[y]`.

Estos son ejemplos del comportamiento de los tipos lineales con funciones divergentes.

Recordemos que la noción para identificar si algo está siendo consumido de forma lineal es: *si $(f\; u)$ se consumió exactamente una vez, el argumento $u$ se consumió exactamente una vez.*

Las funciones `repeat` e `ignore` son ejemplos de funciones que no cumplen el antecedente de la condición de linealidad. Puesto que son funciones divergentes, la aplicación de las funciones `repeat x` e `ignore x y` no puede ser consumido en su totalidad exactamente una vez. Por lo tanto, no se cuenta con garantías para tales situaciones: puede o no pasar la verificación de tipos.

Las reglas para determinar si algo pasa la verificación de tipos sigue siendo dirigida por las reglas sintácticas del cálculo $\lambda_{\rightarrow}^q$, más no corresponden necesariamente a alguna noción intuitiva útil.

Por ejemplo, la siguiente modificación de `ignore` es observacionalmente equivalente a la versión anterior, sin embargo, no pasa la verificación de tipos:

```haskell
ignore :: a ⊸ a ⊸ [a]
ignore x _ = repeat x
```

Debido a que las funciones divergentes nunca consumen su resultado completamente, resulta sin utilidad que reciban argumentos de forma lineal.