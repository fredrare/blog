---
title: Pipelines 2.0
description: Pipelines un poco más complejos para manejo multimedia
date: 2021-07-08
aliases: ["bash","pipe","pipelines","linux","unix"]
tags:
  - linux
  - scripting
author: Fredrare
---

## Introducción
Ya vimos cómo funcionan los pipelines en este post, así que ahora podemos usar pipes de una manera un poco más útil e interesante. Si no lo has leído y no sientes comodidad al usarlos, te sugiero hacerlo.

Ahora veremos pipes más complejos que manejan más que texto plano, te permiten manipular el portapapeles, hacer fetch a algunas apis y, además, postear tus resultados en internet. Recuerda que solo son ideas, así que puedes usarlas como base para tus propios scripts, solo tomarlas como referencia para algo nada similar, copiarlas o de plano ignorarlas por completo jajajaja

## Ideas
### Capturar pantalla al portapapeles
Me gusta usar [`maim`](https://github.com/naelstrof/maim) para poder tomar capturas de pantalla, porque la salida del comando va directamente a `stdout`. Así, puedo pasársela a `xclip`, una aplicación que me permite colocar el resultado en el portapapeles directamente como una imagen si estoy usando [X](https://en.wikipedia.org/wiki/X_Window_System). Entonces, podría hacer realidad lo que propongo con el siguiente comando.

```bash
maim | xclip -selection clipboard -t image/png
```

Primero se ejecuta `maim` y el resultado va directamente a `stdout`. Como estoy usando un pipe, ese resultado no se imprime en la pantalla, sino que pasa como input del comando `xclip`. Los parámetros que he configurado son `-selection clipboard` para que la salida vaya al portapapeles y `-t image/png` para que el resultado final no sea el contenido de la imagen como texto, sino que se reconozca como imagen.

Si me interesara, además de esto, guardar la captura de pantalla como un archivo, podría usar el comando `tee` de la siguiente manera.

```bash
maim | tee /home/anti/Pictures/Screenshots/ss.$(date +'%d-%m-%y.%H-%M-%S').png | xclip -selection clipboard -t image/png
```

Aquí podemos ver una sintaxis un poco más compleja, porque he usado una [sustitución](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html). Lo que sucede es lo siguiente:

1. Se ejecuta `maim` y la salida pasa como entrada al siguiente comando: `tee`.
2. Se ejecuta `tee` y la entrada se maneja de la siguiente manera:
   1. Se guarda en archivo llamado `/home/anti/Pictures/Screenshots/ss.$(date +'%d-%m-%y.%H-%M-%S').png`, donde ocurre una sustitución y se reemplaza `$(date +'%d-%m-%y.%H-%M-%S')` por una fecha como `31-02-21.21-42-00`. Entonces, el archivo en realidad se llamaría `ss.31-02-21.21-42-00.png`.
   2. Pasa como entrada para el siguiente comando del *pipeline*: `xclip`.
3. La salida de `tee` es exactamente la misma que en el ejemplo anterior, como si fuera exactamente el resultado de `maim`.

### Publicar la salida de un comando en internet
Supongamos que quiero publicar el código de un pequeño script que estoy haciendo en internet para dárselo a alguien más. Una forma muy común sería usar pastebin, pero gastaría tiempo entrando a la página y poniendo los detalles de lo que quiero publicar.

Por eso existe una página llamada [ix.io](http://ix.io/). Si entramos, podremos ver que tienen un manual bastante útil y podríamos hacer algo como lo siguiente.

```bash
cat test.py | curl -F 'f:1=<-' ix.io
```

A mí me gusta tener el segundo comando de ese *pipeline* en un alias llamado `post`, de forma que se vuelve bastante intuitivo usar el comando luego. El código sería el siguiente:

```bash
# Este es mi alias
alias post="curl -F 'f:1=<-' ix.io"

# Y lo uso de esta manera
cat test.py | post
```

### Usar [`awk`](https://en.wikipedia.org/wiki/AWK)
`awk` es una deidad. Se trata de un lenguaje muy potente de scripting que también viene incluido como comando en sistemas tipo Unix y permite manipular *strings*.

Veamos cómo lo usaría para saber qué servicios mantienen puertos abiertos en mi pc.

```bash
# Con este comando
sudo netstat -tulpn
# Obtengo esta salida
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      481/sshd: /usr/bin/ 
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      480/cupsd           
tcp6       0      0 :::22                   :::*                    LISTEN      481/sshd: /usr/bin/ 
tcp6       0      0 ::1:631                 :::*                    LISTEN      480/cupsd           

# No necesito las primeras dos líneas
# Así que uso tail e imprimo desde la tercera
sudo netstat -tulpn | tail -n +3
# Y obtengo esta nueva salida
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      481/sshd: /usr/bin/ 
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      480/cupsd           
tcp6       0      0 :::22                   :::*                    LISTEN      481/sshd: /usr/bin/ 
tcp6       0      0 ::1:631                 :::*                    LISTEN      480/cupsd           

# Ahora solo quiero mostrar las columnas 4 y 7
sudo netstat -tulpn | tail -n +3 | awk '{ print $4, $7}'
# Y obtengo esto
0.0.0.0:22 481/sshd:
127.0.0.1:631 480/cupsd
:::22 481/sshd:
::1:631 480/cupsd
```

Con este comando, puedo determinar qué puertos tengo abiertos y el PID de cada uno, por si los necesito matar o analizar más a fondo. Aquí podría leer el output de la siguiente manera:

> Tengo 4 puertos abiertos.
>
> El puerto `22` lo usa `ssh` para permitirme acceder a la terminal desde otro dispositivo y lo estoy usando con IPv4 e IPv6. Puedo acceder por este puerto desde cualquier host.
> 
> El puerto `631` lo usa `cups` para mantener mi servicio de impresiones. También funciona en IPv4 e IPv6. Solo puedo acceder por aquí desde localhost.

Se puede notar fácilmente que con un par de pruebas se puede lograr el *output* deseado. Solo es necesario saber qué herramientas existen para lo que queremos hacer y cómo usarlas.

### Hacer tu vida más fácil con aliases
Hemos visto que los *pipelines* pueden ser muy útiles porque son modulares y podemos combinar comandos a nuestro gusto. No obstante, si los comandos son muy grandes, puede ser una tarea algo aburrida usarlos en cadena. Para liberarte de ese problema, puedes usar aliases, como en el caso anterior.

Claramente, `post` es mucho más fácil de leer que `curl -F 'f:1=<-' ix.io`. Así que veamos cómo podemos acortar nuestros comandos de arriba para hacerlos más sencillos. **No es necesario usar aliases para todo**. Yo prefiero solo usarlos para comandos que uso constantemente en muchos *pipes*.

#### Alias de copy paste
macOS viene con dos comandos de manejo de portapapeles por defecto: `pbcopy` y `pbpaste`. Ambos se usan muy ampliamente en *pipelines*, pero no vienen incluidos fuera de macOS. Pero ***fear no more***, porque podemos replicar ese mismo comportamiento en X. Solo necesitamos crear dos aliases con `xclip`, como en el primer ejemplo de este post.

```bash
# Creo estos aliases así
alias pbcopy='xclip -selection clipboard'
alias pbpaste='xclip -selection clipboard -o'

# Copio texto de esta manera
ls | pbcopy

# Y lo puedo mostrar de esta otra
pbpaste
```

Podemos ver que es mucho más sencillo copiar y pegar con esos aliases en lugar de escribir el comando entero. Además, esto me permite usar mis scripts tanto en mi distro Linux como en macOS, así que gano en tiempo y portabilidad. **10/10**

También puedo usar `pbcopy` en el primer ejemplo que mostré. Solo tendría que modificarlo un poco:

```bash
# Este es el comando original
maim | xclip -selection clipboard -t image/png

# Y este sería el acortado
maim | pbcopy -t image/png
```

Así se entiende más el comando y se puede ver que un alias no le quita la capacidad de aceptar más parámetros al comando original.

#### Alias para decodificar Base64

Si te dedicas al hacking, probablemente veas texto en Base64 muy seguido. Yo uso este alias para decodificar el texto automáticamente. Aquí, partiré de que ya tengo el alias de `pbpaste` creado.

```bash
# Este sería mi comando usual
pbpaste | base64 -d

# Entonces creo un alias
alias decode='pbpaste | base64 -d'

# Y podré decodificar el portapapeles así
decode
```

Con lo anterior, puedo hacer más productivo mi trabajo. Como siempre tengo la terminal abierta, me cae de maravilla tener ese alias listo para usar.

Con eso termina este post. Recuerda que las combinaciones no tienen límite. Todo depende de qué tanto sepas usar la terminal y los comandos que quieres combinar en un *pipeline*.