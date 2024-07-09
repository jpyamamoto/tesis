Primero introducimos las definiciones de algunas operaciones que se utilizarán más adelante.

**Contextos:** Los contextos para las reglas de tipificado se definen como
$$ \Gamma, \Delta := (x :_{\mu} A), \Gamma | \varnothing $$

Nótese cómo cada término ligado a su tipo, hace explícita su multiplicidad.

**Suma de contextos:**
![[context-addition.png]]

La idea detrás de la suma de contextos corresponde a la noción de agregar el número de usos de una misma variable en dos distintos contextos, como el uso de una misma variable un mayor número de veces en un sólo contexto unificado.

**Escalamiento de contextos:**
![[context-scaling.png]]

Por otro lado, el escalamiento de contextos es útil para determinar cuántas veces se necesitan utilizar las premisas que construyen una variable, cuando tal variable es consumida múltiples veces.

**Equivalencia de multiplicidades:** La relación de equivalencia entre multiplicidades con sus operaciones de suma y producto se define como la relación transitiva y reflexiva más pequeña tal que se cumplen las siguientes reglas:

![[mutiplicity-equivalence.png]]

Es posible ver que las multiplicidades forman un semi-anillo, extensible a un módulo sobre los contextos de tipificado.

Si bien hasta ahora he estado apelando a la noción de "contar el número de veces que es usada una variable", la verdad es que ese "número de usos" va a colapsarse en 2 posibilidades: un único uso $1$ o un número indeterminado de usos $\omega$.

Notemos que la única forma de mantener linealidad es bajo la operación $(\pi \cdot 1)$, pues cualquier otro caso manda las multiplicidades a $\omega$.

A continuación se presentan las reglas para el tipificado de términos en $\lambda_{\rightarrow}^q$.

![[typing-rules.png]]

> [!warning]
> Typo: En la regla *abs*, la $q$ debe ser $\pi$.  
> La notación $\mu_i [\pi_1 ... \pi_n]$ es sustitución de $[\pi_1 ... \pi_n]$ en $\mu$.

La regla _var_ presenta una situación interesante pues el contexto tiene multiplicidad $\omega$ a excepción del término que se obtiene del contexto. Esto es para garantizar la linealidad. Si la regla fuera la siguiente:
$$\overline{\Gamma + x:_1 A \vdash x : A}$$
entonces en $\Gamma$ podríamos verse como $\Gamma = \Gamma' + y :_1 B$, dándose el caso en que la variable $y$ que debería usarse de forma lineal, no está siendo utilizada.

La regla de abstracción indica la multiplicidad de su argumento como subíndice de la lambda, y nótese que en la premisa de la regla, la multiplicidad queda expresada en el contexto.

Para la regla de aplicación, es importante notar que el número de veces que es utilizado el argumento queda expresado con $\pi \Delta$, siendo $\pi$ el número de veces que se utiliza el argumento, y $\Delta$ el contexto necesario para construirlo.

Las otras reglas interesantes son aquellas referentes a datos algebráicos.

En este sistema, los tipos de datos hacen explícita la relación de multiplicidad en cada una de sus entradas. Además, cada entrada del tipo de dato puede tener una multiplicidad distinta, por lo que un mismo dato puede tener ciertas restricciones en el número de veces que se ocupa uno de sus componentes, distinto a otros.

Para el destructor `case` tenemos que verificar que se cumplen las condiciones de multiplicidad en cada uno de los componentes del dato, acorde a su definición. Algo a resaltar es el hecho de que un término siendo destruido mediante `case` se considera está siendo utilizado linealmente, sí y sólo si todos sus campos lineales se utilizan de forma lineal; no existen restricciones en cuanto a la forma en que se consumen los campos no-lineales. Otra forma de verlo sería: si `case` consume el campo lineal de un término en forma no-lineal, entonces el término se está consumiendo no linealmente.

La intuición para la regla `let` se puede obtener recordando que usualmente `let` es azúcar sintáctica para una abstracción seguida de una aplicación.

Finalmente, tenemos dos reglas interesantes que permiten implementar el polimorfismo de multiplicidad.

Muy al estilo del polimorfismo de tipificado en el sistema F, el polimorfismo de multiplicidad utiliza una $\lambda$ abstracción al nivel de los valores de multiplicidad, cuantificando sobre el dominio $\{1, \omega\}$. Con la regla de aplicación se debe aplicar una operación de sustitución recursiva sobre las etiquetas de multiplicidad:

>[!info]
>Escribir en latex definición recursiva sobre los términos del lenguaje, para sustituir multiplicidades.

Las condiciones para consumir linealmente términos dadas anteriormente por las reglas de la semántica estática, se pueden entender intuitivamente como a continuación (esta descripción resulta de utilidad al programar en Haskell):

Definición. Consumir exactamente una vez (o linealmente).
1. Para consumir un término de un tipo base atómico una vez, hay que evaluarlo. Esto incluye tipos como Int, Char, Float.
2. Para consumir una función linealmente, aplica la función a un argumento, y consume su resultado exactamente una vez.
3. Para consumir un par (tupla de dos entradas), realiza la coincidencia de patrones y consume cada entrada exactamente una vez.
4. En general, para consumir un valor de un dato algebráico linealmente, realiza la coincidencia de patrones y consume cada entrada lineal exactamente una vez.

Nótese que la regla 3 es una instancia particular de la regla 4.