---
author: fiunchinho
categories:
- Code
date: 2013-11-02T18:11:01Z
dsq_thread_id:
- "2648640818"
guid: http://armesto.net/?p=24
id: 24
tags:
- php
title: Geolocalización en PHP y las páginas de afiliados locales
url: /geolocalizacion-en-php-y-las-paginas-de-afiliados-locales/
---

Una de las webs de las que soy responsable contiene artículos sobre productos electrónicos: análisis, comparativas, novedades, etc. Utilizando el sistema de afiliados de Amazon y, si tus análisis convencen a los usuarios para rascarse el bolsillo, puedes sacar un porcentaje de la venta.

El problema con Amazon es que tiene **una tienda distinta para cada país**. Es decir, Amazon para España no es el mismo que para Francia o Estados Unidos. El catálogo es distinto, sus usuarios diferentes y hasta el programa de afiliados funciona de manera distinta. De hecho, para participar en el programa de afiliados tienes que darte de alta por separado en cada uno de los países que quieras participar.

Mientras hacíamos esto, mi compañero y yo nos dimos cuenta de un detalle. Si un usuario desde México lee nuestro análisis y quiere comprarse el producto, seguramente lo quiera comprar en la tienda de Estados Unidos, no en la Española. Sin embargo, tener que poner en todos nuestros artículos varias versiones de enlaces del tipo “_si vienes desde México entra aquí, si vienes desde España entra allí_“, no nos parecía una opción viable. Así que, ¿cómo podíamos solventar esto?

<!--more-->

## Intentando traducir los enlaces “al vuelo”

Al principio pensé en crear un script en Javascript que al terminar de cargar la página cambiase todos los enlaces de la página, _traduciéndolos_ a la tienda del país local del visitante. Pero como decía antes, los catálogos no son iguales y los enlaces de Amazon no se pueden traducir tan fácilmente. Hay un plugin de WordPress que hace esto, pero parece que **falla más que una escopeta de feria**.

## Sistema de enlaces propio

Así que finalmente opté por crear mi propio sistema de enlaces geolocalizados. ¿Qué es esto? Pues básicamente se trata de no utilizar los enlaces de tu sistema de afiliados, sino de utilizar unos enlaces propios que yo mismo genero y que **redirigirán a la tienda local dependiendo del país del visitante**.

Para ello, a partir de ahora en mis artículos no utilizaré el link de Amazon sino unos enlaces bajo mi propio dominio, como por ejemplo

  * <a title="www.example.com/products/ipad" href="www.example.com/products/ipad" target="_blank">www.example.com/products/ipad</a>
  * <a title="www.example.com/products/ipad-mini" href="www.example.com/products/ipad-mini" target="_blank">www.example.com/products/ipad-mini</a>
  * <a title="www.example.com/products/nexus10" href="www.example.com/products/nexus10" target="_blank">www.example.com/products/nexus10</a>
  * <a title="www.example.com/products/nexus7" href="www.example.com/products/nexus7" target="_blank">www.example.com/products/nexus7</a>

Cuando un usuario visite esos enlaces, mi sistema hará dos cosas

  1. Detectar el país del visitante
  2. Basándonos en el país y el producto visitado, redirigir al usuario a la tienda correcta

### Detectando el país del visitante

Para geolocalizar al usuario he utilizado la librería <a title="Geocoder" href="http://geocoder-php.org/" target="_blank">Geocoder</a> que está lista para utilizar en los principales frameworks como Symfony o Laravel, así como un port a Javascript.

Es una librería muy completa ya que te permite configurar completamente el mecanismo que se utilizará para geolocalizar. Aunque <a title="Geocoder" href="http://geocoder-php.org/Geocoder/" target="_blank">la documentación</a> no es perfecta, da mucha información sobre todas las posibilidades que ofrece.

Lo más básico que necesitas saber si quieres usarla, es que necesitas un proveedor de geolocalización y un adaptador para hablar con ese proveedor. **Hay muchísimos proveedores**. Desde FreeGeoIP, hasta Google Maps, pasando por GeoIP. Para utilizar estos proveedores necesitas un adaptador que lo ejecute, que puede ser desde un simple curl hasta librerías como Buzz o Guzzle.

En mi caso, quería lo más sencillo posible, así que opté por utilizar FreeGeoIP a través de curl. Para eso utilicé este código

    $adapter     = new Geocoder\HttpAdapter\CurlHttpAdapter();
    $provider    = new Geocoder\Provider\FreeGeoIpProvider( $adapter );
    $geocoder    = new Geocoder\Geocoder();
    $geocoder->registerProvider( $provider );
    

Con esto ya tenemos un objeto que dados unos datos del usuario, nos puede dar información sobre la posición de éste

<div>
  <pre><code data-lang="php">$result = $geocoder-&gt;geocode('88.188.221.14');
// Result is:
// "latitude"       =&gt; string(9) "47.901428"
// "longitude"      =&gt; string(8) "1.904960"
// "bounds"         =&gt; array(4) {
//     "south" =&gt; string(9) "47.813320"
//     "west"  =&gt; string(8) "1.809770"
//     "north" =&gt; string(9) "47.960220"
//     "east"  =&gt; string(8) "1.993860"
// }
// "country"        =&gt; string(6) "France"
// "countryCode"    =&gt; string(2) "FR"
// "timezone"       =&gt; string(6) "Europe/Paris"</code></pre>
  
  <h3>
    Redirigiendo a la tienda local
  </h3>
  
  <p>
    Ahora que podemos saber el país del visitante, tan solo tenemos que coger de un diccionario cual es el enlace que nos interesa. Además del país del usuario, el otro dato que usaremos en nuestro diccionario, es el producto en cuestión. Esto podemos obtenerlo a través de la URL de nuestro sistema de enlaces.
  </p>
  
  <p>
    Utilizando los enlaces de ejemplo que puse más arriba, si el usuario entrase desde España en <a title="www.example.com/products/ipad-mini" href="www.example.com/products/ipad-mini" target="_blank">www.example.com/products/ipad-mini</a>, yo le redireccionaré a la página del iPad Mini de la tienda española.
  </p>
  
  <p>
    Para trabajar con la URL de forma segura, así como para conseguir la IP del usuario de forma fácil, he decidido utilizar el componente HttpFoundation de Symfony. Esta librería también me permite realizar cómodamente la redirección 302 a la tienda local.
  </p>
  
  <p>
    La gestión del diccionario puedes hacerla con un simple array en el mismo archivo PHP, o mediante algún archivo de configuración como YAML o XML. Si el catálogo de productos fuese muy extenso, habría que realizar un backend para poder administrarlo, pero de momento es pequeño y manejable.
  </p>
  
  <p>
    Para conseguir que todas las peticiones a esos enlaces se procesen a través del script que estamos creando, tendrás que cambiar tu virtual host. En Apache sería añadir algo parecido a esto
  </p>
  
  <pre><code>&lt;Directory /var/www/project/products/&gt;
        &lt;IfModule mod_rewrite.c&gt;
            RewriteEngine On
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteCond %{REQUEST_FILENAME} !-d
            RewriteRule . /products/index.php [L]
        &lt;/IfModule&gt;
&lt;/Directory&gt;</code></pre>
  
  <h2>
    Conclusión
  </h2>
  
  <p>
    Con este mecanismo, tengo totalmente centralizado el sistema de enlaces de mi programa de afiliados. Además del beneficio que yo buscaba de poder utilizar la tienda local del usuario, en vez de siempre la misma, hay otro bastante interesante. Si yo quisiera cambiar de tienda, y en vez de utilizar Amazon utilizar otro proveedor, no tendría que cambiar todos mis artículos y anuncios, tan solo el diccionario del sistema de enlaces.
  </p>
  
  <p>
    Nosotros en nuestra web tenemos la política de enlazar a la tienda con el precio más barato para el usuario, sin importar que esa tienda no nos de un porcentaje por la venta. Con este mecanismo, todo se ha vuelto mucho más fácil, porque si hay cambios en los precios <strong>tan solo tenemos que cambiar una línea en el diccionario</strong>.
  </p>
  
  <p>
    Podéis ver el código de todo este sistema en <a title="Github fiunchinho" href="https://github.com/fiunchinho/affiliation" target="_blank">mi cuenta de Github</a>.
  </p>
</div>