El polimorfismo de multiplicidad corresponde a la posibilidad de escribir funciones con argumentos cuya multiplicidad no es conocida.

Esta propiedad se hereda del cálculo $\lambda_{\rightarrow}^q$, con la distinción de que en Haskell no tenemos la posibilidad de escribir multiplicidades suma o producto.

En ese sentido, el polimorfismo de multiplicidad en Haskell es menos expresivo, y su única utilidad es en evitar la repetición de código que podría usarse para su versión tanto lineal como no-lineal.

No obstante, puesto que la suma y el producto de multiplicidades en casi todos los casos resultan en $\omega$, no sería de gran utilidad tener las operaciones sobre las multiplicidades al nivel de los tipos. Siempre que el desarrollador tenga una multiplicidad suma, puede cambiarse por la flecha no restringida, y las multiplicidades producto son lineales siempre y cuando sus partes sean lineales.

Para comprender el funcionamiento del polimorfismo de multiplicidad en Haskell, hay que notar que existe una relación de orden en las multiplicidades dado por $1 \leq \omega$. Dada esta relación, podemos asumir que toda función con argumento lineal, puede verse de forma no-lineal y continuará pasando la verificación de tipos. El inverso no es necesariamente cierto.

De lo anterior se desprende el cómo funciona el polimorfismo de multiplicidad. Dada una función con firma polimórfica en su multiplicidad, basta verificar los tipos en su versión lineal. De ser así, basta cambiar las multiplicidades `One` a `Many` para tener una versión no-lineal con tipos correctos.

Es decir, el polimorfismo de multiplicidad se puede pensar como un mecanismo para generar ambas versiones (lineal y no lineal) de una función.

Cabe mencionar que esto no es del todo correcto al nivel de implementación, puesto que la cantidad de funciones generadas sería exponencial en el número de argumentos polimórficos. De la misma manera que las funciones polimórficas en tipos no generan todas las posibles instancias para tipos existentes. No obstante, la ilustración resulta didáctica.

Así como tenemos sintaxis para `a %1 -> b` y `a -> b`, el polimorfismo de multiplicidad tiene su representación en menor nivel. Similar al polimorfismo en tipos, se puede ver como una cuantificación universal sobre `Multiplicity`.

```haskell
a %p -> b ≡ forall a b (p :: Multiplicity) . a %p -> b
```

donde `a` y `b` están cuantificados sobre tipos, y `p` sobre la enumeración `Multiplicity` definida anteriormente.

Revisemos el siguiente ejemplo. Tomemos el operador `$` que sirve para aplicar su argumento izquierdo a su argumento derecho. Es decir, `f $ x ≡ f x`.

Si no tenemos tipos lineales ni polimorfismo de multiplicidad, su definición sería la siguiente:

```haskell
($) :: (a -> b) -> a -> b
($) f x = f x
```

Este mismo operador se ajusta a la definición donde el primer argumento es lineal, pues sólo utilizamos el parámetro `f` una vez:

```haskell
($) :: (a -> b) ⊸ a -> b
($) f x = f x
```

No sólo eso, sino que además, si la función dada como argumento es lineal, el segundo argumento también debe ser lineal:

```haskell
($) :: (a ⊸ b) ⊸ a ⊸ b
($) f x = f x
```

De forma que tenemos entonces dos niveles de multiplicidades: cuántas veces se utiliza la función dada como argumento, y cuántas veces utiliza tal función su argumento.

Podemos hacer polimórfica a la función sobre la multiplicidad de su primer argumento, como sigue:

```haskell
($) :: (a -> b) %q -> a -> b
($) f x = f x
```

Y además, podemos hacer polimórfica la multiplicidad de la función dada y el argumento que utiliza, siempre y cuando tengan la misma multiplicidad:

```haskell
($) :: (a %p -> b) %q -> a %p -> b
($) f x = f x
```

Por lo tanto, es correcto usar el operador en cualquiera de las siguientes formas:

```haskell
f1 :: Int -> Int
f1 x = x * x

f2 :: Int ⊸ Int
f2 x = x + 1

v :: Int
v = 5

app1 :: (a -> b) -> a -> b
app1 = $

app2 :: (a -> b) ⊸ a -> b
app2 = $

app3 :: (a ⊸ b) -> a ⊸ b
app3 = $

app4 :: (a ⊸ b) ⊸ a ⊸ b
app4 = $

f1 `app1` v
f1 `app2` v
f2 `app3` v
f2 `app4` v
```