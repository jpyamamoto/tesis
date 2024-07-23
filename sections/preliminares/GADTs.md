Los Tipos de Datos Algebráicos Generalizados (GADTs por sus siglas en inglés) son una forma más general de definir tipos de datos. Contrario a la intuición, esta generalización permite definir tipos más particulares. De aquí que algunos autores prefieran el nombre **constructores restringidos recursivos para tipos de datos**.

Lo que hacen estos tipos de datos es permitir que el desarrollador sea explícito en la firma de los constructores para un tipo de dato. Esto tiene varias ventajas:

- **Polimorfismo Ad-Hoc por constructor.** Puesto que cada constructor puede tener una firma distinta, entonces cada constructor puede hacer uso de distintos tipos de datos restringidos.
```haskell
data Example a where
	ExampleReal :: Real a => a -> Example a
	ExampleFrac :: Fractional a => a -> Example a
```
- **Tipos de retorno más específicos.** Dado que la sintaxis nos permite indicar la firma del tipo de dato, podemos ser más específicos en el tipo de dato de retorno de cada constructor.
```haskell
data Term a where
	NumTerm  :: Int  -> Term Int
	BoolTerm :: Bool -> Term Bool
```

Esta sintaxis se ha introducido en varios lenguajes como Coq, Lean, Scala, y en Haskell mediante la extensión `GADTs`.