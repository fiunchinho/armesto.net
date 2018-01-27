---
author: fiunchinho
categories:
- Linux
date: 2014-04-19T09:00:56Z
dsq_thread_id:
- "2648654375"
guid: https://blog.armesto.net/?p=77
id: 77
tags:
- puppet
- vagrant
title: Utilizando Puphpet también en producción
url: /utilizando-puphpet-tambien-en-produccion/
---

Ya <a title="Herramientas para el programador PHP moderno" href="https://blog.armesto.net/herramientas-para-el-programador-php-moderno/" target="_blank">hablé en el pasado sobre Puphpet</a>, una herramienta web que genera manifiestos de <a title="Puppet" href="http://puppetlabs.com/" target="_blank">Puppet</a> y de <a title="Vagrant" href="http://www.vagrantup.com/" target="_blank">Vagrant</a> de forma rápida y sencilla.

Llevo tiempo utilizándolo para configurar mis máquinas virtuales de desarrollo junto a Vagrant, pero siempre estaba un poco con la mosca detrás de la oreja por el hecho de no poder utilizarlo también en producción.

Así que el otro día me decidí a echar un ojo al código fuente de <a title="Puphpet" href="https://puphpet.com/" target="_blank">Puphpet</a> y ver si podía utilizar solo la parte generada de <a title="Puppet" href="http://puppetlabs.com/" target="_blank">Puppet</a>, y olvidarme de la parte de <a title="Vagrant" href="http://www.vagrantup.com/" target="_blank">Vagrant</a>.

<!--more-->

Resulta que para hacer su magia, <a title="Puphpet" href="https://puphpet.com/" target="_blank">Puphpet</a> tan solo le dice a Vagrant que ejecute unos scripts de _bash_. Así que pensé, **¿qué pasa si los ejecuto yo manualmente en producción?** Los scripts solo se encargaban de instalar puppet, curl y alguna librería más, así que no debía de haber mucho problema. Después de hacer esto, tan solo le diría a puppet que leyese el manifiesto generado por Puphpet y listo. Mi plan sonaba bien, así que me propuse a intentarlo.

Para asegurarme de que nada fallaba, coloqué los archivos generados por <a title="Puphpet" href="https://puphpet.com/" target="_blank">Puphpet</a> en el mismo sitio que cuando <a title="Vagrant" href="http://www.vagrantup.com/" target="_blank">Vagrant</a> los utiliza, es decir _/vagrant_. Entré en mi servidor de producción y subí los archivos de forma que quedaron así:

[<img class="alignnone size-full wp-image-78" alt="Puphpet tree" src="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_035.png" width="365" height="509" srcset="https://blog.armesto.net/wp-content/uploads/2014/04/Selección_035.png 365w, https://blog.armesto.net/wp-content/uploads/2014/04/Selección_035-215x300.png 215w" sizes="(max-width: 365px) 100vw, 365px" />](https://blog.armesto.net/wp-content/uploads/2014/04/Selección_035.png)

Una vez que tenía todo en su sitio, tan solo me quedaba ejecutar los scripts **en el mismo orden** que <a title="Vagrant" href="http://www.vagrantup.com/" target="_blank">Vagrant</a> lo hace. Acordaos de dar permisos de ejecución a esos ficheros. El orden es el siguiente:

<pre>chmod +x /vagrant/puphpet/shell/*.sh
sudo ./vagrant/puphpet/shell/initial-setup.sh /vagrant/puphpet
sudo ./vagrant/puphpet/shell/update-puppet.sh
sudo ./vagrant/puphpet/shell/r10k.sh</pre>

Esto instalará todas las dependencias necesarias para que podamos empezar a utilizar <a title="Puppet" href="http://puppetlabs.com/" target="_blank">puppet</a> en producción. Así que una vez finalizado, **solo queda decirle a puppet que aplique los cambios del manifiesto**.

<pre>sudo puppet apply --debug --verbose --hiera_config /vagrant/puphpet/puppet/hiera.yaml --parser future /vagrant/puphpet/puppet/manifest.pp</pre>

Este comando hará todo lo que hemos pedido en el manifiesto. Para asegurarnos de que todo sigue en orden en el servidor, podemos crear un cron que lo ejecute cada cierto tiempo, o tan solo ejecutarlo manualmente cuando añadimos o borramos algo al manifiesto.

Así que con esto tengo la misma configuración en mi máquina virtual de desarrollo que en mi máquina de producción. Y es una configuración que he ido seleccionando a través de una cómoda interfaz web gracias a <a title="Puphpet" href="https://puphpet.com/" target="_blank">Puphpet</a>. Épico.
