---
title: SSH más seguro
description: A veces, no poner una contraseña puede ser más seguro. Aprende a usar la infraestructura de llave pública con ssh.
date: 2021-09-29
aliases: ["ssh","sysadmin","seguridad","pki","id_rsa"]
tags:
  - seguridad
  - sysadmin
  - consejos
author: Fredrare
---

## Introducción
`ssh` nos permite establecer un túnel de comunicación segura con un equipo remoto para muchas cosas, entre las que se encuentran usar una `shell`. Para poder ingresar a una máquina usando este protocolo, usualmente nos enfrentaremos a un escenario como este:

```txt
❯ ssh root@bolt.htb
root@bolt.htb's password: 
```

El comando ejecutado ha sido `ssh root@bolt.htb`, que básicamente le dice al comando que me quiero conectar al *host* `bolt.htb` con el usuario `root`.

Justo después de hacerlo, veo que obtengo una respuesta que más bien es una pregunta: ¿Cuál es la contraseña?

Entonces, ahora me enfrento a que alguna persona con malas intenciones, pueda hacer un ataque de fuerza bruta intentando millones de contraseñas hasta dar con la correcta. Así que... ¿cómo podemos evitar eso? Conozcamos a la infraestructura de llave pública.

## PKI (*Public Key Infrastructure*)
Sería bueno empezar por una descripción sencilla de qué es esto. Afortunadamente, he hecho [un video al respecto](https://youtu.be/TD05p7ncsp0) y verlo ayudaría a comprender mejor el concepto.

Para poder aplicar este mismo concepto a `ssh`, usaremos dos tres elementos principales. Una llave pública, una privada y una lista de llaves admitidas.

Como se explica en ese video, una llave pública es capaz de verificar si un mensaje ha sido firmado por una llave privada. Y, vaya sorpresa, ese es justo el mecanismo que usaremos para poder resolver nuestro problema

## Directorio `.ssh`
Aquí se guarda, la configuración y algunos archivos importantes para que `ssh` funcione correctamente. Hoy usaremos tres de estos archivos.

Si el directorio no ha sido creado para tu usuario (se debe encontrar en la carpeta `home` o `~`), puedes generarlo junto con tu par de llaves con el comando

```bash
ssh-keygen
```
### Fichero `id_rsa`
En este archivo se encuentra nuestra llave privada. Es importante nunca compartirla con nadie, porque podría conllevar una suplantación de identidad.

### Fichero `id_rsa.pub`

Aquí podemos ver a la pareja de `id_rsa`, pues se trata de nuestra llave pública. Este archivo puede ser compartido con cualquier usuario sin riesgo de comprometer nuestro sistema. Por eso se llama llave **pública**.

### Fichero `authorized_keys`
Dentro de este archivo encontramos llaves públicas con autorización para acceder al sistema.

## Configuración
Como el archivo `authorized_keys` tiene una lista de llaves públicas admitidas por la cuenta, tendremos una estructura de diretorios como la siguiente

```txt
Localhost             Remotehost
~/.ssh                ~/.ssh
  \_ id_rsa             \_ authorized_keys
  \_ id_rsa.pub
```

1. En localhost, solo será necesario crear el par de llaves con el comando `ssh-keygen`. Esto generará tanto `id_rsa` como `id_rsa.pub`.
2. En la máquina remota, crearemos un directorio `.ssh` en la carpeta del usuario al que se quiere acceder sin contraseña y, dentro, tendremos un archivo llamado `authorized_keys`.
3. Dentro de `authorized_keys` colocaremos las llaves públicas de las máquinas desde las que nos queremos conectar. Para hacerlo, solo necesitamos copiar el contenido del `id_rsa.pub` que se creó automáticamente y pegarlo dentro de `authorized_keys`. Si ya existe ese archivo y tiene contenido, solo hace falta agregar una línea más y colocarlo ahí.

Cada vez que queramos conectarnos desde una máquina nueva, solo hará falta colocar la llave pública de ese equipo en el archivo `authorized_keys`.

## ¿Cómo funciona?
Como la máquina remota tiene la llave pública de la local, el problema está resuelto gracias a la **criptografía asimétrica**.

Cuando se establece la conexión entre ambas computadoras, la remota puede saber si la local tiene la llave privada que le corresponde a alguna de las llaves públicas de `authorized_keys`. ¿Cómo? La llave pública solo hace par con una llave privada, de manera que el *host* que tenga un `id_rsa` que haga juego con un `id_rsa.pub` dentro de `authorized_keys` tiene que haber sido explícitamente ingresado en ese archivo. Por lo tanto, se permite el ingreso sin contraseña y el usuario vería una respuesta del comando como esta.

```bash
❯ ssh anti@10.10.10.101
anti@debian: ~ $ 
```

Ahora no necesitamos poner una contraseña para ingresar al *host*. Solo con el fichero `id_rsa` dentro del directorio `.ssh` en nuestro `home` será suficiente para poder abrir una *shell* en el *host* remoto.