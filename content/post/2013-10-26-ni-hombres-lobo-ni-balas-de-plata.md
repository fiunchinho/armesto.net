---
author: fiunchinho
categories:
- Opinion
date: 2013-10-26T17:56:47Z
dsq_thread_id:
- "2648858287"
guid: http://armesto.net/?p=6
id: 6
title: Ni hombres lobo ni balas de plata
url: /ni-hombres-lobo-ni-balas-de-plata/
---

Durante años, programadores de todos los lenguajes han buscando sin descanso la respuesta a todos sus problemas. Una metodología, un principio. Algo que termine de un plumazo con las complicaciones a la hora de programar. A esta respuesta se le llama comúnmente **la bala de plata**.

Muchas veces es costoso tener que decidir si este principio es mejor que ese otro; o si es mejor aplicar tal patrón de diseño en vez de otro distinto. Os suenan cosas como:

_&#8211; Siempre tienes que tener 100% de code coverage._

_&#8211; Nunca deberías utilizar un ORM._

La buena noticia que quiero compartir hoy contigo es que no tienes por qué elegir. No tiene sentido hacerlo. ‘X’ no es mejor que ‘Y.’ Pero tampoco ‘Y’ es mejor que ‘X’.

<!--more-->

En la programación todas las decisiones son un **_tradeoff_**: hay pros y hay contras, y es tu responsabilidad conocerlas y decidir en consecuencia. No elegir algo porque alguien lo haya dicho, o peor aún, porque sí.

No esperes a que alguien te diga cómo se resuelve algo. Pregúntale qué ganas haciéndolo de esa forma, y a qué renuncias haciéndolo. Entiéndelo y entonces, solo entonces, decide.

Somos vagos por naturaleza. Por eso seguimos a gurús en Twitter que nos digan cuál es el mejor framework; evangelistas que nos confiesen cuál la mejor metodología; o peor aún, ninjas y jedis que nos enseñen a usar la fuerza.

Como buen gallego, para mí la respuesta a todo este tipo de preguntas es siempre la misma.

¿Debería utilizar siempre Domain Driven Design? Depende.

¿Debería testear métodos privados o protegidos en mis tests? Por regla general no, pero depende.

¿Debería hacer TDD siempre? Depende.

Es muy fácil generalizar. Es como el teórico del carnet de conducir: decir SIEMPRE o NUNCA **es muy arriesgado**.

Divulgadores como <a title="Robert C. Martin" href="https://twitter.com/unclebobmartin" target="_blank">Robert C. Martin</a>, a través de su mensaje al más puro estilo predicador católico americano, consiguen que muchas veces tomemos su mensaje como leyes absolutas indiscutibles que deben aplicarse siempre. Como balas de plata.

En el caso de Robert C. Martin, es curioso como el mayor defensor del Test Driven Development ha tenido que escribir <a title="The Pragmatics of TDD" href="http://blog.8thlight.com/uncle-bob/2013/03/06/ThePragmaticsOfTDD.html" target="_blank">un post explicando los casos en los que no lo utiliza</a>. Y no hay ningún problema con ello.

El problema es poner el piloto automático aplicando todo lo que se consideran buenas prácticas sin pararse a pensar si lo es para el problema concreto que tienes delante. Pero claro, **nos es mucho más cómodo seguir unas reglas que otros nos dan, antes que pensar**.

Cosas tan aparentemente inocentes como el <a title="Don't Repeat Yourself" href="http://c2.com/cgi/wiki?DontRepeatYourself" target="_blank">DRY (Don’t Repeat Yourself)</a>, pueden tener malas consecuencias si lo aplicas ciegamente.

Los únicos principios que pueden parecer infalibles son los principios SOLID, pero he llegado a encontrarme casos donde la “Dependency Inversion” había alcanzado un nivel de locura que no era normal. Como cuando aprendes un patrón de diseño nuevo e intentas aplicarlo PARA TODO.

En definitiva: por mucho que nos cueste, todavía tenemos que seguir pensando. Aprendamos qué beneficios y desventajas tiene cada práctica y decidamos. No busquemos balas de plata.