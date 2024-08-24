La programación funcional es un paradigma que tiene sus raíces en el cálculo lambda. Consiste en el uso de funciones como primitivas, permitiendo la composición y aplicación de estas.

Este modelo, a pesar de ser relativamente simple, permite la representación y estudio del cómputo.

En particular un lenguaje de programación funcional que resulta de interés es Haskell. Esto es debido a su sistema de tipos que ahora incluye tipos lineales, y se analizará a lo largo de este trabajo.

Para comprender este trabajo se asumirá algo de familiaridad con lenguajes de programación funcionales. Para más información, es recomendable remitirse a otros recursos.

> [!TODO]
> Citar recursos

A continuación menciono algunas propiedades que conveniene tener presentes para comprender el resto del texto. No son propiedades exclusivas de lenguajes funcionales, ni tampoco son propiedades derivadas del hecho de un lenguaje ser funcional; pero sí son conceptos con mayor presencia en tales lenguajes, particularmente en Haskell.

## Transparencia Referencial
La transparencia referencial es un concepto que en computación utilizamos para describir lenguajes de programación, pero que fue tomada originalmente de la filosofía del lenguaje.

Este concepto es introducido originalmente en [[Quine_2013]] como:

> I call a mode of containment Φ referentially transparent if, whenever an occurrence of a singular term t is purely referential in a term or sentence ψ(t), it is purely referential also in the containing term or sentence Φ(ψ(t)).

Esto es explicado de forma más clara en [[Strachey_2000]]. La idea es que, dado un programa de la forma $(f(g(t)))$, decimos que $f$ tiene la propiedad de transparencia referencial, si basta conocer el valor $y = g(t)$ para poder realizar la aplicación $f(y)$. Es decir, no importa la estructura subyacente de $g$, sólo su evaluación.

Una forma simple de identificar la transparencia referencial en una función, o en general, en las expresiones de un lenguaje de programación, es garantizando que una función siempre evalúa al mismo resultado relativo a los argumentos que recibe la función.

Si bien, para cualquiera con un trasfondo matemático es inconcebible un contexto en el que no se cumpla lo anterior, en los lenguajes de programación es bastante popular encontrar lenguajes en donde una misma rutina puede retornar valores distintos en cada evaluación. 

Esto es algo que los lenguajes funcionales tienden a evitar, permitiendo un razonamiento ecuacional que facilita el análisis de los programas. En particular, Haskell cumple con esto.

Esta es la principal característica por la cuál un lenguaje funcional se hace llamar **"puro"**.

## Tipificado Estático Fuerte

## Seguridad

La seguridad en un lenguaje de programación, se suele entender (como vemos en [[Pierce_2002]]) mediante la frase "Seguridad = Progreso + Preservación".

Esta idea fue presentada originalmente en [[Milner_1978]], como parte de su trabajo en lenguajes de programación.

Los conceptos de Preservación y Progreso se suelen enunciar a manera de teoremas que describen propiedades del lenguaje en cuestión. Veámos en qué consisten estas ideas, utilizando la formulación de [[Harper_2016]].

### Preservación de Tipos
*Preservación de Tipos* es como suele conocerse al teorema que enuncia la propiedad que garantiza que el tipo de una expresión es invariante bajo evaluación.

De manera formal se enuncia:

>[!Info]
> Dado un término del lenguaje bien tipificado $t$ con tipo $T$, si $t \rightarrow t'$, entonces $t' : T$.

Esta es una propiedad que no necesariamente encontramos en otros lenguajes, particularmente del paradigma imperativo (con sus excepciones), donde el sistema de tipos no previene la asignación arbitraria de tipos a una expresión dada. ^[Siendo estrictos, Haskell no cumple con la preservación de tipos debido a la existencia del tipo $\bot$ y a algunos detalles técnicos en la implementación de Excepciones, pero en general los usuarios procuran mantenerse en el fragmento seguro de Haskell.]

### Progreso
En un lenguaje de programación, se define una subconjunto de todas las expresiones del lenguaje, llamados valores simples o primitivos. Estos tienen la particularidad de estar en una forma normal (con una definición propia de cada lenguaje), y que además, no pueden ser evaluados o reducidos. Un ejemplo de esto en la programación funcional son las funciones, que no pueden ser reducidas por sí mismas, sino sólo como parte de una expresión más grande (una aplicación).

El teorema de *Progreso* dice que cuando un término tiene un tipo bien formado, entonces este siempre podrá reducirse bajo evaluación, o en su defecto, corresponde a un valor simple.

De manera formal se enuncia:

>[!Info]
>Dado un término bien tipificado $t$ con tipo $T$, entonces $t$ es un valor en forma normal, o existe un $t'$ tal que $t \rightarrow t'$.

Es importante notar que el teorema de Progreso en general no garantiza que para todo término siempre existe una forma normal que se puede alcanzar en una cantidad finita de pasos. Esto implicaría que el lenguaje no es Turing-completo, y por tanto se suele evitar imponer tal condición (aunque algunos lenguajes como Gallina y Agda sí lo requieren).

Una consecuencia de ello es que podemos definir funciones divergentes en lenguajes como Haskell, a pesar de cumplir el teorema de *Progreso*. ^[Igual que con la Preservación de Tipos, Haskell estrictamente no cumple con el teorema de Progreso, pero es posible mantenerse en un fragmento seguro.]
