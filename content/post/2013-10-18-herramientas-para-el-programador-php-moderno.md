---
author: fiunchinho
categories:
- Code
date: 2013-10-18T23:47:55Z
dsq_thread_id:
- "2648474400"
guid: https://blog.armesto.net/?p=42
id: 42
tags:
- composer
- php
- vagrant
title: Herramientas para el programador PHP moderno
url: /herramientas-para-el-programador-php-moderno/
---

<div>
  <p>
    <span style="font-size: 14px; line-height: 1.5em;">La comunidad de PHP ha evolucionado muchísimo en los últimos años, no pareciéndose en nada a las versiones anteriores. No solo ha cambiado mucho, si no que cada vez cambia más frecuentemente. Y cuando hablo de comunidad, me refiero tanto al lenguaje, como a las personas que lo utilizan, así como a las herramientas que nacen alrededor.</span>
  </p>
</div>

<div>
  <p>
    Hoy vengo a hablaros precisamente de herramientas. No puede ser que programes PHP con las mismas herramientas que utilizabas hace 3 años. Haber actualizado tu IDE a la última versión es un comienzo, pero no es suficiente. Estás perdiendo la posibilidad de trabajar más cómodamente y ser más productivo, pudiéndote centrar en lo que realmente importa: crear cosas.
  </p>
  
  <p>
    <!--more-->
  </p>
  
  <p>
    De entre todas las herramientas que han nacido en los últimos años, voy a hacer hincapié en las que para mí son las más útiles, y ya no podría vivir sin ellas.
  </p>
  
  <p>
    Estas herramientas son:
  </p>
  
  <h2>
    Boris
  </h2>
  
  <p>
    Cuando empecé a aprender Python, una de las cosas que más me sorprendió fue que para dar los primeros pasos con el lenguaje, no había que crear nuevos ficheros y ejecutarlos. Sencillamente ibas a un intérprete por línea de comandos y programabas directamente allí. Adiós complicaciones de cómo ejecutar lo que acabas de escribir en otro fichero, o de como incluir el fichero A en el fichero B. Todo eso fuera para poder centrarte en lo que realmente quieres: aprender el lenguaje.
  </p>
  
  <p>
    Este intérprete se conoce comúnmente como un <a title="REPL" href="http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop" target="_blank">REPL: Read-Eval-Print-Loop </a>. PHP trae por defecto con un <a title="PHP Interactive Mode" href="http://www.php.net/manual/en/features.commandline.interactive.php" target="_blank">modo interactivo </a>para poder lanzar comandos, que puedes utilizar así
  </p>
  
  <pre>php -aInteractive shellphp &gt; 1 + 3;php &gt; var_dump( 1 + 3 );int(4)php &gt; function plusTwo( $number ){php {php { return $number + 2;php { }php &gt; echo plusTwo( 3 );5php &gt;</pre>
  
  <p>
    pero es bastante limitado. No printa automáticamente el resultado de las expresiones ejecutadas, por tanto tienes que manualmente hacer <em>echo’s</em> o<em>var_dumps</em>. Además, si se produce un fatal error, el modo interactivo terminará y perderás el estado actual. Esto no ocurriría con <a title="Boris" href="https://github.com/d11wtq/boris" target="_blank">Boris </a>. Os dejo el gif con la demo de <a title="Boris" href="https://github.com/d11wtq/boris" target="_blank">Boris </a>, porque una imagen vale más que mil palabras (sobretodo si es animada):
  </p>
  
  <p>
    &nbsp;
  </p>
  
  <div>
    <div>
      <img alt="Boris Demo" src="https://mail.google.com/mail/u/0/?ui=2&ik=10c6244d6d&view=att&th=14540c648bdc9adb&attid=0.2&disp=emb&zw&atsh=1" width="684" height="476" />
    </div>
  </div>
  
  <p>
    &nbsp;
  </p>
  
  <h2>
    LadyBug
  </h2>
  
  <p>
    Calidad que sale desde nuestro país. <a title="Raúl Fraile" href="https://twitter.com/raulfraile" target="_blank">Raúl Fraile </a>se ha currado un sustituto para el <em>var_dump()</em> nativo de <strong>PHP</strong> que te dejará ojiplático. Se llama <a title="LadyBug" href="https://github.com/raulfraile/ladybug" target="_blank">LadyBug </a>y después de verlo te preguntarás cómo has sido capaz de interpretar el output del <em>var_dump</em> de <strong>PHP</strong> sin quedarte ciego. Para utilizarlo solo tienes que instalarlo via Composer, y utilizar la función <em>ladybug_dump()</em> en vez de <em>var_dump()</em> cuando quieras mostrar el valor de algo por pantalla. En su <a title="LadyBug" href="https://github.com/raulfraile/ladybug" target="_blank">página de Github </a>hay bastantes ejemplos de cual sería el output por pantalla, pero quiero poner uno aquí mismo:
  </p>
  
  <p>
    &nbsp;
  </p>
  
  <div>
    <div>
      <img alt="LadyBug" src="https://mail.google.com/mail/u/0/?ui=2&ik=10c6244d6d&view=att&th=14540c648bdc9adb&attid=0.1&disp=emb&zw&atsh=1" width="710" height="585" />
    </div>
  </div>
  
  <p>
    &nbsp;
  </p>
  
  <p>
    Eso es el output cuando quieres pintar un objeto por pantalla y ver qué contiene. Fíjate que además de ponerte el contenido de las propiedades del objeto, como hace <em>var_dump()</em>, te muestra información de la clase del objeto, como donde está definida, constantes de clase e incluso los métodos y <strong>su visibilidad mediante unos iconos</strong>. Todo esto está genial, pero queda a la sombra de lo que me hubiese ahorrado horas de búsqueda en mis años iniciales con <strong>PHP</strong>: te dice en qué línea has llamado a la función <em>ladybug_dump()</em>. ¿Quién no se ha pasado un buen rato buscando dónde habías puesto el <em>var_dump()</em>? ¿Nadie? ¿Solo yo? Ok, vale.
  </p>
  
  <h2>
    Composer
  </h2>
  
  <p>
    Si todavía no lo conoces es que has vivido debajo de una piedra los últimos 2 años. No me extenderé mucho: solo utilízalo. En serio. <a title="Composer" href="http://getcomposer.org/" target="_blank">Aquí tienes la documentación </a>.
  </p>
  
  <h2>
    Bonus track: Vagrant y Puphet
  </h2>
  
  <p>
    <a title="Vagrant" href="http://vagrantup.com/" target="_blank">Vagrant </a>no es una herramienta para <strong>PHP</strong> específicamente, pero en el mundo de muchos gigas de RAM en el que vivimos, trabajar con máquinas virtuales locales se está convirtiendo en un standard. El hecho de que con un simple archivo de configuración puedas tener el mismo entorno que tu compañero, es increíble.
  </p>
  
  <p>
    Hace 2 años que hice el curso sobre <a title="Puppet" href="http://puppetlabs.com/" target="_blank">Puppet </a>, y aunque no considero que haya sido perder el tiempo ni mucho menos, gracias a herramientas como <a title="Puphet" href="https://puphpet.com/" target="_blank">Puphet </a>, cada vez es menos necesario saber las entrañas de muchas cosas. <a title="Puphet" href="https://puphpet.com/" target="_blank">Puphet </a> es una abstracción de <a title="Puppet" href="http://puppetlabs.com/" target="_blank">Puppet </a> orientada a entornos <strong>PHP</strong>, de tal forma que como si un catálogo se tratase, vas eligiendo las tecnologías y configuraciones que quieres que tenga tu entorno de desarrollo. Con un par de clicks puedes elegir todo lo que necesites. Esto generará un archivo de configuración que será leído por <a title="Vagrant" href="http://vagrantup.com/" target="_blank">Vagrant </a>para moldear tu máquina virtual. Así de simple. Olvídate de como se instalan y configuran un montón de cosas: un par de clicks y listo. Si quieres cosas más avanzadas, tendrás que saber cómo funcionan, pero para el desarrollador <strong>PHP</strong> medio, esto es más que suficiente.
  </p>
  
  <h2>
    Conclusión
  </h2>
  
  <p>
    Además de estas herramientas, no olvides utilizar un IDE que te sea cómodo, y si no has probado <a title="PHPStorm" href="http://www.jetbrains.com/phpstorm/" target="_blank">PHPStorm </a>, te lo recomiendo.
  </p>
  
  <p>
    Así que ya sabes: utiliza Puphet y asegúrate de crear una máquina virtual que contenga composer, ladybug y boris, y fabrícate el entorno de desarrollo perfecto. No puedes programar igual que lo hacías hace 3 o 5 años. Esto va demasiado rápido como para permitírtelo.
  </p>
  
  <p>
    Y tú, ¿usas alguna herramienta interesante?
  </p>
</div>
