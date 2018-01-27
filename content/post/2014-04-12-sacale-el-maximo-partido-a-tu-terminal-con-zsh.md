---
author: fiunchinho
categories:
- Linux
date: 2014-04-12T13:42:17Z
dsq_thread_id:
- "2648474449"
guid: https://blog.armesto.net/?p=62
id: 62
title: Sácale el máximo partido a tu terminal con zsh
url: /sacale-el-maximo-partido-a-tu-terminal-con-zsh/
---

No quiero tirar de términos de moda y hablar de _DevOps_, pero sí es cierto que, yo por lo menos, paso cada vez más tiempo delante de una terminal de UNIX. Estoy empezando a sentirme como en casa en algo que hace unos años me asustaba bastante.

Hoy vengo a hablaros de <a title="zsh" href="http://www.zsh.org/" target="_blank">zshell</a>, una alternativa a _bash_, el intérprete que viene por defecto en la mayoría de distribuciones. Algunos pensaréis&#8230; &#8220;¿_Cambiar el intérprete? ¡Eso suena muy difícil!_&#8220;, pero nada más lejos de la realidad! Otros diréis&#8230; &#8220;¿_Para qué iba yo a querer uno distinto? ¿Cual es el problema con bash?_&#8221; Problema ninguno! Pero con este post espero enseñaros todo lo que os estáis perdiendo por no utilizar zshell. Además, lo más importante es que no hay que aprender ningún comando nuevo: **practicamente todo lo que usas en bash funcionará en zsh**.

<!--more-->

## Instalación de zshell

Instalarlo es muy simple en la mayoría de distribuciones, ya que hay un paquete ya listo para nosotros. No os preocupéis que por el hecho de instalarlo, no lo estamos activando todavía. Por ejemplo, en Ubuntu:

<pre>sudo apt-get install zsh</pre>

Una vez instalado tenemos que configurarlo. Una de las ventajas de zshell, es que está pensado para que puedas configurarlo de forma modular a través de plugins y templates. Para ayudarnos a gestionar la configuración, vamos a instalar otra cosa rápidamente: <a title="oh-my-zsh" href="http://ohmyz.sh/" target="_blank">oh-my-zsh</a>. Es un framework hecho por la comunidad **con un montón de plugins** que además de aportarnos funcionalidad extra, hará que sea fácil configurar zsh a nuestro antojo. Tranquilos, instalar esto tampoco activará zsh. Para instalar oh-my-zsh automáticamente solo ejecutamos:

<pre>curl -L http://install.ohmyz.sh | sh</pre>

Para los indecisos (espero que no quede ninguno al finalizar el post), podéis probar este intérprete sin ponerlo por defecto, y solo hacer el cambio cuando estéis 100% seguros. Para probarlo y ver de qué va esto, ejecutad en la consola:

<pre>zsh</pre>

Veréis que de repente vuestro _prompt_ ha cambiado. Tranquilos, es normal: zsh tiene configurado un prompt distinto al que bash tiene. Vamos a ver algunas de las ventajas más obvias de zsh. Recordad que siempre podéis volver a dejarlo todo como estaba escribiendo:

<pre>bash</pre>

## Ventajas de zsh

### Prompt

Una de las cosas que más nos gusta a los frikis de la consola es configurar nuestro prompt para que aparezca información relevante más allá del _path_ actual en el que estamos. El que _zsh_ trae por defecto ya es bastante bueno, indicando información de la rama actual de git en la que estamos. Esto ya es configurable con _bash_, pero al haber instalado _oh-my-zsh_ tenemos acceso a <a title="zsh themes" href="https://github.com/robbyrussell/oh-my-zsh/wiki/Themes" target="_blank">todos los themes de la comunidad</a> donde podremos elegir **verdaderas maravillas**. Para activar un theme, tan solo tenemos que modificar el archivo de configuración en _~/.zshrc_ y cambiar el que viene por defecto, por uno del listado. La línea a modificar es:

<pre>ZSH_THEME="robbyrussell"</pre>

Si queremos hacerlo nosotros mismos, podemos configurar no solo el prompt normal, sino que **nos permite configurar texto a la derecha del prompt**. Para que os hagáis una idea, el que yo utilizo actualmente muestra, además de lo habitual, la rama actual de git, si hay cambios por comittear pendientes, el _load_ de la máquina y la hora actual, estos dos últimos en la parte derecha del prompt. Luce así:

[<img class="alignnone size-large wp-image-67" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_031-1024x47.png" alt="zsh prompt" width="620" height="28" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_031-1024x47.png 1024w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_031-300x14.png 300w" sizes="(max-width: 620px) 100vw, 620px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_031.png)

### Navegación entre directorios

Probad a escribir &#8216;cd &#8216; y darle a tabulador para que os sugiera los posibles directorios. Dadle varias veces a tabulador y podréis ir eligiendo uno a uno. ¿Que no queréis ir uno a uno? ¡Sin problema! Utilizad las flechas del teclado para navegar entre las sugerencias.

Elegid uno de los directorios para entrar en él. Ahora probad a escribir &#8216;..&#8217;. Con eso iréis al directorio padre. Y si ponéis &#8216;&#8230;&#8217; al superior. Y así sucesivamente. Útil, ¿no?

A veces es un poco engorroso recordar exactamente el nombre del directorio al que queremos acceder. Tranquilos, **zsh es muy listo** y si intentáis entrar en un directorio pero poniendo solo parte de su nombre, zsh intentará averiguar a cual te refieres y, si lo consigue, entrará en él. **Incluso aunque lo que hayas escrito no sea el principio del nombre del directorio**.

Además, escribir &#8220;cd&#8221; para movernos es _poco eficiente_. Si probáis a escribir tan solo el nombre de la carpeta, sin ponerle el &#8220;cd&#8221; delante, también entrará en el directorio.

Si llegados a esto todavía no os parece canela en rama, tranquilos, que hay más.

### Mejor historial

En la consola acabamos repitiendo los mismos comandos varias veces. Hasta ahora, yo utilizaba control+R para buscar en el historial, o si era un comando reciente, le daba a la flecha hacia arriba del cursor para ir hacia atrás en el historial. Esto está bien, pero zsh lo mejora haciendo que solo vaya recorriendo **aquellos comandos que coinciden con lo que tienes actualmente escrito**. Es decir, si yo escribo &#8220;vim&#8221; y voy dándole hacia arriba, solo irá navegando por el historial de comandos que empezaban con &#8220;vim&#8221;.

Además, siempre podemos conseguir que zsh sea más eficiente, viendo qué comandos son los que más utilizamos y creando alias para ellos. Igual que en _bash_, con la diferencia de que zsh nos proporciona un comando con información detallada sobre lo que más utilizamos:

[<img class="alignnone size-full wp-image-69" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_032.png" alt="zsh_stats" width="493" height="182" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_032.png 493w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_032-300x110.png 300w" sizes="(max-width: 493px) 100vw, 493px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_032.png)

### Plugins

Todo esto es lo que nos ofrece zsh por defecto, pero lo bueno de oh-my-zsh es que **nos ofrece un montón de plugins** que añaden alias a acciones de nuestro día a día para que la shell sea todavía más cómoda. Para añadir o quitar plugins, tan solo tenemos que editar el archivo de configuración de zsh situado en ~/.zshrc y modificar la línea de los plugins. Actualmente la mía está así:

<pre>plugins=(git autojump colored-man colorize extract zsh-syntax-highlighting)</pre>

Con estos plugins obtengo desde archivos del manual con texto coloreado, hasta un comando nuevo llamado **_extract_** que puedo utilizar para descomprimir cualquier tipo de archivo comprimido **sin preocuparme de cómo se descomprime**: él decidirá qué es lo que debe utilizar: zip, gz, etc. Además, como habréis notado en los pantallazos anteriores, cuando estoy escribiendo un comando en la terminal, está coloreado en verde si la shell lo reconoce. Si estoy escribiendo un comando que no existe, zsh lo escribe en rojo.

#### AutoJump

Quiero hacer mención especial a uno de los plugins que tengo activados: **autojump**. Si la navegación entre directorio ya es la más cómoda del mundo con zsh, autojump va todavía más allá y va guardando tus directorios más utilizados en tu día a día. De tal forma que tan solo tienes que escribir &#8220;j&#8221; seguido de una cadena que él intentará matchear contra tu historial de directorios, y entrará en aquel que más hayas visitado. Esta cadena puede ser desde una letra, hasta el nombre entero de la carpeta. En este ejemplo, escribiendo &#8220;j w&#8221;, él ya sabía que me estaba refiriendo a _/var/www_.

[<img class="alignnone size-full wp-image-70" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_034.png" alt="Autojump" width="373" height="145" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_034.png 373w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_034-300x116.png 300w" sizes="(max-width: 373px) 100vw, 373px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_034.png)

Para instalarlo tan solo ejecuta:

<pre>sudo apt-get install autojump</pre>

## Dejando zsh por defecto en vez de bash

Si finalmente os he convencido, solo queda activar zsh para que sea siempre vuestra shell, en vez de bash. Para hacer esto, solo tenéis que ejecutar:

<pre>chsh -s $(which zsh)</pre>

Si por algún motivo que se me escapa quisierais volver a utilizar bash, solo tenéis que ejecutar:

<pre>chsh -s $(which bash)</pre>

## Conclusión

Hay <a title="zsh features" href="http://www.slideshare.net/jaguardesignstudio/why-zsh-is-cooler-than-your-shell-16194692" target="_blank">muchísimas más cosas</a> que me dejo en el tintero. Como véis, **zsh** aporta un montón de comodidades extras a _bash_. Realmente, no hay nada que no podamos hacer a mano en bash, configurando alias y creando scripts, pero es mucho mejor apoyarnos en la sabiduría de toda una comunidad dedicada a esto.

¿Alguno no se ha convencido de cambiarse?

## Enlaces de interés

<a title="Problema con ciertos prompts que duplican el comando" href="http://unix.stackexchange.com/questions/90772/first-characters-of-the-command-repeated-in-the-display-when-completing" target="_blank">Problema con ciertos prompts que duplican el comando</a>. Asegúrate de que tu fichero .zshrc tiene estas dos líneas:

<pre>export LANG=en_US.utf8
export LC_ALL=en_US.utf8</pre>

<a title="Tutorial de zsh" href="Getting started with ZSH on Ubuntu" target="_blank">Tutorial de zsh</a>

<a title="oh-my-zsh" href="http://ohmyz.sh/" target="_blank">Oh-My-zsh!</a>

<a title="Plugins de oh-my-zsh" href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins" target="_blank">Plugins de oh-my-zsh</a>

<a title="autojump" href="https://github.com/joelthelion/autojump" target="_blank">Autojump</a>
