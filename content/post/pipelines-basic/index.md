---
title: Pipelines básicos
description: Los pipes son de las herramientas más útiles que puedes tener en una shell
date: 2021-06-08
aliases: ["bash","pipe","pipelines","linux","unix"]
tags:
  - linux
  - scripting
author: Fredrare
---

## Introducción
Supongamos que quieres cocinar papas fritas. Si trataras de prepararlas sin poder usar *pipelines*, tendrías que seguir el siguiente proceso:
1. Lavar las papas y **guardarlas en un cajón**
2. Sacarlas del cajón, pelarlas y **guardarlas en otro cajón**
3. Sacarlas del cajón, cortarlas y **guardarlas en otro cajón**
4. Sacarlas del cajón, ponerlas a freír, sacarlas y **guardarlas en otro cajón**
5. Sacarlas del cajón, ponerles sal y **guardarlas en otro cajón**
6. Sacarlas del cajón y comerlas

Si he hecho bien mi trabajo, se puede notar algo bastante curioso: tengo que guardar las papas en algún lado luego de realizar cualquier acción con ellas. No obstante, naturalmente, nadie en sus cabales haría eso. Lo que se esperaría de nuestra realidad sería que pueda usar el producto que se obtiene en cada paso directamente en el paso siguiente. Eso es exactamente lo que nos permiten hacer los *pipelines* en una *shell*: tener un flujo más natural de información.

Si has visto `|` en un comando y no tienes idea de qué hace, estás en el lugar indicado (según yo). Ese símbolo se llama `pipe` y permite redirigir la salida de un comando a otro para poder hacer operaciones complejas.

La forma común en la que se usan es más o menos como esta:

```bash
command1 | command2 | command3 | command4 #...
```

El flujo exacto será explicado más adelante. No obstante, desde ya se puede ver que es una sintaxis bastante intuitiva y limpia. Da la idea de que los comandos siguen una línea uno luego de otro. Esta es la razón del nombre, porque *pipe* significa tubería. Así que podemos entender que, a partir de un comando principal, por donde entran datos, vamos pasándolos por una tubería, lugar en el que serán tratados de distintas maneras por distintos comandos. Todo esto para que obtengamos lo que queremos al final de la tubería: un *output* limpio.

## Ejemplos
### Filtrar la salida con `grep`
Los pipes son muy usados en diversos contextos. Uno de los más comunes va de usar grep para poder filtrar la salida de un comando dado.

Pongámonos en un escenario en el que yo tenga un archivo de texto de varias líneas. Si quiero ver el contenido en mi terminal, podría usar el comando cat:

```bash
$ cat texto 
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

Si yo quisiera buscar todas las veces en las que se menciona la palabra `better`, sería un poco complicado leer todo el texto. Para facilitarme el trabajo, puedo filtrar la salida con el comando `grep` de la siguiente manera:

```bash
$ cat texto | grep better
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Now is better than never.
Although never is often better than *right* now.
```

Lo que he hecho ha sido simplemente tomar la salida del primer comando (`cat texto`) y pasarla como la entrada de `grep`. Este último permite filtrar todas las líneas de acuerdo a un patrón que yo defina. Aquí, el patrón es que se deben filtrar todas las líneas que contengan `better`.

### Salida interactiva con `less`
Si nos ponemos en el ejemplo del archivo de texto nuevamente, podremos entender este caso de uso de una forma también sencilla.

Si el archivo de texto tiene muchas líneas, es un poco complicado desplazarnos por la terminal desde el inicio del archivo. Tendríamos que hacer *scroll* de forma muy constante y eso es un poco molesto jajajaja

Afortunadamente, el comando `less` permite que nos desplacemos por el texto de entrada de forma interactiva. Como ya vimos qué pasa si escribimos `cat texto`, ahora veamos qué sucede si escribimos el siguiente comando:

```bash
cat texto | less
```

El resultado sería el siguiente:

```txt
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
0Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
:
```

Podemos ver que el formato es distinto a la última vez. De hecho, en la salida no se puede ver el comando que hemos introducido. Esto se debe a que la terminal ha sido capturada por el *buffer* de `less`. En esta nueva pantalla, podemos movernos verticalmente haciendo *scroll* con el ratón (a veces) y con las teclas (arriba: flecha o `k`; abajo: flecha o `j`). Para salir podemos presionar la tecla `q`.

Hay que notar que, casualmente, el comando `less` puede tomar como parámetro el archivo de texto directamente (lo mismo sucede con `grep`), así que estas dos líneas son básicamente equivalentes:

```bash
cat texto | less
less texto
```

No obstante, sí se puede apreciar el verdadero poder de `less` y los *pipes* si, por ejemplo, estamos en una carpeta con muchísimos archivos o usamos algún comando con una salida más grande. Si usáramos `ls` en un directorio muy grande, estaríamos en un escenario similar al que acabamos de ver, así que tendríamos que usar un *pipeline* (esto lo puedes probar en tu *shell*, solo reemplaza `mi-directorio-grande` por un directorio grande xd).

```bash
ls mi-directorio-grande | less
```

### Redirección múltiple de salida con `tee`
Con el ejemplo 1 podemos hacer nuestro primer *pipeline* de tres partes. Imaginemos que queremos guardar todas las líneas del archivo texto que contienen `better` en un nuevo archivo llamado `solo-better`. Para eso, podríamos usar la redirección de flujo (tema de otro post), de la siguiente manera:

```bash
cat texto | grep better > solo-better
```

Si yo corro ese comando, obtengo el resultado fácilmente, pero no tengo *output* (la redirección de flujo se hace con `>` y manda la salida del comando `cat texto | grep better` al archivo `solo-better`).

Para ver el resultado de lo que he ejecutado, tendría que hacer `cat solo-better`. Esto es un poco trabajoso, puesto que tengo que esperar que se termine de realizar toda la operación para poder ver el resultado que ha sido guardado en un archivo.

Afortunadamente, tenemos al dios `tee`. Este comando nos permite guardar el texto en un archivo y mostrarlo en la pantalla al mismo tiempo (justo lo que buscamos). Podríamos usarlo de la siguiente manera:

```bash
$ cat texto | grep better | tee solo-better
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Now is better than never.
Although never is often better than *right* now.
```

### Combinando todo lo anterior
Basándonos en lo que he repetido varias veces, esto se ve bastante bien si se trata de archivos pequeños, pero, si tuviera muchas líneas, podría ser un poco engorroso usar el comando `tee` para ver toda la salida. Si el *output* filtrado tuviera unas 100 líneas, sería más conveniente poder hacer *scroll* directamente en la terminal, como lo que nos permite hacer el comando `less`, así que podemos integrarlo a nuestro *pipeline* de la siguiente manera:

```bash
cat texto | grep better | tee solo-better | less
```

Con el comando anterior, hacemos lo siguiente:

1. Obtenemos todo el contenido del archivo `texto`.
2. Filtramos ese contenido para obtener solo las líneas que contienen `better`.
3. Guardamos la salida en el archivo `solo-better`.
4. Mostramos la misma salida de forma interactiva en la *shell*.

En el siguiente post de *pipelines*, veremos cómo hacer operaciones un poco más complejas.

Hasta la vista xd