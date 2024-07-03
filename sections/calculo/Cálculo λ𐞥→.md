Existen diversas propuestas de extensiones al cálculo $\lambda$ para sistemas de tipos que corresponden a la lógica lineal bajo la correspondencia *"proposiciones como tipos"*.

Para el caso de Haskell se propone el cálculo $\lambda_{\rightarrow}^q$ que se enfoca en el fragmento implicacional de la lógica lineal. Es decir, serán ignorados los operadores $\oplus, \otimes, \&,$ ⅋; los exponenciales $!, ?$ y las constantes de verdad y falsedad. Tenemos un fragmento muy reducido, en donde únicamente consideramos la implicación lineal $\multimap$ y se hace compaginar su comportamiento junto con la implicación usual $\rightarrow$.

Aquí no estamos considerando la definición de la implicación $\rightarrow$ presentada por Girard como: $A \rightarrow B \equiv (!A) \multimap B$ ([[Girard_1995]]). Tampoco la definición dada por Wadler en su presentación del cálculo $\lambda$ con tipos lineales y convencionales combinados: $A \rightarrow B \equiv !(A \multimap B)$ ([[Wadler_1990]]). Más bien, estamos asumiendo que ambas implicaciones $\rightarrow$ y $\multimap$ son primitivas.

Más adelante veremos que a partir de estas implicaciones, junto con los tipos de datos algebráicos generalizados, es posible definir también el exponencial $!$ para poder escapar las restricciones lineales de una forma segura.

Además, este sistema cuenta con polimorfismo de linealidad. Es decir, tenemos un mecanismo para definir funciones que se comportan correctamente tanto en contextos lineales como no-lineales.

El sistema $\lambda_{\rightarrow}^{q}$ se presenta en [[Bernardy_2018]], y la documentación de los aspectos técnicos de la implementación en Haskell como la extensión LinearHaskell se presentan en la documentación [[GHCTeam_2023]]. Además, hay variedad de recursos que se listan en las referencias que profundizan en la implementación y uso de tipos lineales en Haskell.