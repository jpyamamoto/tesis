La Lógica Lineal fue introducida por Jean-Yves Girard en [[Girard_1987]], originalmente como una manera de atacar problemas referentes al cómputo paralelo, bajo la noción de *"Proposiciones como Tipos"*.

Posteriormente múltiples autores continuaron el estudio de este sistema y ha sido aplicado a distintas áreas como el cómputo concurrente, cómputo cuántico, lingüística, entre otros.

A continuación reviso las partes más esenciales de la lógica lineal que serán de utilidad para entender la motivación detrás de los tipos lineales, y algunas decisiones en su implementación. Esto será con base en la presentación de Wadler en [[Wadler_1993]], que presenta la Lógica Lineal en una versión con notación de secuentes y supuestos lineales e intuicionistas.

Para conocer la presentación usual de la lógica lineal desde el punto de vista de Teoría de la Prueba, se recomienda revisar [[Troelstra_2000]]. Para profundizar en algunos temas relacionados a la aplicación de la Lógica Lineal en Lenguajes de Programación no cubiertos en este trabajo, se recomienda revisar [[Pfenning_2002]] que toca temas como Programación Lógica y Tipos Lineales en el contexto de Tipos Dependientes.
## Sintaxis
La gramática para el lenguaje de las fórmulas de la Lógica lineal es la siguiente:
$$ A,B,C := X | A \multimap B| A \otimes B | A \& B | A \oplus B | ! A $$
donde $X$ son constantes lógicas, y las demás son variables proposicionales.

Los secuentes son de la forma $\Gamma \vdash A$, donde $\Gamma$ es una colección de supuestos que pueden estar en dos formas:
- Supuestos lineales: Cuya notación es $\langle A\rangle$ para decir que $A$ debe utilizarse exactamente una vez (linealmente).
- Supuestos intuicionistas: Cuya notación es $[A]$ para decir que $A$ puede utilizarse múltiples veces.

## Observaciones
Para quienes tengan conocimientos previos de Lógica Lineal, se puede ver que la presentación de la sintaxis difiere de la usual, en que no contamos con el operador ⅋ (par), con el operador $\cdot^{\bot}$ (dual) y con el exponencial $?$.

Esto es debido a que estamos utilizando una versión que refina la lógica intuicionista, a diferencia de la presentación usual que refina la lógica clásica.

Por un lado perdemos la evidente dualidad que tenemos en la presentación usual de la lógica lineal, que incluso permite reducir el número de operadores mediante un sistema unilateral con el uso de $\cdot^{\bot}$. Por otro lado, obtenemos una versión con menos problemas técnicos al momento de incrustar la lógica intuicionista en él.

## Interpretación de Recursos
Antes de revisar las reglas que rigen al sistema, es conveniente conocer una de las posibles interpretaciones para el sistema que facilitarán su comprensión.

Esta es la interpretación conocida como "Interpretación de Recursos". Así como la lógica intuicionista captura la noción de "demostrabilidad" de una proposición, y la lógica clásica captura la noción de "verdad", la lógica intuicionista captura el uso de los recursos. Es decir, los teoremas no son absolutos, sino dependientes en la disponibilidad de recursos para construirlas.

Esta interpretación resulta coherente con el cómputo, puesto que no tenemos una cantidad infinita de recursos, ni podemos (en general) utilizar un mismo recurso de forma no-restringida.

Podemos comenzar por entender los operadores. El uso del operador $\otimes$ en la fórmula $A \otimes B$ nos indica que contamos con dos recursos: $A$ y $B$; y debemos utilizarlos a ambos linealmente. Es notorio que su interpretación es muy similar a la de $\land$ en lógica intuicionista.

El operador $\oplus$ se comporta de manera similar al operador intuicionista $\lor$. La fórmula $A \oplus B$ quiere decir que contamos con alguno de los dos recursos: $A$ ó $B$; más no sabemos decidimos con cuál.

El operador $\&$ puede parecer un punto medio entre los dos anteriores. La fórmula $A\& B$ quiere decir que se cuenta con ambos recursos, pero únicamente podemos consumir uno de nuestra elección.

Para entender mejor los operadores anteriores podemos usar la siguiente ilustración basada en la descripción de [[Buss_2000]]:

Supongamos que tenemos al general de un armamento que puede recibir un ataque por dos frentes: este u oeste. El operador $\otimes$ nos indicaría que el general sabe que va a recibir un ataque en ambos frentes al mismo tiempo, por lo que debe desplegar suficientes soldados a ambas posiciones.
El operador $\oplus$ le dice al general que va a recibir un ataque en sólo uno de los dos frentes, pero no sabe en cuál, por lo tanto, también debe desplegar suficientes soldados a ambos frentes, para estar bien protegido.
El operador $\&$ indica al general que va a recibir un ataque en uno de los dos frentes, y además, le avisan de antemano en cuál será. Por lo tanto, basta con desplegar soldados al frente que será atacado.

Regresando a la descripción del sistema, nuestro único operador unitario es $!$. Lo que este operador nos dice es que, dado $!A$, podemos utilizar $A$ tantas veces como deseemos. Es decir, este operador nos permite escapar la linealidad. Abusando un poco del ejemplo anterior, es como tener un soldado al que nunca se le acaban las balas.

Podemos notar que no tenemos un operador de implicación $\rightarrow$ como usualmente, pero contamos con el operador $\multimap$. Esta es la implicación lineal: $A\multimap B$ quiere decir que al consumir el recurso $A$, obtenemos el recurso $B$.

Notemos que a diferencia de la lógica intuicionista, $A$ y $A \rightarrow B$ no nos permite concluir $A \land B$, sino únicamente $B$ (pues consumimos $A$ y deja de estar disponible).

Entendiendo lo anterior, resulta evidente que pmás en profundidadara obtener el comportamiento intuicionista, bastaría tener $A$ no sujeta a la restricción de linealidad. Es decir, $$ A \rightarrow B \equiv (!A) \multimap B $$
