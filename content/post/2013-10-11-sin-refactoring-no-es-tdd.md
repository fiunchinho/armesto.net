---
author: fiunchinho
categories:
- Opinion
date: 2013-10-11T17:57:56Z
dsq_thread_id:
- "2650288247"
guid: http://armesto.net/?p=8
id: 8
tags:
- refactoring
- tdd
title: Sin refactoring no es TDD
url: /sin-refactoring-no-es-tdd/
---

Parece evidente, ¿verdad? Cuando explicas <a title="TDD" href="http://c2.com/cgi/wiki?TestDrivenDevelopment" target="_blank">TDD (Test Driven Development)</a>, explicas que tiene 3 fases: rojo, verde y refactoring. Sin embargo, parece que son las 2 primeras las que entran con mayor facilidad en la mente del programador.

Presentas la kata y allí van disparados a escribir su primer test para estar en rojo. Una vez allí, _baby step_ en el código con lo mínimo indispensable para pasar el test y ponernos en verde. Es entonces cuando deberíamos pensar en el refactoring, pero normalmente es difícil encontrar cosas a refactorizar en nuestro (poquito) código en las primeras iteraciones, así que pasamos a rojo otra vez escribiendo un nuevo test.

Este proceso se repite de forma rápida durante los primeros tests, y con la excusa de que con pocos tests es difícil encontrar duplicaciones o cosas a mejorar en el código que hace pasar estos tests, poco a poco entramos en el modo semárofo del rojo-verde.

<!--more-->

Los más aventajados del lugar se aventuran a mejorar alguna cosilla cuando están en verde, pero suele ser algún cambio pequeño sin importancia, mientras la casuística de if’s y else’s sigue creciendo sin que nadie haga nada por evitarlo. Se olvidan los principios <a title="SOLID" href="http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod" target="_blank">SOLID</a>, como si fuesen principios teóricos que poco o nada tienen que ver con la práctica. “La teoría no es lo mío, yo soy más práctico”: ¿os suena?

Como facilitador, solo te queda realizar las preguntas adecuadas:

  * ¿si quiero ampliar el funcionamiento con una **nueva** regla de negocio, tendré que cambiar mi clase? ¿cómo de **costoso** será el cambio?
  * ¿está ese comportamiento bien **encapsulado**? ¿hay suficiente **abstracción**?
  * ¿hace esta clase una y solo una cosa?

Es sobretodo con la primera pregunta con la que la gente se da cuenta de los problemas de su diseño actual, quizá porque es la pregunta más “práctica”, por así decirlo. En el fondo son los mismos principios, pero forzando a la gente a pensar en un caso práctico, por ejemplo dando tú mismo nuevas e imprevistas reglas de negocio, la gente entiende por qué es mejor seguir <a title="SOLID" href="http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod" target="_blank">SOLID</a>.

Estas preguntas hay que realizarlas desde muy temprano. Si la gente va posponiendo el refactoring, cada vez es más difícil cambiar el diseño actual porque cada vez habrá más reglas de negocio en el código, y cada vez habrá más tests que quizá haya que retocar, si el refactoring más que un simple refactoring.

También se tiende a olvidar que en ese momento tenemos un conocimiento grande del dominio, y podemos plasmar las reglas del dominio de una forma que ahora entendemos, pero quizá no entendamos en el futuro. Justo ahora es el mejor momento para dar nombres que expresen la intención del código. Por ejemplo: introducir condiciones en métodos o variables con nombres expresivos.

Una de las grandes ventajas del TDD, es que el refactoring está dentro del ciclo de desarrollo. Si no lo hacemos frecuentemente, se pierde el efecto.

Sin refactoring no es TDD.