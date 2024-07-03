Multiplicidades: $\pi, \mu ::= 1 | \omega | p | \pi + \mu | \pi \cdot \mu$
Tipos: $A, B ::= A \rightarrow_{\pi} B | \forall p . A | D p_1 ... p_n$
Declaraciones de datos: $\texttt{data } D p_1 ... p_n \texttt{ where } (c_k  : A_1 \rightarrow_{\pi_1} ... {A_n}_k \rightarrow_{{\pi_n}_k} D)_{k=1}^m$
Términos: $e,s,t,t ::= x | \lambda_{\pi} (x:A).t | t s | \lambda p . t | t \pi | c t_1 ... t_n | \texttt{case}_{\pi} t \texttt{ of } \{c_k x_1 ... {x_n}_k \rightarrow u_k \}_{k=1}^m | \texttt{let}_{\pi} x_1 : A_1 = t_1 ... x_n : A_n = t_n \texttt{ in } u$
Vamos a analizar cada parte de la sintaxis.

En la multiplicidades, por ahora únicamente contamos con 2 multiplicidades concretas: $1$ y $\omega$, donde la primera representa la multiplicidad lineal (un sólo uso) y la segunda es la multiplicidad usual (sin restricciones en el uso). La multiplicidad $p$ es una variable, para permitir el polimorfismo en linealidad (más adelante profundizaré en ello). Las otras dos multiplicidades son operaciones suma y producto, que serán útiles en la composición de multiplicidades.

Algo a resaltar es que la sintaxis del sistema es suficientemente flexible como para permitir otros tipos de multiplicidades (con sus debidos ajustes en la semántica) que generen sistemas de tipos subestructurales distintos ([[Walker_2004]]), como por ejemplo:
- Tipos afines: cada valor debe ser utilizado a lo más una vez.
- Tipos relevantes: cada valor debe ser utilizado al menos una vez.
- Tipos ordenados: Cada valor debe ser utilizado exactamente una vez en su orden de creación.

Los tipos son los usuales, a excepción de que el tipo función ahora tiene una multiplicidad en la flecha. Esto servirá para indicar cuántas veces puede utilizarse el argumento.

Las declaraciones de datos se realizan al estilo de los tipos de datos algebráicos generalizados, en donde se definen los constructores como funciones. El hecho de que sean funciones, permite indicar la multiplicidad sobre la flecha función.

Los términos que tenemos son los usuales del cálculo $\lambda_{\rightarrow}$ con constructores y expresiones **let** y **case**. Nótese que las abstracciones $\lambda$ indican la multiplicidad del argumento. De la misma manera, tenemos abstracción y aplicación de multiplicidad. Las expresiones **case** y **let** indican la multiplicidad de los términos que ligan.