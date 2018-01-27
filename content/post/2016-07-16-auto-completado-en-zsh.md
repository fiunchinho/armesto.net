---
author: fiunchinho
categories:
- Linux
date: 2016-07-16T17:49:15Z
dsq_thread_id:
- "4990566637"
guid: https://blog.armesto.net/?p=135
id: 135
title: Auto completado en zsh
url: /auto-completado-en-zsh/
thumbnail: https://ih1.redbubble.net/image.154489147.8220/sticker,375x360.png
---

Hace tiempo escribí un post sobre [cómo sacarle el máximo rendimiento a tu terminal utilizando zsh](https://blog.armesto.net/sacale-el-maximo-partido-a-tu-terminal-con-zsh/), en el que introducía [zsh](http://www.zsh.org/) y el framework [oh-my-zsh](http://ohmyz.sh/) para tener una experiencia más placentera usando el terminal.

Hoy veremos algunas funcionalidades extras que nos aporta esta herramienta y que nos puede ayudar a decidirnos a empezar a utilizarla.

<!--more-->

## Auto Completado

Una de las cosas que comentamos [en el artículo original](https://blog.armesto.net/sacale-el-maximo-partido-a-tu-terminal-con-zsh/) era que el auto completado de comandos era bastante potente, pero podemos mejorarlo aun más. Añadiendo esto a nuestra configuración

<pre>zstyle ':completion:*' verbose yes
zstyle ':completion:*' group-name ''</pre>

obtendremos la lista de posibles acciones agrupadas por tipo. Por ejemplo, escribiendo vi y dándole al tabulador

<pre>$ vi
 --- external command
 VirtualBox  vi          view        vim         vimdiffvisudo
 --- shell function
 vi_mode_prompt_info     virtualenv_prompt_info
 --- alias
 vim
 --- local directory
 videos/</pre>

Como podéis ver, en vez de salirme un listado enorme con todo, vemos las opciones agrupadas por tipo. Como dijimos anteriormente, no hace falta usar el tabulador para moverme por las opciones, si no que puedo utilizar los cursores del teclado para ir más rápido.

Y no solo vale con comandos, si no también con argumentos de los comandos:

<pre>$ ping -
 --- option
 -L  -- suppress loopback of multicast packets
 -Q  -- somewhat quiet
 -R  -- record route
 -a  -- audible for each packet
 -d  -- set SO_DEBUG on the socket
 -f  -- flood ping
 -n  -- numeric output only
 -q  -- quiet
 -r  -- bypass normal routing tables
 -v  -- verbose
 -I  -P  -S  -T  -c  -i  -l  -m  -p  -s  -t</pre>

O subcomandos:

<pre>$ git
 --- common commands
 add       -- add file contents to the index
 bisect    -- find by binary search the change that introduced
 branch    -- list, create, or delete branches
 checkout  -- checkout a branch or paths to the working tree
 clone     -- clone a repository into a new directory
 commit    -- record changes to the repository
 diff      -- show changes between
 fetch     -- download objects and refs from another repository
 grep      -- print lines matching a pattern
 init      -- create an empty Git repository
 log       -- show commit logs
 merge     -- join two or more development histories together
 mv        -- move or rename a file, a directory, or a symlink
 pull      -- fetch from and merge with another repository
 push      -- update remote refs along with associated objects
 rebase    -- forward-port commits to the updated upstream
 reset     -- reset current HEAD to the specified state
 rm        -- remove files from the working tree
 show      -- show various types of objects
 status    -- show the working tree status
 tag       -- create, list, delete or verify a tag object signed</pre>

## Sugerencias

He visto muchas utilidades en Gthub que cuando escribimos un comando incorrectamente en la terminal, nos sugiere comandos válidos porque seguramente nos hemos equivocado en algún caracter. Esto que es bastante útil viene también con zsh. Podemos comprobar cómo funciona

<pre>$ vom
 zsh: correct 'vom' to 'vim' [nyae]? y</pre>

Cuando detecta que el comando es incorrecto, nos ofrece alguna sugerencia y nos da cuatro posibles respuestas:

  * n: no, quiero probar mi comando aunque no lo reconozca zsh
  * y: sí, quiero abrir la sugerencia de zsh
  * a: abortar, olvídate de mi comando y volvamos a la terminal
  * e, volver a la terminal para editar mi comando antes de ejecutarlo

### Sugerencias en auto completado

Fíjate que incluso mezcla estas dos funcionalidades! Utilicemos el auto completado otra vez

<pre>$ cat tost
 corrections (errors: 1)
 test.jmx            test.json
 --- original
 tost</pre>

Al escribir **_cat tost_** y pulsar tabulador, me sugiere posibles correciones, ya que no hay ningún fichero que coincida con el nombre &#8220;tost&#8221;, pero sí con &#8220;test&#8221;.

Lo mismo con comandos

<pre>$ vom
 corrections (errors: 1)
 comptry                             vim
 compcall                            vimdiff
 composer                            homebrew/
 --- original
 vom</pre>

## Conclusión

Como véis, hay muchos motivos para utilizar zsh y que estar en la terminal sea un placer. ¿Conocéis algún otro truco que no esté aquí?
