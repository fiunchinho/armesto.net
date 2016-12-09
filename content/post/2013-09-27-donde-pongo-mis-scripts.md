---
author: fiunchinho
categories:
- Linux
date: 2013-09-27T18:00:27Z
dsq_thread_id:
- "2648573035"
guid: http://armesto.net/?p=12
id: 12
title: ¿Donde pongo mis scripts?
url: /donde-pongo-mis-scripts/
---

Vale, te has creado un _script_ en <a title="Unix" href="http://www.unix.org/" target="_blank">Unix</a> para automatizar una tarea, pero después de programar durante horas, ahora viene lo más difícil de todo… donde lo pones?

Podrías ponerlo en la carpeta **/bin**… o en **/usr/bin**… incluso en **/home/../bin**. Para qué tantas carpetas de binarios?

Vamos a intentar explicar para qué sirve cada una.

<!--more-->

## Jerarquía

Los directorios en Unix siguen una jerarquía que, aunque no lo parezca, intenta tener un sentido. Los más importantes son:

  * **/boot**: Cosas necesarias para el arranque.
  * **/lib**: Librerías del sistema.
  * **/etc**: Configuraciones del sistema.
  * **/proc**: Información sobre los procesos que están ejecutándose.
  * **/dev**: Aquí encontramos archivos representando dispositivos conectados, como los discos duros o dispositivos usb.
  * **/var**: Archivos variables, como los logs.
  * **/tmp**: Directorio temporal. La información que hay aquí se perderá cuando reiniciemos.
  * **/bin**: Ejecutables indispensables que se usan antes de que se monte la carpeta _/usr_, o cuando se inicia en modo “_single-user_“. Aquí hay programas como _cat_ o _ls_.
  * **/sbin**: Lo mismo que antes, pero con ejecutables para administradores del sistema. Un usuario normal no tiene permisos para ejecutar estos binarios.
  * **/usr**: Contiene ejecutables (bin), librerías (lib) o documentación (doc). 
      * **/usr/bin**: Lo mismo que /bin, pero para ejecutables de propósito general.
      * **/usr/sbin**: Igual que /usr/bin, pero para administradores del sistema.
      * **/usr/local**: Contiene programas que no son administrados por paquetes del sistema o el gestor de paquetes
  * **/home**: Todo lo relacionado con los usuarios que utilizarán el sistema 
      * **/home/bin**: Ejecutables creados por el usuario, para tareas que no son del sistema

Cuando crees un programa propio para _parsear_ un csv con información, ponlo en tu **/home**, y cuando instales algún programa, ponlo en** /usr/local/bin**. Recuerda también que tanto **/bin**, como **/usr/bin** y **/usr/local/bin** están incluídos en el **_$PATH_** del sistema, así que cuando quieras ejecutarlos no hace falta que pongas su ruta completa. Esto no es así para tu **/home**, que deberás o bien añadirlo al _**$PATH**_, o crear un _symlink_ de tu programa a **/usr/local/bin** por ejemplo.

<pre># ln -s /home/fiunchinho/my_program /usr/local/bin/</pre>