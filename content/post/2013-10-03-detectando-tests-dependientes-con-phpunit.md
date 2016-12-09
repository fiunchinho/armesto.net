---
author: fiunchinho
categories:
- Code
date: 2013-10-03T18:07:55Z
dsq_thread_id:
- "2662518483"
guid: http://armesto.net/?p=20
id: 20
tags:
- php
- unit testing
title: Detectando tests dependientes con PHPUnit
url: /detectando-tests-dependientes-con-phpunit/
---

A día de hoy, <a title="PHPUnit" href="https://github.com/sebastianbergmann/phpunit/" target="_blank">PHPUnit</a> es el standard _de facto_ para escribir pruebas unitarias en PHP. En estas pruebas unitarias, intentamos que los tests sean **totalmente aislados**, es decir, que no tengan efectos secundarios, que no se conecten a servicios como API’s o bases de datos, y que no dependan unos de otros.

Sin embargo, cuando estamos escribiendo tests de integración para comprobar que ciertas clases se comunican correctamente entre ellas, o con otros servicios (base de datos, API’s, etc), los tests dejan de ser aislados, porque esa conexión es precisamente lo que queremos probar. Además, puede que tengan efectos secundarios porque, siguiendo con el ejemplo de la base de datos, vamos a comprobar si se han escrito datos correctos o si se han actualizado como deberían.

<!--more-->

Lo que sí vamos a querer mantener, salvo que realmente nos haga falta lo contrario, es que los tests **no sean dependientes entre sí**. Es decir, que el orden en el que se ejecutan sea importante. No queremos que para que el `testB()` pase en verde, deba ejecutarse siempre después del `testA()`, y si esto no ocurre el test falle. Esto podría pasar por ejemplo cuando en el `testB()` leemos de base de datos algo que estamos insertando en el `testA()`. Si los cambiamos de orden, el `testB()` no encontrará los registros, y el test fallará irremediablemente.

Si conseguimos que los tests no compartan estados y que sean independientes, evitaremos que cuando alguien cambie el orden de los tests estos empiecen a fallar de forma aleatoria. En la vida real, sin embargo, estamos resolviendo problemas complejos y  podríamos meter la pata. El problema es que cuando la cantidad de código que tienes empieza a ser muy grande, se hace complicado saber si alguien ha metido la pata en algún test y lo ha hecho dependiente de otro.

Para solucionar esto, <a title="phpunit-randomizer" href="https://github.com/fiunchinho/phpunit-randomizer" target="_blank">he creado una librería</a> que podéis utilizar para lanzar los tests de PHPUnit de **forma aleatoria**. Es decir, los test cases dentro de una clase de test se ejecutarán en un orden aleatorio, y si alguno dependía de la acción de otro, fallará y se pondrá de manifiesto que tiene que arreglarse.

PHPUnit ya ofrece una opción para que cada test case se lance en un proceso de php independiente, llamada _process-isolation_, pero esto provoca que los tests tarden mucho en ejecutarse, y cuando el número es elevado, el tiempo de espera se hace demasiado.

La idea con mi librería es **no tener que modificar código de PHPUnit** para que podamos seguir actualizando el framework sin miedo a perder cambios. Por tanto todo son clases nuevas y hasta un ejecutable nuevo, que nos evita tocar nada de PHPUnit.

PHPUnit nos ofrece una serie de <a title="Extending PHPUnit" href="http://phpunit.de/manual/3.7/en/extending-phpunit.html" target="_blank">mecanismos para añadir o cambiar funcionalidad existente del framework</a>, pero realmente **son una mierda **(por lo menos en su versión actual 3.7). Nos permite extender clases para sobrescribir comportamiento, pero no da un mecanismo fácil (por configuración, por ejemplo) para decirle que utilice las nuevas clases que acabamos de crear. Debido a esto, la solución final implica que haya que utilizar un nuevo ejecutable incluído en mi librería llamado `phpunit-randomizer`, en vez del `phpunit` que viene por defecto con el framework.

Podéis coger la librería <a title="PHPUnitRandomizer" href="https://github.com/fiunchinho/phpunit-randomizer" target="_blank">PHPUnitRandomizer en github</a>.