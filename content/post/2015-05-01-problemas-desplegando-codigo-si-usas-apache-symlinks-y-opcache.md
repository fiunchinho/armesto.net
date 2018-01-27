---
author: fiunchinho
categories:
- Code
- Linux
date: 2015-05-01T12:14:32Z
dsq_thread_id:
- "3727662731"
guid: https://blog.armesto.net/?p=123
id: 123
tags:
- apache
- deployment
- php
title: Problemas desplegando código si usas Apache, symlinks y opcache
url: /problemas-desplegando-codigo-si-usas-apache-symlinks-y-opcache/
thumbnail: "images/apache.png"
---

Muchas de las soluciones disponibles en el mercado para desplegar aplicaciones se basan en el uso de enlaces simbólicos (o symlinks) para activar la última versión de código en el servidor.

Simplificando mucho, podríamos decir que un flujo habitual a la hora de desplegar sería el siguiente:

  * Ejecutamos comando para iniciar el proceso de despliegue de código nuevo.
  * Se descarga el código del repositorio y se construye la aplicación. Esto suele significar instalar dependencias, generar ficheros, etc.
  * Se mueve el resultado del paso anterior al servidor y se pone en una carpeta nueva.
  * La carpeta a la que apunta el _document root_ de nuestro servidor web es en realidad un enlace simbólico a otra carpeta que contiene código en la versión anterior. Por tanto solo nos queda cambiar ese enlace simbólico para que apunte a la nueva que acabamos de crear.

Como el cambio de enlace simbólico es practicamente instantáneo, conseguimos reducir la ventana de tiempo en la que el servidor está en un estado inconsistente, por ejemplo, porque todavía no se hayan terminado de copiar ficheros. Mientras se están subiendo la versión nueva, seguimos sirviendo la versión vieja, sin dejar de dar servicio. Y solo cuando la nueva está lista, hacemos el cambio de forma casi instantánea.

<!--more-->

## Problemas con este enfoque

Esta manera de desplegar, que parece sencilla y perfecta, tiene algunas complicaciones. Con la que la gente más suele pelearse es con el hecho de que a pesar de haber desplegado una versión nueva en el servidor, a veces siguen viendo la versión vieja, debido a la extensión <a href="https://www.google.es/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0CCUQFjAA&url=http%3A%2F%2Fphp.net%2Fmanual%2Fes%2Fbook.opcache.php&ei=HlFDVZfPI8T2Us6sgLAI&usg=AFQjCNF9sKlRWdBbTEBKa1M2w25s5TBQGw&sig2=6xMwOb3cZagbFnheUWkDTQ&bvm=bv.92291466,d.d24" target="_blank">Opcache</a> (antiguo APC) que guarda la compilación del código PHP interpretado en memoria. Por tanto, aunque la versión nueva ya está activa, PHP sigue tirando de esta caché para no tener que leer del disco y volver a compilar código PHP, así que se sirve el código de la versión vieja hasta que caduque esta caché (si es que lo tenemos configurado para que caduque), o hasta que reiniciemos el servidor dejando de dar servicio mientras dure el reinicio del servidor.
  
Antes de ver cómo solventarlo, vamos a ver un par de detalles interesantes.

## Te presento a tu nueva amiga realpath_cache

Cada vez que utilizas una ruta del sistema de archivos, por ejemplo porque vas a hacer un require/include de ese archivo, o porque vas lees/escribir en esa ruta, **el sistema tiene que resolver esa ruta**: saber donde es exactamente, si es un directorio o un archivo, etc.

Para mejorar su rendimiento y minimizar lecturas de disco, PHP utiliza una caché interna donde guarda información sobre el sistema de archivos. No estoy hablando de Opcache, sino de otra caché llamada <a href="https://php.net/manual/es/ini.core.php#ini.realpath-cache-size" target="_blank">realpath_cache</a>. Si intentas resolver la misma ruta dos veces seguidas, solo en la primera PHP le pedirá información al lentísimo sistema de archivos: la segunda se leerá directamente de la caché, mejorando mucho el rendimiento.

Esto es bueno, ¿no?

Sí, claro. El tema es que, como con todas las cachés del mundo, el problema viene a la hora de invalidar la caché y decirle que queremos utilizar contenido nuevo.

> There are only two hard things in Computer Science: cache invalidation and naming things.

Cuando desplegamos código nuevo, el contenido de la ruta de nuestro _Document Root_ cambia. Aunque en nuestro código siempre usemos la misma ruta para hacer algo, piensa por ejemplo la ruta relativa que usas para incluir el autoload, despues de cada deploy, **esa ruta se resuelve a un lugar distinto en el disco**, porque el enlace simbólico apunta a otro sitio. Esto evitará que veamos la versión nueva del código desplegado, hasta que esta caché no caduque o hagamos algo al respecto.

## Cómo funciona Apache

A diferencia de Opcache, que se guarda en memoria compartida por todos los procesos, la realpath_cache es local para cada proceso del sistema. Este detalle es importante porque si utilizas Apache _prefork_ para servir tu aplicación, cuando inicias Apache, este crea varios procesos hijos, tantos como le hayas configurado. Cada proceso hijo creado servirá X número de peticiones en su vida (esto también es configurable), y una vez que ha cumplido su deber, Apache lo matará y lo reemplazará con otro proceso hijo, poco a poco renovando todos los procesos que sirven páginas.
  
Es decir, **no podemos preveer exactamente cuando los procesos dejarán de existir**. Sumado a que la realpath_cache es local a cada proceso,** la nueva versión que acabamos de desplegar se irá sirviendo aleatoriamente**, dependiendo de qué proceso de Apache te haya asignado el servidor.

Como dijimos antes, podríamos solventarlo haciendo un reinicio de Apache despues de cada despliegue, pero dejaríamos de servir páginas el tiempo que tardase en reiniciar.

> Mal, mal, mal, verdadera mal, por no deci borchenoso

## Reiniciando elegantemente

Pero tranquilo, no sufras, hay solución. Apache nos ofrece una variante al reinicio, llamada _graceful restart_. Esta variante, en vez de matar al proceso de Apache y todos sus hijos para reiniciarlo, lo que hace es que el proceso padre revisa a los procesos hijos de forma que:

  * Si no están haciendo nada, los sustituye por un proceso nuevo.
  * Si está sirviendo una petición en este momento, cuando termine lo sustituye por un proceso nuevo.

Como dijimos que realpath_cache era local a cada proceso, cuando Apache levanta un nuevo proceso hijo **la realpath_cache está vacía para ese proceso y las rutas se resolverán al código nuevo que acabamos de desplegar**. Todo esto sin dejar de dar servicio, porque siempre hay procesos sirviendo páginas.

## _Pesao_. Que yo venía aquí a solventar el problema de Opcache

Cierto. Opcache no es más que un diccionario (piensa en un array PHP), en el que cada _key_ es la ruta del fichero compilado, y el _value_ es el resultado de esa compilación. Cuando se va a ejecutar un fichero PHP, se ve si la ruta de ese archivo ya es una de las _keys_ en el diccionario, si ya lo está significa que ya lo hemos compilado antes, y se utiliza directamente el _value_. Si no, se compila y se guarda en el diccionario.

El &#8216;_final plot twist&#8217;_ de todo esto es que **Opcache utiliza realpath_cache internamente para resolver la ruta de los ficheros**. Por tanto, si hacemos un _graceful restart_ después de cada despliegue, la ruta del archivo habrá cambiado, resuelve a una carpeta distinta, así que **será como un fichero totalmente nuevo para Opcache y volverá a compilarlo, haciendo que sirvamos la versión nueva**.

<img class="alignnone" src="https://s-media-cache-ak0.pinimg.com/originals/ce/9c/94/ce9c949d6c73dbfb889f6036bac022dd.jpg" alt="Mind Blown" width="480" height="360" />

## Conclusión

Lo que hemos visto hoy es tan solo uno de los posibles problemas a la hora de desplegar código. Otro problema, por ejemplo, sería el que se produce cuando iniciamos un despliegue, un visitante entra en la web, justo después se cambia el enlace simbólico y estamos sirviendo archivos estáticos como javascript o css. Es posible que algunos archivos hayan sido de la versión vieja, y otros de la versión nueva, llevando a posibles inconsistencias. <a href="https://codeascraft.com/2013/07/01/atomic-deploys-at-etsy/" target="_blank">Hay módulos de apache que intentan solventar este tipo de problemas</a>.

El despliegue de código es un tema complicado y muy interesante. Últimamente está avanzando mucho con conceptos como servidores inmutables y los contenedores, pero eso ya es tema de otro post.
