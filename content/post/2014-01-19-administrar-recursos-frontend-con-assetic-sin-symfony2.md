---
author: fiunchinho
categories:
- Code
date: 2014-01-19T18:12:29Z
dsq_thread_id:
- "2656043465"
guid: http://armesto.net/?p=26
id: 26
tags:
- performance
- php
title: Administrar recursos frontend con Assetic (sin Symfony2)
url: /administrar-recursos-frontend-con-assetic-sin-symfony2/
---

Buscando mejorar el rendimiento de nuestras aplicaciones web, muchas veces nos centramos en el backend sin prestar suficiente atención al frontend. Una de las mejoras que podemos aplicar en la parte frontal de nuestras webs para que vayan más rápidas es reducir el número de peticiones HTTP que son necesarias para cargar la web. Para ello podemos, por ejemplo, combinar varios archivos CSS o JS en uno solo para que, aunque tengamos que cargar muchos recursos, solo una petición HTTP sea necesaria.

Hacer esto a mano puede ser tedioso y llevar a problemas, por tanto una buena solución sería preguntarnos si alguien ya ha solucionado este problema antes. Como tantas otras veces la respuesta es que sí.

Hoy vengo a hablaros de <a title="Assetic" href="https://github.com/kriswallsmith/assetic" target="_blank">Assetic</a>, una herramienta que nos permite administrar fácilmente los recursos de la web, como archivos CSS o Javascript, para combinarlos, minificarlos u optimizarlos.

<!--more-->

El problema de <a title="Assetic" href="https://github.com/kriswallsmith/assetic" target="_blank">Assetic</a> es que, aunque encontrarás muchos ejemplos y artículos en internet, el 99% de ellos son utilizando <a title="Symfony" href="http://symfony.com/" target="_blank">Symfony2</a>. Mi objetivo con este post es explicar assetic para que puedas utilizarlo en cualquier sitio, independientemente del framework elegido.

## Vocabulario básico

Vamos a definir un vocabulario básico que nos puede ayudar cuando busquemos ayuda en internet o intentemos entender el funcionamiento de assetic:

  * **Asset**: Un recurso estático como un archivo CSS, un fichero Javascript o una imagen. Hay 2 clases que nos permitirán describir assets: _FileAsset_, que es un archivo normal; y _GlobAsset_, que representa un directorio que contiene varios archivos que queramos cargar.
  * **Filter**: Un filtro que transforma de alguna forma el contenido del fichero, como por ejemplo minificándolo, o traduciendo de SASS a CSS. Hay muchos filtros distintos.
  * **AssetManager**: Un administrador de los assets. Nos permitirá ponerle nombre, y crear grupos nombrando a otros assets declarados anteriormente. El asset al que le ponemos nombre puede ser un _FileAsset_, _GlobAsset_ o _AssetCollection_.
  * **AssetCollection**: Un conjunto de assets, es decir, varios _FileAsset_ o _GlobAsset_. **Puede contener otras colecciones**. El constructor acepta dos arrays, uno con los diferentes assets que componen el _collection_, y otro con el conjunto de _filters_ que aplicaremos.
  * **FilterManager**: Lo mismo que el _AssetManager_ pero para filtros.
  * **AssetFactory**: Una clase que nos facilitará el trabajo de conectar todo lo que hemos visto hasta ahora, pasándole un _AssetManager_ y un _FilterManager_.
  * **Formulae**: Lo que define a un asset: fichero/s y/o filtro/s utilizados para crearlo.
  * **Dump**: Generar el recurso especificado, pasándole los filters elegidos. Podemos utilizarlo para mostrar el contenido por pantalla de forma dinámica o para guardarlo en un fichero y servirlo de forma estática.

## Instalación de Assetic

Podemos instalar assetic <a title="Assetic" href="https://packagist.org/packages/kriswallsmith/assetic" target="_blank">a través de composer</a>.

## Sirviendo contenido dinámicamente

Vamos a realizar un ejemplo básico en el que vamos a coger todos los ficheros javascript de nuestro proyecto y vamos a hacer que assetic los combine en uno solo y los sirva dinámicamente, es decir, el resultado de combinarlos no lo guardará en disco,**sino que tendremos una URL en nuestra aplicación que generará el javascript combinado**. Esa URL es la que utilizaríamos en nuestro HTML para incluir el javascript.

<pre><code data-lang="php">require_once __DIR__.'/../vendor/autoload.php';

use Assetic\Asset\AssetCollection;
use Assetic\Asset\GlobAsset;

$js = new AssetCollection(array(
 new GlobAsset( '/var/www/my_project/web/js/*' ),
));

// Vamos a mostrar código Javascript, por tanto
// debemos especificárselo al navegador con la cabecera correspondiente
header( 'Content-type: text/javascript' );

// Mostramos el código Javascript
echo $js-&gt;dump();</code></pre>

**Es nuestra labor crear una página/controlador/sección con este código y una ruta de nuestra web apuntando hacia ahí**. Si la visitamos, veremos por pantalla todo el código Javascript correspondiente a todos los archivos que había en la carpeta _/var/www/my_project/web/js/_ combinado en un solo archivo. Por tanto, en nuestro HTML podríamos cambiar todos los archivos javascript incluídos y dejar solo este, **reduciendo el número de peticiones HTTP y mejorando la velocidad y rendimiento de la web**.

No es obligatorio utilizar _AssetCollection_ para esto, ya que podríamos haber utilizado directamente un _FileAsset_ o _GlobAsset_, pero normalmente vamos a tener más de un archivo.

Si no queremos coger todos los archivos de una carpeta, sino solo algunos, tendríamos que hacer lo siguiente

<pre><code data-lang="php">require_once __DIR__.'/../vendor/autoload.php';

use Assetic\Asset\AssetCollection;
use Assetic\Asset\FileAsset;
use Assetic\Asset\GlobAsset;

$js = new AssetCollection(array(
 new FileAsset( '/var/www/my_project/web/js/main.js' ),
 new FileAsset( '/var/www/my_project/web/js/jquery.min.js' ),
 new GlobAsset( '/var/www/my_project/web/js/bootstrap/*' )
));
// Vamos a mostrar código Javascript, por tanto
// debemos especificárselo al navegador con la cabecera correspondiente
header( 'Content-type: text/javascript' );

// Mostramos el código Javascript
echo $js-&gt;dump();</code></pre>

Si os fijáis en el output del javascript combinado que genera este código, salvo que el JS estuviese minificado de antes, no estará minificado ahora. En el caso de haber tenido coffeescript, esto no nos lo hubiese compilado a javascript. Para hacer este tipo de tareas tenemos que utilizar los filtros.

## Usando filtros

Los filtros nos van a permitir transformar el contenido de los asset que habíamos definido. Para definirlos tenemos distintos métodos. Podemos definir un determinado asset con un filtro específico si solo queremos que se aplique a ese en particular. Pero también podemos asignar filtros a un AssetCollection y que lo aplique a todos. Dependerá de qué es lo que queremos hacer.

En el siguiente ejemplo vamos a ver los dos casos. Un filtro particular para compilar de código LESS a CSS, y el <a title="YUI" href="http://yuilibrary.com/" target="_blank">compresor YUI</a> para que minifique todos los assets de la colección.

<pre><code data-lang="php">require_once __DIR__.'/../vendor/autoload.php';

use Assetic\Asset\AssetCollection;
use Assetic\Asset\FileAsset;
use Assetic\Asset\GlobAsset;
use Assetic\Filter\LessFilter;
use Assetic\Filter\Yui;

$css = new AssetCollection(array(
    new FileAsset('/path/to/src/styles.less', array(new LessFilter())),
    new GlobAsset('/path/to/css/*'),
), array(
    new Yui\CssCompressorFilter('/path/to/yuicompressor.jar'),
));

header( 'Content-type: text/css' );
// this will echo CSS compiled by LESS and compressed by YUI
echo $css-&gt;dump();</code></pre>

Creando un controlador con el código anterior, y una ruta apuntando a él, podríamos ir a nuestro código HTML y cambiar todas las peticiones de CSS en una sola hacia dicha ruta.

<a title="Assetic Filters" href="https://github.com/kriswallsmith/assetic#filters" target="_blank">La cantidad de filtros disponibles es enorme</a> y salvo que quieras realizar algo extraño, encontrarás uno que hace lo que buscas. Recuerda que es probable que para utilizar estos filtros, tengas que instalar la herramienta en cuestión. Por ejemplo, para utilizar YUI como en mi ejemplo, tendrías que instalar YUI (en Ubuntu):

<pre>sudo apt-get install yui-compressor</pre>

Y **el lugar de instalación es el que debes poner en el constructor del filtro**, para que sepa donde está el _jar_ ejecutable.

## Organizando mejor los recursos

Los ejemplos que hemos visto están bien, pero en un proyecto más grande tendrás muchos recursos css o js que quieras cargar y la cosa puede complicarse.

<a title="Assetic" href="https://github.com/kriswallsmith/assetic" target="_blank">Assetic</a> intenta hacernos la vida más fácil a través del _AssetManager_ y el _FilterManager_.

### AssetManager

El _AssetManager_ es… bueno, eso, un administrador de assets. Básicamente podemos ponerle un nombre a cada asset definido. ¿Por qué es esto importante? Pues porque luego podemos hacer referencia a un asset definido con anterioridad, y el_AssetManager_ se encargará de que assetic no haga todo el trabajo sobre el mismo recurso dos veces.

Por ejemplo, si queremos cargar jQuery y además, un plugin de jQuery que necesita, obviamente, de jQuery.

<pre><code data-lang="php">
use Assetic\Asset\AssetCollection;
use Assetic\Asset\FileAsset;
use Assetic\AssetManager;
use Assetic\Asset\AssetReference;

$jquery = new FileAsset( '/path/to/jquery.min.js' );

$am = new AssetManager();
$am-&gt;set( 'jquery', $jquery );

$plugin1 = new AssetCollection( array(
    new AssetReference( $am, 'jquery' ),
    new FileAsset( '/path/to/jquery.plugin.js' )
));

$plugin2 = new AssetCollection( array(
    new AssetReference( $am, 'jquery' ),
    new FileAsset( '/path/to/another/jquery.plugin.js' )
));

$js = new AssetCollection( array(
    $jquery,
    $plugin1,
    $plugin2
));
header( 'Content-type: text/javascript' );
echo $js-&gt;dump();
</code></pre>

En el código anterior, aunque utilicemos jQuery varias veces, Assetic solo lo tratará una vez. Con este mecanismo podríamos generar distintos paquetes dependiendo de la sección de la web en la que estemos, cargando solo el Javascript necesario para esa sección, y assetic solo haría el trabajo una vez, aunque repitiésemos ficheros.

### FilterManager

El _FilterManager_ es algo muy parecido al _AssetManager_, pero para filtros. Simplemente damos de alta los filtros que estarán disponibles a utilizar luego cuando creemos assets con el factory.

Igual que con el _AssetManager_, si Assetic detecta que vamos a aplicar el mismo filtro al mismo archivo, dos veces, solo lo hará una. Un poco abajo se ve el código necesario para utilizar el _FilterManager_.

### AssetFactory

Para que todo esto sea más fácil de utilizar y no tengamos que andar creando y conectando todos estos objetos a mano, <a title="Assetic" href="https://github.com/kriswallsmith/assetic" target="_blank">Assetic</a> tiene una clase llamada AssetFactory para generar assets de forma sencilla.

<pre><code data-lang="php">require_once __DIR__.'/../vendor/autoload.php';

use Assetic\Asset\AssetFactory;
use Assetic\Asset\FilterManager;
use Assetic\Filter\Yui\CssCompressorFilter;
use Assetic\Filter\Yui\JsCompressorFilter;

$fm = new FilterManager();
$fm-&gt;set('yui_css',new CssCompressorFilter('/path/yuicompressor.jar'));
$fm-&gt;set('yui_js',new JsCompressorFilter('/path/yuicompressor.jar'));

$factory = new AssetFactory( '/path/doc_root' );
$factory-&gt;setAssetManager( new AssetManager() );
$factory-&gt;setFilterManager( $fm );

$css = $factory-&gt;createAsset(
    array(
        'css/style.css',
        'css/bootstrap/*.css', // css in "/path/doc_root/css/bootstrap"
), array(
        'yui_css' // filter through the filter manager's "yui_css"
));

header( 'Content-type: text/css' );
echo $css-&gt;dump(); </code></pre>

La factory funciona de forma parecida a una _AssetCollection_, ya que le pasamos un array de assets y otro de filtros. Como se basa en un _AssetManager_ y en un _FilterManager_, todas las propiedades de estos se aplican al utilizar la factory.

Con respecto al _FilterManager_, fíjate que hemos creado dos filtros, aunque luego solo estamos utilizando uno en el factory, el filtro llamado “_yui_css_“.

## Mejorando la velocidad: guardando en disco

Hasta aquí todo bien. El único problema es que cada vez que se visita la ruta correspondiente a un asset y se ejecuta la llamada al método _dump()_, tiene que leer el contenido del disco, juntarlo y aplicar los filtros elegidos. Esto hará que la web cargue más lenta, con lo cual nuestro objetivo inicial de mejorar el rendimiento se va al traste.

Pero no sufras: todo tiene solución. En vez de generarlo cada vez, podríamos generarlo solo una vez y guardarlo en disco para que en las siguientes peticiones se sirva estáticamente la versión generada.

Para ello, <a title="Assetic" href="https://github.com/kriswallsmith/assetic" target="_blank">Assetic</a> nos proporciona un _AssetWriter_ para escribir en el disco aquello que generemos.

<pre><code data-lang="php">require_once __DIR__.'/../vendor/autoload.php';

use Assetic\Asset\AssetFactory;
use Assetic\Asset\FilterManager;

$css = $factory-&gt;createAsset(
    array(
        'css/style.css',
        'css/bootstrap/*.css', // css in "/path/doc_root/css/bootstrap"
), array(
        'yui_css' // filter through the filter manager's "yui_css"
));

$writer = new AssetWriter( '/path/doc_root/generated' );
$writer-&gt;writeAsset( $css );</code></pre>

Tan solo le pedimos al writer que escriba en disco los assets que queramos. Si tenemos varios assets dentro de un _AssetManager_, también podemos pasarle un _AssetManager_ y que automáticamente escriba en disco todos los assets configurados en el manager. Lo normal suele ser escribirlos en una carpeta aparte de archivos “compilados” o “generados”, pero eso ya depende de como te quieras organizar.

De esta forma, no serviríamos css y js de forma dinámica como estábamos viendo hasta ahora, es decir, no tendríamos una ruta que generase “al vuelo” el contenido. Por el contrario, cargaríamos un archivo normal del disco, archivo que generamos mediante el último código visto. Este código podemos ponerlo en un script PHP de consola que ejecutaremos manualmente cada vez que queramos re-escribir en el disco nuestros recursos, como, por ejemplo, **cuando hacemos deploy de nuestra aplicación**.

El nombre que el fichero tendrá en el disco es generado automáticamente por Assetic, utilizando el hash **SHA1**, basándose en los assets, los filtros y las opciones elegidas, de tal forma que si esto varía, el SHA1 variará y el nombre del fichero sería distinto, por lo que **tendríamos que volver a guardarlo en el disco**.

Si no nos interesa este comportamiento, Assetic también nos deja elegir el nombre final del archivo, pasándoselo en un array de opciones al factory.

<pre><code data-lang="php">require_once __DIR__.'/../vendor/autoload.php';

use Assetic\Asset\AssetFactory;
use Assetic\Asset\FilterManager;

$css = $factory-&gt;createAsset(
    array(
        'css/style.css',
        'css/bootstrap/*.css', // css in "/path/doc_root/css/bootstrap"
), array(
        'yui_css' // filter through the filter manager's "yui_css"
), array(
        'output' =&gt; 'my_awesome_css.css'
));

$writer = new AssetWriter( '/path/doc_root/generated' );
$writer-&gt;writeAsset( $css );</code></pre>

## Conclusión

Con esta introducción creo que queda más claro qué es qué dentro de <a title="Assetic" href="https://github.com/kriswallsmith/assetic" target="_blank">Assetic</a>, y cómo podríamos empezar a utilizarlo en **una aplicación que no utilice Symfony2**.

En el siguiente post sobre Assetic, veremos algunos conceptos más avanzados como su integración con <a title="Twig" href="http://twig.sensiolabs.org/" target="_blank">Twig</a>, o cache busting.