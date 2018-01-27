---
author: fiunchinho
categories:
- Opinion
date: 2014-05-23T20:35:44Z
dsq_thread_id:
- "2707892574"
guid: https://blog.armesto.net/?p=108
id: 108
tags:
- tdd
- unit testing
title: 'Mis impresiones sobre el debate #isTDDDead'
url: /mis-impresiones-sobre-el-debate-istdddead/
thumbnail: https://25.media.tumblr.com/45bb569edfcf14a03e9448397cd4ff27/tumblr_mh6v96548x1qhoi9qo1_r1_500.jpg
---

El <a title="Yo no soy DHH. Long live TDD" href="https://blog.armesto.net/yo-no-soy-dhh-long-live-tdd/" target="_blank">debate sobre si TDD está muerto</a> o no sigue _vivito y coleando_, y tras una guerra fría de artículos por ambas partes, ahora el intercambio de opiniones se ha pasado a un formato de <a title="isTDDDead" href="http://martinfowler.com/articles/is-tdd-dead/" target="_blank">vídeo debate</a> donde Martin Fowler, Kent Beck y el mismísimo David Heinemeier Hansson hablan sobre el tema.

La verdad es que cuando anunciaron que Kent Beck iba a enfrentarse a David Heinemeier sobre el tema TDD, me alegré bastante por ver a gente tan buena en esto que hacemos debatiendo sobre distintas metodologías. &#8220;_Seguro que aprendo de ambas partes_&#8220;, pensaba yo.

Ingenuo de mí.

<!--more-->

Tres _hangouts_ despues, la idea general que para mí resume el debate es que DHH dice que TDD puede desencadenar en mal código. El problema fundamental que tiene su argumento es que esto no es exclusivo de TDD. Hacer TDD te puede llevar a un mal código de la misma manera que utilizar mal cualquier otra herramienta podría llevarnos por el mal camino.

## Tests como herramientas de diseño

A veces parece que lo que no le gusta de TDD y de los tests unitarios, es que a diferencia de los tests de sistema que él prefiere, los unitarios son tests que no te ocultan los problemas que tiene tu código. **No puedes crear un tests en aislamiento cuando tu clase tiene mil dependencias y no cumples cosas como la ley de Demeter o los principios SOLID**. Sin embargo, los tests de sistema no se quejarán en absoluto. No son nada exigentes en ese sentido. A cambio, ¿qué pierdes? Pierdes feedback instantáneo sobre cómo de bien (o mal) estás diseñando algo. Pierdes velocidad en los tests que pasan de segundos a minutos.

Eligiendo el camino de los tests de sistema puede dar la impresión de que vamos más rápido porque tenemos que pensar menos los tests, pero lo que realmente estamos haciendo es **esconder el polvo debajo de la alfombra**. El problema es que llegará un día que la alfombra no será suficiente, y cuando nos queramos dar cuenta, **las termitas han tomado el lugar**.

## Arquitectura Hexagonal

Otro argumento que me chocó de DHH, que menciona en el segundo _hangout_, es ese que dice que TDD penaliza el diseño porque tiendes a cosas como la arquitectura hexagonal. Para empezar, no entiendo qué relación tiene una cosa con la otra. Puedes hacer una arquitectura hexagonal sin hacer TDD y viceversa.

Además, dice que elegimos utilizar arquitectura hexagonal porque nos ayuda a crear tests. Que el principal motivo de este tipo de arquitectura es el testing.

En <a title="Arquitectura Hexagonal" href="https://www.youtube.com/watch?v=vX5PBaXopmg" target="_blank">mi charla sobre arquitectura hexagonal</a> (<a title="Arquitectura Hexagonal" href="https://speakerdeck.com/fiunchinho/hexagonal-architecture" target="_blank">slides</a>), no se mencionan los tests hasta la slide número 55 (son 60). Y lo que digo es que la facilidad de hacer tests con este tipo de arquitecturas, es un buenísimo efecto secundario que tendremos. Pero nunca se vende que sea el principal motivo. Cierto es que en <a title="Hexagonal Architecture" href="http://alistair.cockburn.us/Hexagonal+architecture" target="_blank">el artículo original</a> se hace más hincapié en los tests, pero para mí la principal ventaja es **la independencia de herramientas**: frameworks, bases de datos, etc,. además de promocionar al dominio a un sitio más visible y acorde a la importancia que tiene.

## Conclusión

Me veré el último hangout, porque siempre se puede aprender algo de estas tres fieras, pero para mi, el debate no se ha llevado en ningún momento por cauces interesantes. Prefiero no pensarlo, pero ya van varias personas que mencionan que DHH solo pretendía hacer ruido. Espero que no fuese así.
