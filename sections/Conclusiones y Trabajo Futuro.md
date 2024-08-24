## Conclusiones
A lo largo de este trabajo se describieron conceptos varios que forman parte del corpus de la Programación Funcional y la Teoría de Tipos. En particular se puso al lenguaje de programación puramente funcional Haskell como un punto central en la investigación, utilizándolo como un caso de estudio de interés por la profunda integración con los avances académicos en estas áreas.

Comenzamos con una introducción a la Lógica Lineal, en donde pudimos apreciar el uso de la implicación lineal y cómo esta nos sirve para imponer garantías cuantitativas sobre el uso y acceso a los recursos.

Durante la segunda sección de este trabajo analizamos el Cálculo $\lambda_{\rightarrow}^q$. Esta extensión al Cálculo Lambda nos permitió estudiar una forma de integrar la implicación lineal en un sistema de tipos con datos algebráicos, como una función con garantías sobre el uso sus parámetros bajo la correspondencia Curry-Howard. También se revisó como la sintáxis y la semántica de este sistema dan lugar a ciertas extensiones más allá de los tipos lineales, que podrían estudiarse a futuro; ejemplo de esto son los tipos afines que han sido brevemente estudiados por otros autores.

En la tercera sección revisamos la implementación que han realizado los desarrolladores de GHC a través de la extensión *LinearTypes*. Se discutieron las consecuencias de algunos detalles en la implementación, las garantías derivadas del sistema $\lambda_{\rightarrow}^q$ y las optimizaciones que se pueden realizar durante la compilación aprovechando los beneficios en la semántica de los tipos lineales.

También se examinó el concepto de Polimorfismo de Multiplicidad como una técnica para la generación de funciones en sus dos variantes de multiplicidad: lineal y no-restringida.

Como último punto de la implementación de *LinearTypes* se analizó la definición del tipo `Unrestricted` a partir de un tipo de dato algebráico generalizado, para escapar de un contexto lineal. También se describieron algunas utilidades que se han desarrollado para facilitar el uso de valores sin restricciones en contextos lineales.

Durante la última sección se tomo como objeto de estudio el caso de *Arreglos* utilizando las herramientas mencionadas anteriormente. Los *Arreglos* fueron de interés debido a que, por su naturaleza imperativa, los lenguajes de programación funcionales (en particular aquellos con transparencia referencial) suelen presentar dificultades para su construcción. Se hizo explícito el hecho de que Haskell, previo al uso de tipos lineales, requería sacrificar eficiencia o seguridad, y en general no era posible contar con ambos.

Los tipos lineales permiten expresar en la misma semántica del programa las garantías suficientes para asegurar que los arreglos pueden ser utilizados de forma eficiente en un contexto seguro.

Esto ha permitido el desarrollo de distintas interfaces alrededor del concepto de arreglos con distintos propósitos:
- Arreglos mutables simples: una implementación segura y eficiente de los arreglos en su presentación usual.
- Arreglos Destino: una interfaz para construir arreglos utilizando el *estilo de paso de destino*.
- Arreglos Polarizados: un flujo para construir por arreglos por etapas, reduciendo la interacción con la memoria.

Mediante este trabajo se ha logrado el objetivo de estudiar el concepto de linealidad en los dos frentes que interesan a las Ciencias de la Computación por la correspondencia Curry-Howard: desde la lógica y desde los lenguajes de programación.

Un punto importante de este trabajo fue el conocer las ventajas que resultan de la cercanía entre la lógica y los sistemas de tipos, brindando beneficios tangibles como lo es en el caso de los arreglos.

Este es un claro caso de éxito que surge de la colaboración entre la teoría y la práctica.

## Trabajo Futuro

Si bien la lógica lineal ha sido ampliamente estudiada en las últimas décadas, todavía hay mucho que puede analizarse desde el lado de Ciencias de la Computación.

Específico a Haskell, aún hay varios aspectos que pulir en la implementación:
- El uso de `let` y `where` en contextos lineales. Si bien es posible utilizarlos, el desarollador aún no tiene la capacidad de hacer explícito que el tipo de las definiciones tienen restricciones de linealidad, y en general Haskell va a asumir prematuramente la no-linealidad.
- El uso de `case` y el emparejamiento de patrones. El análisis de patrones tiende a asumir no-linealidad, obligando al programador a escribir código más complicado en situaciones donde un mejor algoritmo de inferencia podría evitarlo.

Un ejemplo de tales deficiencias en la implementación se encuentra en el [issue 25081](https://gitlab.haskell.org/ghc/ghc/-/issues/25081) que el autor de este trabajo encontró, donde el uso de la sintaxis de listas por comprehensión permitía escapar de forma insegura la linealidad. Esto ahora ha sido resuelto, pero puede ser indicativo de la existencia de aspectos en la sintaxis que todavía no han sido probados exhaustivamente en contextos lineales.

Otro tema que resulta de interés son las *Restricciones Lineales* (linear constraints), que es presentado en  [[Spiwack_2022]] y [[Spiwack_2023a]]. Esta característica permite codificar la linealidad de las operaciones a manera de Clases en el sistema de tipos de Haskell, etiquetando el contexto en que se realizan las operaciones para garantizar la compilación a funciones lineales. De implementarse, esto reduciría la cantidad de código que requiere actualmente hacer uso de tipos lineales, y simplificaría la sintaxis.

En [[Spiwack_2020]] se discute el tema de Excepciones en el contexto de tipos lineales, su relación con tipos afines, y algunas ideas que han sido consideradas para su implementación en la extensión Linear Types. Considero que aún hay espacio para un análisis más detallado, y el desarrollo de utilidades alrededor de estas abstracciones.

Otra aplicación interesante de los tipos lineales es su uso como primitivas para el desarrollo de bibliotecas que permitan el manejo manual de memoria. Se presenta la teoría para lograrlo en  [[Walker_2004]], y existe una posible implementación en [[Mesquita_2024]]. Aún hay posibilidad de ahondar en el uso de tales bibliotecas y su aplicación para simular ciertas capacidades de los lenguajes imperativos en lenguajes funcionales de forma segura.
