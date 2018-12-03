---
author: fiunchinho
categories:
- Opinion
date: 2014-04-24T00:35:31Z
dsq_thread_id:
- "2648449214"
guid: https://blog.armesto.net/?p=83
id: 83
tags:
- tdd
- unit testing
title: Yo no soy DHH. Long live TDD
url: /yo-no-soy-dhh-long-live-tdd/
---

DHH es un gran programador, creador de algo como <a title="Rails" href="http://rubyonrails.org/" target="_blank">Rails</a>, framework que quedará en la historia de la programación web. Hoy, él, ha sido noticia por escribir <a title="TDD is dead" href="http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html" target="_blank">un post titulado &#8220;TDD is dead&#8221;</a>.

Quizá DHH está en lo cierto diciendo que TDD es algo que está muerto, que es momento de seguir hacia adelante y dejarlo atrás. Como cuando crecemos y dejamos de utilizar los <a title="ruedines de bici" href="http://1.bp.blogspot.com/_95Yb4E_y8Cs/S_HClbzs9YI/AAAAAAAADVs/iiwdG-vqq3Y/s1600/orbea-bicicleta-kids-atlantis-14.jpg" target="_blank">ruedines de la bicicleta</a>: es algo para niños pequeños, pero de mayores ya no nos hacen falta. Quizá tiene razón.

Si eres lo suficientemente experto, entonces quizá ya no tiene sentido. Porque para mí TDD es eso: unos ruedines para programar que me ayudan a conseguir un mejor código. Si fuese capaz de escribir el mejor código posible a la primera, que siguiese los principios SOLID, que fuese auto explicativo, etc., entonces yo tampoco haría TDD.  **¿Para qué molestarme?**

Si me saliese a la primera, no necesitaría refactorizar muy frecuentemente y tener un unit testing que me avise cuando he roto algo. No me haría falta ese loop de feedback tan corto.

<!--more-->

## No todos somos DHH

El problema es que poca gente tiene la capacidad de hacerlo así de bien siempre. Sobre todo porque normalmente programamos cosas que nunca antes hemos programado. Si estoy haciendo algo que ya hice con anterioridad, quizá no es tan importante seguir un TDD estricto. Ahí tomaré atajos.
  
Pero si lo que estoy haciendo es completamente nuevo y no sé por donde atacarlo, créeme que haré TDD sin dudarlo, **siguiendo todos y cada uno de los pasos**.

Como la mayoría de la gente no tiene esa capacidad que decía antes, la metedura de pata del post de DHH me parece mayúscula. Me lo parece porque no solo dice que TDD no es para él, sino que anima a la gente a no utilizarlo, y deja entrever que hará cambios en rails para animar a la gente a hacer menos unit testings y más tests de otro tipo.

Teniendo en cuenta su posición de lider en la comunidad, me parece una temeridad por su parte. Está invitando a la gente a que rompa algo tan establecido y aprobado como la <a title="Testing Pyramid" href="http://martinfowler.com/bliki/TestPyramid.html" target="_blank">pirámide de testing</a>, basándose en que no han sido tests útiles para testear aplicaciones basadas en rails. Un vistazo rápido a las charlas de las conferencias de Ruby de los últimos años y vemos como hay un tema recurrente en todas ellas: <a title="Deconstructing the framework" href="https://www.youtube.com/watch?v=iUe6tacW3JE" target="_blank">cómo escapar de Rails</a>. Bien sea por hacer <a title="Fast tests" href="https://www.youtube.com/watch?v=bNn6M2vqxHE" target="_blank">tests más rápidos</a> y tener feedback antes; o porque los modelos de mi aplicación no <a title="ActiveRecord" href="https://www.youtube.com/watch?v=yuh9COzp5vo" target="_blank">estén entre mezclados con el sistema de persistencia</a>; o porque quiero tener a<a title="Hexagonal Rails" href="https://www.youtube.com/watch?v=CGN4RFkhH2M" target="_blank">rquitecturas que me permitan intercambiar componentes fácilmente</a>.
  
Siempre lo que se busca es escapar de Rails, minimizar la dependencia con el framework. Los unit testings son costosos de hacer cuando el framework está en el medio. Por eso no le han sido útiles: porque su framework no permitía que lo fuesen. La solución, obviamente, no es dejar de hacerlos, sino arreglar el framework.

<blockquote class="twitter-tweet" width="550">
  <p>
    Sit-ups are dead. They don’t work when I eat all this sugar and take on all this severe stress. Long live gastric bypass surgery.
  </p>
  
  <p>
    &mdash; ☕ J. B. Rainsberger (@jbrains) <a href="https://twitter.com/jbrains/statuses/458983164502093824">April 23, 2014</a>
  </p>
</blockquote>



## El Dogma

De todas formas sí que comparto con él una cosa, y es el nivel de dogma que TDD ha alcanzado en los últimos años. Personas como Uncle Bob, cuando hablan de TDD, se ponen una sotana encima para predicar la palabra sagrada. Uncle Bob ha sido capaz de convertir la práctica de TDD en una religión.

<div style="width: 213px" class="wp-caption alignleft">
  <img alt="Uncle Bob, el predicador" src="/images/uncle_bob.jpg" width="203" height="214" />
  
  <p class="wp-caption-text">
    Uncle Bob, el predicador de Texas
  </p>
</div>

Es una religión porque al igual que las religiones, el TDD de Uncle Bob promete la salvación eterna si cumples con los mandamientos (<a title="3 rules of TDD" href="http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd" target="_blank">3 en el caso del TDD</a>). Pero no solo eso, sino que también asegura el castigo eterno para aquellos que no lo hagan. Eso es lo que hace cuando alude al poco profesionalismo de la gente que no lo practica. Está señalando a los herejes.

En mi opinión, **hacer TDD es una herramienta más**. Una herramienta casi imprescindible para mi, pero una herramienta al fin y al cabo. A alguien que utilizase el bloc de notas para programar le recomendaría que probase Sublime o PHPStorm. Al igual que le recomendaría TDD si no lo utiliza. En ambos casos creo que esa persona será más productiva y hará mejor código. Por tanto será un programador más rentable. ¿Quiere decir esto que no se puede ser rentable si no se usa TDD? Claro que no, al igual que también puedes ser un crack en el bloc de notas.

> Los defensores de TDD también debemos hacer auto-crítica y pensar en si la forma que elegimos para comunicar las bondades de TDD es la más correcta.

## **Conclusión**

Para mi su post de hoy ha sido una gran metedura de pata. Los unit testings aportan muchísimo valor a la hora de detectar errores, y el TDD es una de las mejores herramientas para que nuestro diseño sea mejor.

Yo no soy DHH. No he inventado Rails ni soy el CTO de <a title="Basecamp" href="https://basecamp.com/" target="_blank">una gran compañía</a>. Yo seguiré utilizando TDD para que me ayude a hacer el mejor código posible.
