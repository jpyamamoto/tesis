La meta-teoría para hablar de las propiedades del cálculo $\lambda_{\rightarrow}^q$ se presenta en [[Bernardy_2018]], utilizando dos propuestas: una semántica operacional *ordinaria* y otra semántica *denotacional*.

La semántica operacional *ordinaria* se presenta en el estilo de paso grande, con la particularidad de que utiliza primitivas para permitir la mutación de ciertos términos. Normalmente esto es algo que se evitaría en las semánticas de lenguajes puramente funcionales con evaluación perezosa, pues usualmente esto introduciría no-determinismo al no conocer el orden en que se ejecutan las instrucciones, perdiendo la transparencia referencial.

Para garantizar que lo anterior no sucede y que efectivamente el lenguaje sigue siendo seguro incluso ante la presencia de mutación en un lenguaje funcional con evaluación perezosa, hay que analizar las propiedades que garantiza el uso de tipos lineales, para lo cuál la semántica ordinaria no es tan útil.

La semántica ordinaria resulta mucho más sencilla de implementar como el motor detrás de lenguajes reales, como lo sería Haskell. Pero presenta muchas dificultades en su estudio.

Los teoremas que garantizan la seguridad de $\lambda_{\rightarrow}^q$ son:

Dada la relación de reducción $\longrightarrow$ en la semántica de paso grande, podemos dar su cerradura reflexiva $\Downarrow$. Eventualmente se extiende la definición a una relación de evaluación parcial $\Downarrow^*$ mediante la cerradura reflexiva y transitiva.

**Teorema.** *Preservación de tipos.*
Si $a:A$ es un término bien tipificado, y $a \Downarrow b$ o $a \Downarrow^* b$, entonces $b:A$ es un término bien tipificado.

**Teorema.** *Progreso.* La evaluación es no-obstructiva. Es decir, para cualquier evaluación parcial $a \Downarrow^* b$ con $a$ bien tipificado, la evaluación siempre puede extenderse (existe un $b'$ tal que $b \Downarrow^* b'$ y $a \Downarrow^* b'$).

La intuición detrás de este teorema es que la evaluación de términos "atómicos" siempre puede extenderse puesto que $a \Downarrow a$ (por ser reflexiva la relación). Para otros casos, el término puede eventualmente reducirse a un término atómico, o la evaluación puede diverger.

Los teoremas anteriores no se demuestran directamente, pues están dados en la semántica operacional. Para su demostración se realiza un desvío, dando los teoremas equivalentes y sus demostraciones en la semántica denotacional.

La semántica denotacional utiliza un modelo que permite abstraer el lenguaje y su ambiente de ejecución como objetos matemáticos. Estos permiten razonar de manera más sencilla sobre las propiedades del lenguaje, y es por ello que se utilizan en la demostración de los teoremas importantes del sistema.

Finalmente, se apela a un tercer teorema que relaciona la semántica operacional ordinaria con la semántica denotacional.

**Teorema.** *Equivalencia observacional*.
La semántica con mutación en-sitio (operacional) es observacionalmente equivalente a la semántica pura (denotacional).
