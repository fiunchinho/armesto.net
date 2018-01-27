---
author: fiunchinho
categories:
- Opinion
date: 2014-05-01T13:27:28Z
dsq_thread_id:
- "2652533223"
guid: https://blog.armesto.net/?p=92
id: 92
tags:
- tdd
- unit testing
title: El problema con el code coverage
url: /el-problema-con-el-code-coverage/
---

Me he permitido el lujo de <a title="Considered harmful" href="http://en.wikipedia.org/wiki/Considered_harmful" target="_blank">parafrasear a los maestros</a> en el título de este artículo para hablaros de un tema algo polémico, por lo menos en los círculos en los que lo he hablado. Se trata de la necesidad de una alta cobertura de código, o _code coverage_.

El tema me lo recordó una serie de tweets que intercambiamos el otro día algunos en Twitter, hablando sobre la importancia de tener una cobertura de código del 100%. En el fondo todos estábamos de acuerdo en que no es importante tener un 100%, aunque no es la opinión más habitual que leerás por internet o escucharás en empresas. Mi opinión es que exigir cierto porcentaje de cobertura de código no es que no aporte nada, **sino que es algo perjudicial**.

<!--more-->

<blockquote class="twitter-tweet" width="550">
  <p>
    .<a href="https://twitter.com/SergiGP">@SergiGP</a> <a href="https://twitter.com/theUniC">@theUniC</a> Tener un porcentaje alto de code coverage es una consecuencia, no un objetivo. No hay que olvidarlo nunca.
  </p>
  
  <p>
    &mdash; Jose Armesto (@fiunchinho) <a href="https://twitter.com/fiunchinho/statuses/459038699079344128">April 23, 2014</a>
  </p>
</blockquote>



&nbsp;

## ¿Qué es la cobertura de código?

La cobertura de código es una métrica que nos dice qué líneas de nuestro código ha sido ejecutado tras lanzar un test. De esta forma, yo puedo saber si hay partes de mi código que no están siendo testeadas. Este porcentaje se utiliza para medir la salud de un proyecto ya que si tiene un porcentaje bajo, podemos afirmar que hay muchas partes del código de las que no podemos estar seguros de si funcionan o no.

Una vez que sabemos qué es, la siguiente pregunta sale de forma natural.

## ¿Cuanta cobertura de código es _necesaria_?

Nótese el énfasis en la palabra &#8220;necesaria&#8221;. Y aquí es donde viene el problema. Muchos dirán que es una locura subir un código a producción que no llegue al 80% de cobertura. Otros te dirán incluso que 90%. Y siempre encontrarás al fanático que no programa ni el vídeo VHS y que dice que él no sube a producción nada que baje del 100%, porque es el porcentaje que obtiene al hacer siempre TDD.

Mi problema con todo esto es que los tests son una **herramienta de confianza**. Además de que me ayudan a <a title="Yo no soy DHH. Long live TDD" href="https://blog.armesto.net/yo-no-soy-dhh-long-live-tdd/" target="_blank">conseguir un mejor diseño haciéndome pensar en el problema antes de pensar en la solución</a>, me ayudan a detectar errores ejecutándolos después de cada refactorización. Esto hace que yo tenga toda la seguridad del mundo en refactorizar código: sé que siempre puedo lanzar los tests y ver si he roto algo. Es mi red de seguridad, mi chivato de errores. Entonces, ¿cuantos tests tengo que escribir? La respuesta es obvia: **los tests necesarios para conseguir esa confianza**. Los necesarios para decir, si están en verde es que todo está bien.

¿Cómo se mide esa confianza en un porcentaje de code coverage equivalente? **No se puede**. No se puede porque depende del código que estés haciendo. Si esa clase tiene un _getter_ que lo único que hace es devolver una propiedad y nada más, no escribiré un test para ese método. Si ese otro método es complejo y posiblemente cambiará, créeme que lo testearé concienzudamente. La mejor metáfora que he visto sobre esto es la publicada en 2007, respondiendo a esta misma pregunta <a title="How much code coverage do you need?" href="http://www.developertesting.com/archives/month200705/20070504-000425.html" target="_blank">How much test coverage do you need?</a>:

<div style="border-left: 5px solid #edece4">
  <p style="padding-left: 30px;">
    <em>Early one morning, a programmer asked the great master:</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“I am ready to write some unit tests. What code coverage should I aim for?”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The great master replied:</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“Don’t worry about coverage, just write some good tests.”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The programmer smiled, bowed, and left.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>&#8230;</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>Later that day, a second programmer asked the same question.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The great master pointed at a pot of boiling water and said:</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“How many grains of rice should put in that pot?”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The programmer, looking puzzled, replied:</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“How can I possibly tell you? It depends on how many people you need to feed, how hungry they are, what other food you are serving, how much rice you have available, and so on.”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“Exactly,” said the great master.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The second programmer smiled, bowed, and left.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>&#8230;</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>Toward the end of the day, a third programmer came and asked the same question about code coverage.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“Eighty percent and no less!” Replied the master in a stern voice, pounding his fist on the table.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The third programmer smiled, bowed, and left.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>&#8230;</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>After this last reply, a young apprentice approached the great master:</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“Great master, today I overheard you answer the same question about code coverage with three different answers. Why?”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The great master stood up from his chair:</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“Come get some fresh tea with me and let’s talk about it.”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>After they filled their cups with smoking hot green tea, the great master began to answer:</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“The first programmer is new and just getting started with testing. Right now he has a lot of code and no tests. He has a long way to go; focusing on code coverage at this time would be depressing and quite useless. He’s better off just getting used to writing and running some tests. He can worry about coverage later.”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“The second programmer, on the other hand, is quite experience both at programming and testing. When I replied by asking her how many grains of rice I should put in a pot, I helped her realize that the amount of testing necessary depends on a number of factors, and she knows those factors better than I do – it’s her code after all. There is no single, simple, answer, and she’s smart enough to handle the truth and work with that.”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“I see,” said the young apprentice, “but if there is no single simple answer, then why did you answer the third programmer ‘Eighty percent and no less’?”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The great master laughed so hard and loud that his belly, evidence that he drank more than just green tea, flopped up and down.</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>“The third programmer wants only simple answers – even when there are no simple answers … and then does not follow them anyway.”</em>
  </p>
  
  <p style="padding-left: 30px;">
    <em>The young apprentice and the grizzled great master finished drinking their tea in contemplative silence.</em>
  </p>
</div>

## Problemática

El problema de exigir un code coverage mínimo a los programadores, de proveer una respuesta simple a la pregunta, hace que el foco de los tests se centre en llegar a ese número mágico que nos hemos sacado de la manga, cuando el foco debería estar orientado a pensar qué tests son realmente importantes.

> Una cobertura baja nos indica si un código está mal testeado, pero una alta no nos dice que un código esté bien testeado. De la misma forma que un test en rojo nos dice que hay un error, pero uno en verde no nos asegura la ausencia de errores.

La motivación para exigir un code coverage mínimo creo que sale de <a title="Ni hombres lobo ni balas de plata" href="https://blog.armesto.net/ni-hombres-lobo-ni-balas-de-plata/" target="_blank">la necesidad de buscar respuesta a preguntas complejas</a> del desarrollo de software. Centrándonos en llegar al mínimo de cobertura de código, puede hacer que un comportamiento del programa se nos escape e introduzcamos un bug en el sistema, porque estábamos demasiado ocupados escribiendo tests sin valor para pasar por las líneas que nos exigían.

## Conclusión

Escribir tests es lo mejor que puedes hacer, y soy un gran defensor de utilizar TDD, pero como dijo Kent Beck en su <a title="Kent Beck on code coverage" href="http://stackoverflow.com/a/153565/563072" target="_blank">famosísima respuesta de Stack Overflow</a>:

> Me pagan por escribir código que funciona, no tests.

Tener una alta de cobertura no es un objetivo a tener, sino que es una consecuencia de haber pensado bien en el problema que estamos solventando y en los comportamientos que son interesantes para testear. Los que no lo sean, no tenemos por qué testearlos.
