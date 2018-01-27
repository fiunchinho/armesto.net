---
author: fiunchinho
categories:
- Code
date: 2013-11-05T18:05:53Z
dsq_thread_id:
- "2650334585"
guid: http://armesto.net/?p=18
id: 18
tags:
- unit testing
title: Inspección continua de la calidad de nuestro código
url: /inspeccion-continua-de-la-calidad-de-nuestro-codigo/
---

Una de las formas que tenemos para poder mejorar el código de nuestro proyecto es someterlo a una inspección continua y constante. Lo primero que podemos hacer es lanzar nuestros tests tras cada push al repositorio, de forma que sabremos en todo momento si hay algo que no funciona bien.
  
También podemos analizar la complejidad del código y comprobar si estamos incrementándola o disminuyéndola.

Hasta ahora, todas estas tareas eran posibles solamente teniendo un servidor de integración continua configurado, siendo Jenkins el más famoso. En él podemos ejecutar todas estas tareas tras cada push. Recursos como <http://jenkins-php.org/> te ayudarán a configurar este versátil servidor hecho en Java.

La buena noticia es que cada vez hay más herramientas online que te permiten hacer estas tareas sin necesidad de configurar practicamente nada. Hoy veremos como lanzar tests y analizar la calidad de nuestro código sin utilizar un servidor de integración continua.

<!--more-->

## Ejecutando nuestros tests

Estos dos últimos años han sido la explosión en popularidad de <a title="TravisCI" href="https://travis-ci.org/" target="_blank">TravisCI</a>. TravisCI es un servicio que te permitirá lanzar tu suite de tests tras cada push. ¿Cómo? Pues cada vez que se envía código al repositorio, TravisCI levantará una máquina virtual y clonará tu código en ella. Una vez todo esté allí, **ejecutará los tests y te dirá si algo se ha roto** o todo sigue correcto.

También te permite ejecutar comandos antes de lanzar los tests, por si necesitas instalar dependencias de otras librerías o incluso programas en la máquina virtual.

Para que todo esto funcione, necesitas tener un archivo de configuración en tu repositorio que le diga a TravisCI qué es lo que tiene que hacer. Este archivo se llama _.travis.yml_ y un ejemplo podría ser este

<pre><code data-lang="yaml">language: php
php:
	- 5.3
	- 5.4
	- 5.5
before_install:
	- curl -s http://getcomposer.org/installer | php
	- php composer.phar --dev install</code></pre>

Con esta configuración le estamos diciendo a TravisCI que nuestro lenguaje es PHP, que queremos que lance los tests en 3 máquinas virtuales distintas, una con la versión 5.3 de PHP, otra con la 5.4 y otra con la 5.5. Además, antes de poder lanzar los tests, le decimos que instale las dependencias de nuestro proyecto mediante composer.

El hecho de saber si algo se ha roto con un commit es de vital importancia, sobretodo en proyectos con muchos programadores distintos, como los proyectos Open Source. Para esto, cada vez que hay un cambio de estado (alguien rompe los tests y pasamos de estable a roto, o viceversa), TravisCI nos enviará un email para tenernos informados.

Para que todo el mundo tenga visibilidad del estado actual, también pone a nuestra disposición una insignia que indica si los tests están en rojo o en verde. Podemos colocar esta insignia en el README.md de nuestro proyecto, y cualquiera que entre verá si hay algo roto.

[<img class="alignnone size-full wp-image-55" alt="PHPUnit badge" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_025.png" width="801" height="112" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_025.png 801w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_025-300x41.png 300w" sizes="(max-width: 801px) 100vw, 801px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_025.png)

Por último, recuerda que por defecto TravisCI ejecutará **PHPUnit** para lanzar los tests, utilizando la configuración del fichero _phpunit.xml.dist_ para saber donde están los tests y cómo lanzarlos. Si quieres cambiar algo de lo que TravisCI hace por defecto, <a title="TravisCI Docs" href="http://about.travis-ci.org/docs/user/languages/php/" target="_blank">la documentación</a> está bastante bien.

## Analizando la calidad del código

Existen <a title="PHP QA Tools" href="http://phpqatools.org/" target="_blank">un montón de herramientas de análisis estático de código PHP </a>que nos dan todo tipo de información, desde si hay mucho copy/paste, hasta si nuestro código es poco abstracto. Son herramientas que puedes lanzar cuando quieras desde la consola, pero que como veíamos al principio del post, sería interesante lanzar cada vez que se envía código al repositorio. Si no queremos utilizar nuestro servidor de integración continua propio, una alternativa que tenemos es <a title="Scrutinizer" href="https://scrutinizer-ci.com/" target="_blank">Scrutinizer</a>.

Su funcionamiento es muy parecido al de TravisCI, pero en vez de lanzar tu suite de tests, lanzará todas las <a title="Herramientas de análisis estático de código" href="https://scrutinizer-ci.com/docs/tools/php/" target="_blank">herramientas de análisis estático de código</a> que hayas configurado. Para la configuración, tan solo debes modificar en la web de Scrutinizer el archivo de configuración, o tener un archivo _.scrutinizer.yml_ en tu repositorio. En la web puedes elegir una configuración global para todos tus proyectos, o tener una distinta para cada uno.

El mío tiene esta pinta

<pre><code data-lang="yaml">filter:
 excluded_paths: [vendor/*, app/*, web/*]

tools:
 php_cpd: true
 php_pdepend:
     excluded_dirs: [vendor]
 php_mess_detector: true
 php_analyzer: true
 php_loc:
     command: phploc
     excluded_dirs: [vendor]
     enabled: true</code></pre>

No tengo seleccionadas todas las herramientas, pero para hacer una prueba es suficiente con estas.

Una vez elegida tu configuración, puedes lanzar el análisis directamente eligiendo “_Schedule Inspection_” en la parte superior derecha. De todas formas, salvo que lo desactives en las opciones, cada vez que código nuevo se envíe al repositorio, se realizará una inspección de tu código.

### Informes

Lo bueno es que después de ejecutarse, Scrutinizer genera unos informes muy bonitos con todo tipo de información útil.

[<img class="alignnone size-large wp-image-56" alt="Scrutinizer" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_023-1024x651.png" width="620" height="394" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_023-1024x651.png 1024w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_023-300x190.png 300w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_023.png 1099w" sizes="(max-width: 620px) 100vw, 620px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_023.png)

Arriba de todo podemos ver **la nota final de nuestro código**, 9.18 en la imagen. Esta nota es calculada teniendo en cuenta el resultado de todas las inspecciones que tenemos configuradas. También nos ofrece un histórico con nuestra nota, o los incidentes (issues) de nuestro código que todavía tenemos pendientes por arreglar.

[<img class="alignnone size-large wp-image-58" alt="Scrutinizer" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_024-1024x410.png" width="620" height="248" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_024-1024x410.png 1024w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_024-300x120.png 300w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_024.png 1093w" sizes="(max-width: 620px) 100vw, 620px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_024.png)

Por último, igual que TravisCI, nos ofrece la posibilidad de poner una insignia con nuestra nota en el README.md del repositorio, para que todo el mundo vea a simple vista la calidad del código.

[<img class="alignnone size-full wp-image-60" alt="Scrutinizr badge" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_026.png" width="809" height="236" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_026.png 809w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_026-300x87.png 300w" sizes="(max-width: 809px) 100vw, 809px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_026.png)

## Lo tienes fácil

Antigüamente tenías que pelearte con un servidor tipo Jenkins si querías tener funcionalidades de integración continua, pero cada vez salen más herramientas online a la luz que nos ayudan con la calidad de nuestro código. Todavía no tenemos todo lo que un Jenkins nos puede ofrecer, pero con estos dos simples servicios que hemos visto hoy, ya tendríamos mucho ganado.
