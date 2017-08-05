¿Por qué JS?
- Está en todos lados, en los últimos años surgieron un montón de lenguajes que se compilan a JS.
- Desde hace un tiempo que JS está incorporando cosas funcionales, y el enfoque funcional es muy útil para trabajar con un pipeline.
- Más fácil y sin tanto bajage teórico como otros lenguajes funcionales.

Cosas a las que le apuntamos:
- Construcción de un lenguaje desde un enfoque ingenieril, apuntado al uso, sin tal vez aprender tooooda la teoría de fondo. Aprendiendo sobre la marcha aquello que sea necesario.
- Ir haciendo cosas que resulten de interés y que sean compartidas.

---

Teoría básica como para empezar a hablar...

Sintaxis: cómo se escribe en el lenguaje.
Semántica: carga de intención de la mano de las palabras que forman parte del lenguaje.

Un lenguaje no es un programa, sino que tiene una sintaxis y una semática que luego puede ser implementado de diversas formas. Ej Ruby y ECMA Script son ideas/contratos que luego fueron implementados de distintas formas.

El compilador es aquel que toma el código fuente, escrito en el lenguaje que nos haga más felices, y lo traduce a código máquina para que pueda ser ejecutado.

Hay una serie de pasos bien conocidos que forman parte del proceso de compilación: validar que el programa construído tiene sentido, generar una construcción intermedia que sea una abstracción más interesante que una secuencia de caracteres pero de más alto nivel que el código de máquina (Abstract Syntax Tree, AST para los amigos).

Parser: programa que toma código fuente y genera un AST.

Una expresión como la siguiente:
`(2+3)*5`

Podría parsearse para generar un AST como el siguiente:
```
*
|-> +
    |-> 2
    |-> 3
|-> 5
```

Otra forma tradicional de trabajar es separar el mecanismo de parseo en dos partes:
- Lexer/tokenizer: toma el string y lo parte en cosas más chiquitas y manejables, que serían los tokens.
- Se toma lo que escupe el lexer y se genera el AST.

También existen formas más avanzadas de trabajar como los parser combinators.

Lo que vamos a estar usando es directamente una herramienta que toma como entrada la gramática, y a partir de eso se parsea el código para generar el AST.

Una de las gracias del AST es que, al ser un árbol, es muy fácil de recorrer y no hay ciclos, lo cual allana el camino para el proceso que tome ese AST y genere algo que pueda ser corrido.

Cosas que el compilador hace a partir del AST:
- validaciones, para asegurar que las construcciones tengan sentido
- tipado
- escupir algo que pueda ejecutarse

Linker: Parte del proceso de compilación que transforma el AST en algo que está bien cableado, teniendo en cuenta scopes, imports... y lo transforma en un grafo ejecutable, precalculando información para que no tenga que hacerse en ejecución. También puede incluir optimizaciones y chequeos.

Luego la última parte de la compilación es bajar la información a código máquina (en nuestro caso eso sería pueda correr en el browser mediante el intérprete de javascript - no nos interesa entrar en detalle de si un lenguaje se categoriza como compilado o interpretado). En esta última parte es donde está el peso de la semántica, ya que es donde se determina qué hay que hacer cuando se ejecuta.

A veces se hacen concesiones en la sintaxis, siendo más abarcativa de lo que realmente queremos admitir como válido, teniendo en cuenta que alguno de los procesos posteriores rechace algunas de esas cosas una vez que ya se tiene la estructura intermedia contruída.

Ejemplos de herramientas cotidianas que se basan en la información intermedia
-Un buen IDE trata de encontrar todos los errores posibles de compilación de una sola vez para que puedan ser corregidos más fácil, que se hace mediante un just in time compiler que trabaja sobre porciones del programa.
-Coloreo de sintaxis
-Linter: algo que revisa el código y hace sugerencias, sobre cosas que no son errores.

Como punto de partida vamos a arrancar por un parser, no necesariamente para llegar a un AST (que también es un lenguaje, sólo que no es un texto). Por ejemplo, podríamos parsear JSON o código morse. Luego vamos a tratar de ir avanzando de a poquito hacia la otra punta del proceso de compilación.

-----

Acá tenemos un par de definiciones de lenguajes por comprensión:

L1 = {aᴺ / n pertenece a los naturales positivos sin el 0} => Es un **lenguaje regular**

L2 = {aᴺbᴺ / n pertenece a los naturales positivos} => Es un **lenguaje independiente del contexto**

L3 = {aᴺbᴺcᴺ / n pertenece a los naturales positivos} => Es un **lenguaje dependiente del contexto**

Con L1 podemos decir que aaa es una palabra válida, con L2 podemos decir que aabb es una palabra válida pero abb no lo es.

Lenguaje regular: es un lenguaje formal que puede ser definido por una gramática regular. Esto significa que puede definirse usando terminales (a), o unión de terminales (a|b), o una secuencia de terminales (ab), o una secuencia de 0 a n de un terminal (a*) aka estrella de Kleene. Luego esto mismo puede extenderse a cosas como que la unión de dos lenguajes regulares es un lenguaje regular y cosas como esas.

L1 puede escribirse con la expresión regular: aa* que también se puede escribir como a+

L2 no es regular, no hay forma de escribir una expresión regular que pueda validar que la secuencia de a y la secuencia de b son de la misma longitud. Es independiente del contexto porque puede definirse con una gramática independiente del contexto.

Gramática: conjunto de reglas con cierta forma conocida
-No terminales: con mayúscula
-Terminales: con minúscula

Basándose en reemplazos con reglas de gramática, luego de una cantidad finita de aplicaciones, si se llega sólo a terminales se llega a la gramática.

Si L nos representa nuestro L2, podemos definir:
`L -> aLb | ε` (ε=vacío)

Los lenguajes regulares también pueden definirse con una gramática independiente del contexto. Por ejemplo para L1:
```
L -> aL
L -> a
```

Hay que tener cuidado con las recursiones a izquierda, o sea: `L -> La` tendría problemas porque genera un loop infinito al procesarla.

En un lenguaje dependiente del contexto vale poner un terminal a la izquierda de la -> en la gramática, por ejemplo `bL -> a`. Esto nos lleva a cosas muy complejas en las que **no queremos caer**.

Lo importante es que manejemos expresiones regulares (para cosas típicas como por ejemplo identificadores y para porciones de nuestros problemas) por una cuestión de simplicidad, y gramáticas independientes del contexto (porque por ejemplo si querés tener paréntesis que abren y cierran la expresión regular no sirve).
