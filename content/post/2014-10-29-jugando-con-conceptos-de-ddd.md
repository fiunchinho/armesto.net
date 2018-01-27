---
author: fiunchinho
categories:
- Code
date: 2014-10-29T00:46:39Z
dsq_thread_id:
- "3166973431"
guid: https://blog.armesto.net/?p=119
id: 119
tags:
- ddd
title: Jugando con conceptos de DDD
url: /jugando-con-conceptos-de-ddd/
thumbnail: https://images-na.ssl-images-amazon.com/images/I/51sZW87slRL._SX375_BO1,204,203,200_.jpg
---

El pasado fin de semana se celebró en Barcelona una <a title="Software Craftmanship Barcelona" href="http://www.softwarecraftsmanshipbarcelona.org/" target="_blank">nueva edición de la Software Craftmanship</a>, donde se hablaron de diversos temas relacionados con las buenas prácticas a la hora de crear software. Entre ellos, en mi opinión, hubo uno que pareció suscitar mayor interés en la gente y ocupó un papel protagonista en los dos días de conferencia: **Domain Driven Design**. Es un tema en el que creo que muchos estamos todavía aprendiendo, intentando dar sentido a la inmensa cantidad de conceptos e información que aparecen tanto en <a title="DDD - Eric Evans" href="http://www.amazon.es/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215" target="_blank">el libro azul</a> como en <a title="DDD - Vaughn Vernon" href="http://www.amazon.es/gp/product/0321834577/ref=pd_lpo_sbs_dp_ss_1?pf_rd_p=479290847&pf_rd_s=lpo-top-stripe&pf_rd_t=201&pf_rd_i=0321125215&pf_rd_m=A1AT7YVPFBWXBL&pf_rd_r=1N6RWXVM4AFMHH9P0EGZ" target="_blank">el rojo</a>.

En esta búsqueda de sentido, me surgió una duda que creo que no aparece resuelta directamente en ninguno de los dos libros, y que compartí con el resto de asistentes en una de las charlas sobre DDD. Quiero reproducirla otra vez aquí, para poder hablar un poco más sobre el tema. Creo que no hace falta mostrar código, pero realmente me gustaría vuestra opinión, así que si es necesario, decídmelo y añadiré código.

<!--more-->

## Creando value objects a partir de entidades

Mi duda viene a la hora de combinar dos de los objetos básicos de DDD: las **entidades** y los **value objects**. En todos los ejemplos que he visto, los value objects pueden ser simples objetos que no dependen de ningún otro, o que se construyan utilizando otros value objects. Por su parte, las entidades se construyen tradicionalmente sin dependencias, o utilizando algún value object. Mi pregunta es: **¿hay algo que nos impida construir value objects que dependan de entidades, que reciban entidades en su constructor?**

Cuando lancé esta pregunta durante la sesión de DDD Táctico que impartía <a title="Christian Soronellas" href="https://twitter.com/theUniC" target="_blank">Christian</a>, la mayoría de gente me miró como si fuese un loco (solo <a title="Carlos Ble" href="https://twitter.com/carlosble/" target="_blank">Carlos</a> parecía encontrar sentido a mis palabras), pero dejadme explicar por qué creo que construir value objects con entidades puede ser perfectamente válido.

Un value object es un objeto el cual su identidad viene dada por el valor que contiene. No tiene un identificador como tal, porque su propio valor lo identifica. Dos value objects que contienen el mismo valor, son iguales.

Un ejemplo de caso de uso para mi pregunta podría ser cuando intentamos modelar una aplicación para votar en unas elecciones. Una práctica que suelo utilizar es la de intentar modelar todo el dominio con value objects, y solo &#8220;promocionar&#8221; el objeto a entidad cuando es estrictamente necesario. En este caso, si tenemos ciudadanos que votan, y partidos políticos que pueden ser votados, veo lógico que esos sean entidades, ya que un ciudadano puede hasta cambiarse el nombre y seguir siendo el mismo ciudadano. **Tiene una identidad que lo identifica** (valga la redundancia). Con el partido político pasa algo parecido: podría cambiar su logotipo, o hasta su nombre, y seguir siendo el mismo.

Pero para modelar los votos que la gente hace a los partidos, quise intentar hacerlo con un value object. Este value object debe contener qué ciudadano ha votado, a qué partido político, y quizá el momento exacto en el que ha votado. Si decíamos que tanto los ciudadanos como los partidos políticos son entidades&#8230; **este objeto &#8220;voto&#8221; tendría que construirse a partir de entidades**.

## Inmutabilidad del value object

No recuerdo quién, argumentó que, por definición, los value objects son inmutables, y si un ciudadano se cambiase el nombre entonces el voto habría cambiado, perdiendo su inmutabilidad. Pero esto no es cierto. Precisamente, debido a que las entidades tienen identidad, la forma de decir si una entidad es o no igual a otra, es comparando los identificadores de esas entidades. Si un ciudadano se cambia el nombre, sigue siendo el mismo ciudadano. Por tanto, si un value object _Voto_ contiene una entidad _Ciudadano_, y este ciudadano se cambia el nombre, **el value object sigue sin haber mutado, ya que sigue conteniendo la misma entidad**: la identidad del ciudadano es la misma.

Si cambiase el ciudadano que ha votado, o el partido político al que ha votado, **sería otro voto**. Es el valor del value object lo que le identifica. Es inmutable: yo no puedo coger la papeleta de un voto y tachar una cosa para poner otra.

Alguien podría preguntarme que por qué complicarme la vida y no hacer una entidad. Creo que modelar con value objects es más simple, manteniendo la inmutabilidad lo máximo posible. Estos votos podrían formar parte de una entidad de elecciones (o algo parecido), que sería lo que finalmente persistiría esos votos en base de datos. Cuantas menos entidades, más sencillo de programar y más sencillo de razonar. ¿Para qué añadir identidades a cosas que no las necesitan?

Ya sé que no es un ejemplo que aparezca en los libros, pero a mí es un diseño que a priori me cuadra. Solo lo he desarrollado en mi cabeza, así que no sé con qué problemas me podría encontrar. Como soy masoca, lo expongo aquí públicamente para que me lo crujáis.

**¿Qué os parece?**
