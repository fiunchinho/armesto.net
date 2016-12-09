---
author: fiunchinho
categories:
- Linux
date: 2013-09-20T18:01:09Z
dsq_thread_id:
- "2652012329"
guid: http://armesto.net/?p=14
id: 14
tags:
- unix
title: Evitar borrar todo tu disco con rm -rf
url: /evitar-borrar-todo-tu-disco-con-rm-rf/
---

A quién no le ha pasado, verdad? Quieres borrar el contenido de una carpeta con el comando <a title="Comando rm" href="http://linux.about.com/od/commands/l/blcmdl1_rm.htm" target="_blank"><em>rm</em></a>, pero sin querer pones el asterisco donde no debías y **_ZASCA! _**: todo tu disco duro se ha borrado por completo.

A veces no es tan dramático. A veces solo te equivocas en el directorio que querías borrar. Estabas un nivel más arriba o más abajo del que pensabas y adiós proyecto en el que llevabas toda la tarde trabajando y del que todavía no habías hecho _commit_.

<!--more-->

## Posible solución

Una posible solución sería utilizar la opción _-i_ del comando rm, creando un alias en tu _/.bashrc_ para que siempre que ejecutemos _rm_ sea con ese _flag_ activado. Esto hace que nos pida confirmación siempre que lo usamos, para tener un “_double check_” antes de liarla parda.

    alias rm='rm -i'

El problema que le veo a esto, es que muchas veces estamos borrando solo un fichero, y es un poco incómodo tener que andar confirmando siempre.

Además, da una falsa sensación de seguridad. Cuando te pida confirmación, tanto tú como yo sabemos que en ese momento no te vas a dar cuenta del error que estás a punto de cometer.

## Mejor todavía

Hay una solución mejor. Otro _flag_ que trae este comando es el _-I_. Como su definición indica, nos pedirá confirmación solo cuando estemos a punto de borrar más de 3 ficheros. Si borramos menos de eso, no nos pedirá confirmación.

    alias rm='rm -I'

Está claro que tarde o temprano acabaremos liándola, pero con esta opción quizá podamos retrasar ese día unos añitos más.