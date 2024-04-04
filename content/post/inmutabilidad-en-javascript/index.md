---
title: Inmutabilidad en Javascript
description: La inmutabilidad es genial y deberíamos usarla tanto como podamos. ¡No da tanto miedo como crees!
date: 2021-04-21
aliases: ["javascript","js","immutabilidad"]
tags:
  - programación
  - javascript
  - consejos
author: Fredrare
---

## Introducción
La inmutabilidad es un concepto que se basa en una idea muy sencilla: si algo tiene un valor, no puede cambiar. Esto no solo se ve en la programación, sino también en la vida real. Podríamos identificar rápidamente objetos prácticamente inmutables a nuestro al rededor: el sol, la luna, las estrellas, etc.
La confusión entra, para quienes quieren aprender a programar, cuando les dicen que hay ciertos elementos dentro de la programación que tienen esta característica: son inmutables.

Hoy estudiaremos cómo usar la inmutabilidad para hacer nuestro código más legible, menos propenso a errores y cómo sacarle la vuelta. En general, aprenderemos a cómo cambiar nuestro *mindset* con respecto a las variables

## Mutabilidad e inmutabilidad
La definición es sencilla: algo inmutable no puede cambiar. Esto se aplica tanto a referencias como a objetos. Primero tracemos la diferencia entre estos dos conceptos con un ejemplo: un hotel.

### Referencias
En el hotel tenemos habitaciones y tarjetas que nos permiten entrar a ellas. Cada tarjeta está asignada a un solo cuarto y nos permite saber qué hay en él. En este contexto, la tarjeta estaría relacionada con la habitación: haría **referencia** a la habitación. Sabemos que, cuando alguien deja este lugar, entregará la tarjeta en la recepción y la tarjeta dejará de estar asignada a ningún lugar. Cuando llegue otra persona, la tarjeta será reprogramada y, automáticamente, le permitirá entrar a una suite nueva. Ahora la misma tarjeta de antes está relacionada con otra habitación: hace **referencia** a otra habitación.

### Objetos
En este mismo hotel, las habitaciones referidas por las tarjetas son entidades en sí mismas. Cada una tiene un precio distinto, diferentes prestaciones y puede hacer más o menos cosas por quien se aloje en ellas. Estas **entidades**, que poseen propiedades y pueden hacer cosas, se llaman objetos. Estos, entonces, no son más que una representación. Una habitación puede ser un objeto y la cama y las almohadas que hay dentro de ella también son objetos, ya que cada una de estas también tiene atributos y utilidades distintas. Los **objetos son representaciones**.

#### Objetos mutables
Lo más normal sería pensar que los objetos pueden cambiar, ya que sabemos que es posible que las almohadas tengan una nueva funda o que se coloquen nuevas sábanas. De la misma manera, es posible que pinten la habitación o que se reemplacen las cortinas.

Otro tipo de cambios que podríamos observar sería cuando tenemos las latas de gaseosa en el cooler y vemos cómo la cantidad de líquido que tienen va disminuyendo conforme vamos tomándolo. También podríamos medir cómo disminuye la temperatura de verano cuando se prende el A.C. y cómo se oxidan las barandillas del balcón por estar cerca al mar.

#### Objetos inmutables
Si se pusieron a analizar los ejemplos anteriores verán que les he presentado dos ejemplos de naturalezas distintas. En primer caso hemos visto que los objetos en sí no han cambiado, sino que lo han hecho algunas propiedades anexas a estos.
1. Si cambio la funda de la almohada, la almohada, su funda anterior y la nueva siguen siendo todas iguales. Solo he cambiado las relaciones entre ellas, mientras que estas no han mutado en lo absoluto.

2. Si cambio la sábana de la cama, el escenario es exactamente el mismo.

3. Si pinto la habitación, la vieja pintura seguirá existiendo, solo que, dependiendo de cuánto pague por pintarla, será ocultada detrás de la nueva pintura o será removida y deshechada. La nueva pintura sigue siendo nueva y la pared sigue siendo la misma. Solo he cambiado las relaciones.

4. Reemplazar las cortinas sigue el mismo principio que los casos anteriores.

En este sentido, es posible que entendamos que aquello que normalmente entendemos como cambios en un objeto no son más que **cambios en las relaciones** entre objetos. Por ello, esos casos no representan ningún tipo de mutación. Por otro lado, en el segundo tipo de ejemplos, sí se observa este fenómeno.

1. La lata de gaseosa es un producto terminado. Es un objeto que comprende tanto a la lata como a la gaseosa. Desde el momento en el que se abre, su integridad ha cambiado. Cuando empezamos a tomar el líquido, el producto cambia (no comprarías una lata abierta ni una botella medio vacía). Aquí entendemos al producto como un todo que sí puede cambiar.

2. Cuando disminuye la temperatura, todos los objetos en la habitación cambian. Como la temperatura refleja la cantidad de movimiento de la partículas de los objetos, el A.C. afecta directamente la integridad de todas las cosas en la habitación.

3. Cuando las barandillas se oxidan, cambian sus moléculas a causa de la humedad que existe en el aire. El objeto se altera: muta.

Aquí podemos ver la diferencia entre estos dos tipos de cambios: a las relaciones (**referencias**) y a la integridad (**objetos**). Solo el segundo caso representa una mutación.

Con estos ejemplos y análisis, hemos podido notar que las referencias solo son una relación entre una entidad A y otra B. En el caso de javascript, la entidad A es la variable y la entidad B es el objeto.

## Javascript
En Javascript tenemos inmutabilidad en dos niveles: referencia y objeto. Primero hablemos sobre las referencias y, para esto, necesitamos saber sobre cómo declarar variables.

### Referencias: declaración de variables
Para declarar una variable en Javascript tenemos tres palabras reservadas: `var`, `let` y `const`. Cada una de estas tiene distintos propósitos y funcionalidades. Para efectos de este *post*, no entraremos en detalle a las diferencias entre estos 3, solo nos fijaremos en el aspecto referencial.

#### Usemos `var` y `let`
Estas dos palabras reservadas nos permiten declarar variables como referencias **mutables**. Veamos este ejemplo:
```javascript
var miVar = 5 // miVar apunta a un Number
miVar = 'Hola' // miVar apunta a un String
miVar = [1, 2, 3] // miVar apunta a un Array

let miLet = 'Fredrare.com' // miLet apunta a un String
miLet = 'Best site ever!' // miLet apunta a otro String nuevo
miLet = {isBestSite: true} // miLet apunta a un Object
```
Para ambos casos, he modificado las referencias. `miVar` y `miLet` hacían referencia a un objeto y yo pude cambiar esas referencias haciendo que ahora se relacionen con un objeto totalmente nuevo.

#### Usemos `const`
`const` es una palabra relativamente nueva en el vocabulario de Javascript. Nos permite declarar referencias constantes o **inmutables**, así que llamarlas variables no tendría mucho sentido ahora. Esto significa que la variable no puede hacer referencia a un objeto distinto una vez que ha sido creada. Veamos este ejemplo:
```javascript
const miConst = 'https://fredrare.com' // Ya no puedo reasignarlo
miConst = 'https://google.com' // ERROR: no puedo reasignar
miConst = 42 // ERROR: no puedo reasignar
```
En este caso, podemos ver que `miConst` ya no puede cambiar. Si yo declaré que `miConst` haga referencia a `'https://fredrare.com'`, ya no puedo hacer que haga referencia a otro valor luego. Esto puede sernos muy útil a la hora de escribir código, pues nadie que edite el código podrá accidentalmente cambiar la url y la página web, lo que nos permite tener un desarrollo más seguro. Además de esto, al usar `const`, quien lea el código sabe de antemano que se trata de una constante y no tiene que buscar en qué momento cambia el valor. De esta manera, el código se vuelve mucho más legible y fácil de digerir.

### Inmutabilidad en objetos
Dejando de lado las declaraciones, los objetos en sí pueden ser mutables o inmutables. Los objetos estrictamente hablando de la clase `Object` (`json`) son mutables, ya que podemos agregarles o quitarles características. Otros objetos, como los de la clase `String` o `Number` son totalmente inmutables. Pero de nada sirve que lo diga sin un ejemplo:
```javascript
let x = 1 // x hace referencia a un Number con valor 1
x = x + 1 // x hace referencia a un nuevo Number con valor 2

let s = 'Hey' // s hace referencia a un String con valor 'Hey'
s += ' you' // s hace referencia a un nuevo 'Hey you'
```
En ambos casos, con `Number` y `String`, los objetos no cambian. Si `x` valía `1`, `x` no ha aumentado su valor y ha mutado. Se ha creado un nuevo objeto con valor `2` y `x` ahora hace referencia a este nuevo objeto. El objeto con valor `1` ha sido abandonado y la memoria será liberada automáticamente en algún momento. El caso es exactamente el mismo con el `String`. `s` hacía referencia a un objeto con el valor `'Hey'`. Luego, se creó un nuevo objeto con el valor `'Hey you'` y `s` cambió para referir a ese nuevo objeto. El objeto anterior con valor `'Hey'` sigue existiendo en ese momento y será eliminado automáticamente luego.

### Mutabilidad en objetos
Ahora veamos algunos ejemplos de objetos mutables. Un muy buen ejemplo sería utilizar alguna instancia del tipo `Array`. Veamos esto en acción:
```javascript
let miArray = [1, 2, 3]
miArray.push(4) // miArray muta y ahora tiene un nuevo elemento
console.log(miArray) // [1, 2, 3, 4]
miArray.pop() // miArray se deshace de su último elemento
console.log(miArray) // [1, 2, 3]
```
Con este ejemplo, podemos ilustrar muy fácilmente una gran característica de los objetos mutables: pueden adquirir o perder elementos. Esto significa que mientras que la parte interna de estos objetos puede mutar, el objeto sigue siendo el mismo. Si nos fijamos bien, en ningún momento reasigné el valor de `miArray` con el operador `=`, como lo hice en todos los ejemplos anteriores. Esta es la prueba de que no he modificado la referencia. `miArray` siempre ha hecho referencia al mismo `Array`, solo que los elementos cambiaron porque un `Array` es mutable.

### Entendamos un poco más
Dado que sabemos que existe la inmutabilidad tanto para referencias como para objetos, el siguiente caso sería totalmente factible:
```javascript
const miArray = [1, 2, 3] // Creo un Array con 3 elementos
console.log(miArray) // [1, 2, 3]
miArray.push(4) // Añado un elemento al final de miArray
console.log(miArray) // [1, 2, 3, 4]
x.shift() // Elimino el primer elemento de miArray
console.log(miArray) // [2, 3, 4]
```
Ya vimos que const nos permite declarar constantes. Y también sabemos que las constantes son en realidad referencias inmutables. Esto significa que `miArray` siempre estará haciendo referencia al mismo objeto. No obstante, la palabra constante no se refiere a que el objeto al que hace referencia `miArray` no puede cambiar, sino a que `miArray` no puede hacer referencia a ningún otro objeto.

De esta manera, estamos combinando de una forma muy sencilla la inmutabilidad de una referencia con la mutabilidad de un objeto. Ambos conceptos no son incompatibles y, de hecho, es una buena práctica hacer esto.

#### ¿Por qué estaría bien usar constantes?
Usar `const` permite que nos aseguremos de que siempre estamos trabajando con el mismo objeto y, gracias a esto, evita muchas confusiones. Dado que el objeto en sí puede mutar, usar una constante no impide que la lógica de nuestro programa sea fluida. Lo único que hace es ordenar nuestro código para que no confundamos ni mezclemos referencias de una forma que haga ilegible el código en el futuro.

La regla de oro que se podría usar sería la siguiente: "Si no necesito reasignar el valor de mi variable, usaré `const`". Ya que es muy probable que creen variables que no modifiquen su valor, aplicando esta idea se darán cuenta de que su código está lleno de constantes que antes no habían notado, porque siempre las consideramos como variables. Y las variables, gracias a la mutabilidad, no siempre necesitan ser, justamente, variables.