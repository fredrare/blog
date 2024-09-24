---
title: Tipos útiles para TypeScript
description: Algunos tipos con restricciones lógicas entre llaves, en buildtime
date: 2024-09-19
aliases: ["tipos","TypeScript"]
tags:
  - programación
  - TypeScript
  - trucos
author: Fredrare
---

## Introducción
TypeScript nos sirve principalmente para poder definir tipos que nos advierten de errores en *build-time*. Bajo esa premisa, la principal ventaja que nos brinda sobre JavaScript es que nos advierte de errores que **vamos** a tener si hacemos algo mal en el código con respecto a los tipos de datos que mandamos de un lugar a otro en nuestro código. En este sentido, en este artículo usaremos a TypeScript como una herramienta de metaprogramación y veremos cómo podemos aplicar lógica condicional y repetitiva en nuestros amados tipos con dos ejemplos sencillos.
Antes de comenzar, considera que lo siguiente estará sumamente simplificado y habrá casos de uso en los que el ejemplo "subóptimo" será el mejor.

## Llaves mutuamente excluyentes
Imaginemos que tenemos una función que recibe un objeto como único parámetro. Para efectos prácticos, digamos que el objeto es de tipo `Rect`, que representa un rectángulo. Así que podríamos definir Rect de la siguiente manera:
```typescript
type Rect = {
  width: number
  height: number
  aspectRatio: number
}
```
Claro que este tipo sirve para definir un rectángulo, pero evidentemente es repetitivo en el sentido de que el `aspectRatio` es innecesario si ya tenemos `width` y `height`. Podríamos perfectamente tener solo `width` y `aspectRatio` y la lógica del programa debería encargarse de hacer lo suyo, sin necesidad de repetir los datos que estamos mandando de un lado a otro. Obviamente, podríamos (**y deberíamos**) hacer las validaciones respectivas en *runtime* para que todo salga como se espera, pero *por los loles*, hagamos que TypeScript nos obligue a tomar una de estas dos opciones: o se define `height` o `aspectRatio`, pero no ambas a la vez.

### Solución sencilla
Una primera idea podría ser esta:
```typescript
type Rect = {
  width: number
} & (
  { height: number }
  | { aspectRatio: number }
)
```
Pero la union entre `{ height: number }` y `{ aspectRatio: number }` nos permitiría definir ambos atributos para el objeto. Esto se debe a que estamos usando una unión. Entonces, una forma sencilla y directa para lograr el efecto deseado sería la siguiente:
```typescript
type Rect = {
  width: number
} & (
  { height: number; aspectRatio?: never }
  | { height?: never; aspectRatio: number }
)
```

La lógica que viene detrás es que `width` será siempre un atributo necesario en `Rect`, pero si `height` se encuentra definido, `aspectRatio` no deberá encontrarse en el mismo objeto. De la misma manera, si `aspectRatio` tiene un valor, `height` no deberá existir en dicha instancia.

En este punto, creo que deberíamos empezar a ver que hay un patrón que podría repetirse y si no nos encargamos de él, podría hacer que nuestro código sea bastante engorroso. Si tuviéramos 3 atributos mutuamente excluyentes, sería necesario modificar cada uno de los elementos de la unión para agregar la nueva llave y, unir un nuevo tipo con todas las otras llaves en `?: never` y el nuevo valor definido. En síntesis, es muy trabajoso tener que mantener este esquema cada vez que queramos implementar la lógica de llaves mutuamente excluyentes. Por lo tanto, es un método muy propenso a errores.

Entonces, lo lógico es pensar que debe haber una forma de implementar esto de forma estandarizada que nos permita fácilmente recrear estos tipos sin preocuparnos por la implementación (básicamente, no desviarnos del principio [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)). Así que vamos al meollo del asunto.

### Solución generalizable
#### Idea
La idea es la siguiente: tenemos que crear un tipo que replique la estructura de uniones que ya observamos. En cada elemento de la unión debemos encontrar una única llave con un tipo de dato definido y el resto de llaves com `?: never`. Además, necesitamos una forma práctica de invocar todo esto. Podremos llamar a este tipo `Exclusive<T>`, donde `T` representa un tipo cuyas llaves son todas excluyentes. Solo para ilustrar esto mejor, nuestro propósito es implementar algo que nos permita hacer esto con cualquier cantidad de llaves exclusivas que se nos ocurran:
```typescript
// Debería ser equivalente a la solución sencilla anterior
type Rect = {
  width: number
} & Exclusive<{
  height: number
  aspectRatio: number
}>
```

#### Utility types
Ahora que hemos destilado bastante bien lo que necesitamos hacer, debemos conocer ciertos [Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html) de TypeScript. Los voy a listar con una descripción **muy** breve y simplificada:
- [`Pick<T, Keys>`](https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys):  Crea un nuevo tipo solo con las `Keys` de `T`.
- [`Required<T>`](https://www.typescriptlang.org/docs/handbook/utility-types.html#requiredtype): Crea un nuevo tipo con todas las llaves de `T` obligatorias.
- [`Exclude<U, E>`](https://www.typescriptlang.org/docs/handbook/utility-types.html#excludeuniontype-excludedmembers): Crea un nuevo tipo en el que la unión `U` ya no cuenta con elementos que cumplan también con `E`.
- [`Record<Keys, T>`](https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkeys-type): Crea un nuevo tipo solo con llaves `Keys` y valores de tipo `T`.
- [`Partial<T>`](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype): Crea un nuevo tipo con todas las llaves de `T` opcionales.

#### Implementación base
Para solucionar el problema, iré construyendo la solución desde afuera hacia adentro. Esto significa que desde la primera iteración, tendremos el tipo `Exclusive<T>` definido, aunque no funcione como esperamos.

Iniciamos simplemente definiendo el genérico:
```typescript
type Exclusive<T> = {}
```

Ahora, haré que esto produzca una unión, con una cantidad de elementos igual a las llaves de `T`. Para ello, definiremos todas las llaves de T como llaves del tipo resultante e iteraremos sobre todas las llaves con `[keyof T]`. Esto nos dará como resultado una unión de tipos.
```typescript
// Si T es { a: number; b: number }
// Exclusive<T> sería "a" | "b"
type Exclusive<T> = {
  [Key in keyof T]: Key
}[keyof T]
```

Ya que tenemos una forma práctica de generar uniones con `n` elementos, procederemos a definir cada uno de esos elementos. De acuerdo a lo que hemos visto, cada uno de los elementos será todo lo que vaya luego de `[Key in keyof T]:`, donde `Key` es nuestra llave de iteración que representa cada una de las llaves de `T`. Entonces, comencemos por definir un tipo por cada uno de esos elementos iterativos:
```javascript
// Exclusive<T> sería ahora { a: number } | { b: number }
type Exclusive<T> = {
  [Key in keyof T]: Pick<T, Key>
}[keyof T]
```

Aquí podemos ver que ya hemos replicado la primera idea que habíamos definido, que era incorrecta. Así que podemos seguir armando la solución. Para ello, no está de más expresar explícitamente que la llave en cuestión será requerida, ya que en cada elemento de la unión resultante, al menos una llave debería ser requerida para poder expresar la exclusividad que buscamos.
```javascript
// Exclusive<T> todavía es { a: number } | { b: number }
type Exclusive<T> = {
  [Key in keyof T]: Required<Pick<T, Key>>
}[keyof T]
```

En este momento corresponde llenar el resto del elemento de `?: never` por cada una de las otras llaves de T que no son `Key`. Iniciemos simplemente poniendo las otras llaves en una intersección y luego vemos cómo las ponemos con el valor correspondiente:
```javascript
// Exclusive<T> es ({ a: number } & "b") | ({ b: number } & "a")
type Exclusive<T> = {
  [Key in keyof T]:
    Required<Pick<T, Key>>
    & Exclude<keyof T, Key>
}[keyof T]
```

Aquí podemos notar que todavía estamos ante una estructura sin mucho sentido, pero ya tenemos a un lado de la interesección nuestra llave requerida y al otro las negadas. Luego, crear un tipo en lugar de simplemente tener las llaves en unión. Básicamente, en este punto quiero pasar de `"a"` a `{ a: never }`.
```javascript
// Exclusive<T> es ({ a: number; b: never }) | ({ b: number; a: never })
type Exclusive<T> = {
  [Key in keyof T]:
    Required<Pick<T, Key>>
    & Record<Exclude<keyof T, Key>, never>
}[keyof T]
```

Ya tiene casi la forma que esperamos. Ahora solo nos queda hacer que todas las llaves negadas sean opcionales, para que TypeScript no nos obligue a ponerles un valor imposible, ya que todas se encuentran con `never`.
```typescript
// Exclusive<T> es ({ a: number; b?: never }) | ({ b: number; a?: never })
type Exclusive<T> = {
  [Key in keyof T]:
    Required<Pick<T, Key>>
    & Partial<Record<Exclude<keyof T, Key>, never>>
}[keyof T]
```

Y con eso, definimos correctamente el genérico `Exclusive<T>`. Este tipo tiene la implementación necesaria para que podamos expresar de manera sencilla la exclusividad de llaves cada vez que la necesitemos.

## Llaves no excluyentes, pero necesarias
Con el problema anterior resuelto, me parece razonable hacer un tipo genérico `Some<T>` que se comporte de manera similar a `Exclusive<T>`. La diferencia es que `Some<T>` deberá hacer que **al menos una** de las llaves de `T` esté definida, con el resto de forma opcional. De manera más directa:
```typescript
type Some<T> = { ... }

type Example = Some<{
  a: number
  b: number
  c: number
}>

// Error, no hay nada definido
const foo: Example = {}

// Correcto, al menos una llave definida
const bar: Example = { a: 1 }
const baz: Example = { a: 1, b: 2 }
const qux: Example = { b: 2, c: 3 }
```

La solución no la haré aquí, pero te sugiero que implementes `Some<T>` para que compruebes que no es complicado.

¡Saludos!