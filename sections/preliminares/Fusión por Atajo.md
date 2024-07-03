La Fusión por Atajo es una técnica para optimizar expresiones en lenguajes puramente funcionales.

Funciona al encapsular una serie de operaciones aplicadas secuencialmente, en una sola función que aplica todas las transformaciones en un sólo paso. Ejemplos de esta optimización aplicada a funciones sobre listas se pueden ver en la siguiente figura:

$$ \text{map } f \;\circ\;\text{map } g \;\equiv\; \text{map } (f \circ g)  $$
$$ \text{filter } f \;\circ\;\text{filter } g \;\equiv\; \text{filter } (\lambda x \to f\; x \;\land\; g\; x ) $$
$$ \text{sum } [1:n] \;\equiv\; \text{loop }\{acc := 0, i \leftarrow 1\dots n, acc := acc + i \} $$

Esta técnica ha evolucionado en distintos marcos de trabajo para la identificación y aplicación de tales optimizaciones, incluyendo casos particulares para la deforestación de árboles ([[EnriquezMendoza_2020]]), funciones de orden superior, entre otros.

En particular resulta de interés para este proyecto el uso de Fusión por Atajo para operaciones sobre vectores.

Actualmente, el algoritmo favorecido por la comunidad a través de la biblioteca `vector` se conoce como **Fusión Generalizada de Flujos** (*Generalized Stream Fusion*). [[Mainland_2013]]

Este consiste en ver a los arreglos como *streams*, que representan una secuencia de operaciones a realizar. Todas las operaciones sobre arreglos se operan al nivel de *streams*, permitiendo aplicar optimizaciones de fusión por atajo.

Las funciones de `Stream` en `Stream` suelen ser candidatos a optimización mediante fusión.

```haskell
data Stream a where
	Stream :: (s -> Step s a) -> s -> Int -> Stream a

data Step s a = Yield a s
			  | Skip s
			  | Done
```

Además contamos con funciones para transformar vectores en *streams* y viceversa.

```haskell
stream :: Vector a -> Stream a
unstream :: Stream a -> Vector a
```

donde la equivalencia importante es $\text{stream }\circ\text{ unstream}\equiv\text{unstream }\circ\text{ stream}\equiv\text{id}$.

Podemos ver como ejemplo la función `map :: (a -> b) -> Vector a -> Vector b` siendo reescrita como:

$$ \text{map }f = \text{unstream} \circ \text{map}_s\; f \circ \text{stream} $$
donde $\text{map}_s f$ es una función con firma `Stream a -> Stream b` (aquí ya estamos operando en el nivel de *streams*). Luego entonces

$$ \begin{align*} \text{map }f\;\circ\;\text{map }g &= \text{unstream} \circ \text{map}_s\; f \circ \text{stream} \circ \text{unstream} \circ \text{map}_s\; g \circ \text{stream} \\ &= \text{unstream} \circ \text{map}_s\; f \circ  \text{map}_s\; g \circ \text{stream} \end{align*} $$
Y como $\text{map}_s$ opera al nivel de *streams*, las instrucciones están sujetas a la reescritura mediante fusión por atajo, quedando algo del estilo:

$$ \text{unstream }\circ \text{map}_s\; (f \circ g)\;\circ\text{ stream} $$
Este mismo tipo de optimización se puede aplicar a las distintas funciones para operar vectores, únicamente dando su versión análoga como *stream*.

Las principales ventajas de esta optimización son:
- Las transformaciones se pueden aplicar en tiempo de compilación.
- Encapsulando las operaciones en la mónada `ST`, el compilador puede optimizar las operaciones para realizarse *en sitio*.

Sus desventajas son:
- Requiere definir una por una la versión en *streams* de cada función.
- El compilador no siempre identifica correctamente los candidatos a aplicar fusión para casos no triviales.
- Requiere el uso de mónadas (usualmente es la mónada `ST`).