Las garantías que proporciona el uso de tipos lineales se pueden clasificar en dos categorías: seguridad y eficiencia.

Las garantías de seguridad ya se han analizado brevemente en [[Propiedades]]:
- **Preservación de tipos:** el tipado de todo término se mantiene invariante ante la evaluación.
- **Progreso:** la evaluación de programas bien tipados se realiza sin obstrucciones.

Un caso cuyas garantías no hemos revisado es el de las funciones divergentes.

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

Las funciones `repeat` e `ignore` son ejemplos de funciones que no cumplen el requisito para la condición de linealidad. Puesto que son funciones divergentes, la aplicación de las funciones `repeat x` e `ignore x y` no puede ser consumido en su totalidad exactamente una vez. Por lo tanto, no se cuenta con garantías para tales situaciones: puede o no pasar la verificación de tipos.

Las reglas para determinar si algo pasa la verificación de tipos sigue siendo dirigida por las reglas sintácticas del cálculo $\lambda_{\rightarrow}^q$, más no corresponden necesariamente a alguna noción intuitiva útil.

Por ejemplo, la siguiente modificación de `ignore` es observacionalmente equivalente a la versión anterior, sin embargo, no pasa la verificación de tipos:

```haskell
ignore :: a ⊸ a ⊸ [a]
ignore x _ = repeat x
```

Debido a que las funciones divergentes nunca consumen su resultado completamente, resulta sin utilidad que reciban argumentos de forma lineal.

Es por ello que el preludio de la biblioteca estándar `linear-base` elimina el uso de todas las funciones que en principio son divergentes. En general, una buena regla a seguir para los desarrolladores que hacen uso de tipos lineales es evitar escribir funciones divergentes, y de ser el caso, utilizar la flecha no-restringida.

## Compilación
El uso de tipos lineales, debido a las garantías que proporciona respecto al uso de los datos, nos permite razonar de una manera más precisa sobre el uso de la memoria, y para ciertos fragmentos, incluso resulta posible eliminar el uso del recolector de basura.

En [[Lafont_1988]] se describe una máquina abstracta que ejecuta código verificado bajo el análisis de tipos lineales. Esta no hace uso de un recolector de basura, sino que maneja la memoria como una estructura arbórea. La memoria se aloja y libera de forma automática como parte de las instrucciones básicas.

Además, la máquina poseé la capacidad de introducir efectos secundarios de forma segura.

Si bien la implementación en Haskell difiere en algunos aspectos de la máquina abstracta, se rescatan varias ideas como se menciona en [[Bernardy_2017]].

En particular, para los fragmentos en los cuáles trabajamos en contextos con tipos lineales, el compilador de Haskell puede emitir código que prescinde del recolector de basura: puesto que cada dato se utiliza una única vez, existe un único dueño de la información en cada punto del programa, removiendo la necesidad de copias o múltiples apuntadores cuya memoria habría que liberar.

Además, los datos se pueden alterar *en-sitio*. Es decir, los datos realmente son mutables, aunque el lenguaje preserva una interfaz con todas las garantías de los datos inmutables. Se verá un ejemplo de esto más adelante al revisar el caso de uso de Arreglos.

## Expresividad
Una de las características más útiles de Haskell, y en general de los lenguajes de programación con sistemas de tipos avanzados, es la capacidad de utilizar los tipos para expresar más que sólo los tipos de entrada y retorno.

Los tipos se pueden utilizar para expresar reglas del negocio, algunas garantías e incluso comportamiento. Con los tipos lineales, ahora es posible utilizar el sistema de tipos para expresar multiplicidad e incluso alcance de un dato.

Ejemplo de esto es el uso de tipos lineales en los módulos de Entrada/Salida. Con los tipos lineales es posible definir funciones como:

```haskell
closeFileHandle :: Handle ⊸ IO ()
writeToSocket :: Binary ⊸ SocketState ⊸ IO SocketState
```

que codifican como parte del tipo el hecho de que un descriptor de archivo sólo se puede utilizar una única vez en un estado dado. Es decir, debemos perder acceso al descriptor de archivo una vez que se cerró; y cada escritura subsecuente a un *socket* modifica el estado de este.