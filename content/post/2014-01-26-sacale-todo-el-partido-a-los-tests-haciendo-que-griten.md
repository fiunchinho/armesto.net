---
author: fiunchinho
categories:
- Code
date: 2014-01-26T18:09:33Z
dsq_thread_id:
- "2648548386"
guid: http://armesto.net/?p=22
id: 22
tags:
- php
- unit testing
title: Sácale todo el partido a los tests haciendo que griten
url: /sacale-todo-el-partido-a-los-tests-haciendo-que-griten/
---

Si tuviese que decir en qué se basa una prueba unitaria, diría que las principales características que debe cumplir son (sin ningún orden en particular):

  * Que sea rápido y sin efectos secundarios.
  * Que sea realmente unitario.
  * Que sea auto explicativo.

Pero con el tiempo veo que la gente tiende a **centrarse en las dos primeras de mi lista**.

Se busca que sea rápido y sin efectos secundarios porque de esa forma podemos lanzarlos siempre que queramos, dándonos confianza. Si tuviésemos que esperar minutos en saber el resultado, o tuviésemos que andar limpiando una base de datos cada vez que quisiésemos lanzar pruebas, simplemente no lo haríamos tan frecuentemente como debiéramos.

También se prioriza que sean unitarios y aislados, para que cuando algo falle, tengamos la granularidad suficiente para identificar el problema en cuestión de segundos. Si tuviésemos una prueba que lo probase todo, el día que fallase, no sabríamos cual de las partes ha sido la culpable.

Y aunque considero que estas dos características son importantísimas, **creo que se desprecia la última de mi lista**. Las pruebas deberían ser auto explicativas, en el sentido de que debería ser muy fácil saber qué se está probando en todo momento, y sobretodo, qué hace el componente que estamos probando.

<!--more-->

La mayoría del tiempo que programamos nos la pasamos leyendo código ya existente, intentando entender qué es lo que hace. Esto implica tener que leer las pruebas. Y no sé vosotros, pero yo me he encontrado con tests infumables.

## Como mejorar la legibilidad de los tests

Voy a poner un ejemplo muy sencillo donde podemos aplicar una serie de mejoras. Quizá al ser tan simple se vea menos el efecto, pero aplicándolo a esos mastodontes que nos podemos encontrar en la vida real, creedme que mejoraremos mucho los tests.

> El refactoring de extraer método no sirve para ahorrar líneas de código, si no para que el método sea más fácil de leer.

### Primera aproximación: mocks en métodos privados

En el siguiente ejemplo, tenemos una clase encargada de identificar usuarios. Tiene dos colaboradores: el repositorio de usuarios para encontrar usuarios, y un codificador de contraseñas.

<pre><code data-lang="php">public function testShouldReturnAuthenticatedUserForValidCredentials()
{
    $email 		= 'john@doe.com';
    $encoded_pass 	= 'encoded_pass';
    $password 	        = 'decoded_pass';
    $user 		= new User( $email, $encoded_pass );
    $user_repo 	= $this-&gt;getMock( 'UserRepository' );
    $user_repo
	-&gt;expect( $this-&gt;any() )
	-&gt;method( 'findByEmail' )
	-&gt;will( $this-&gt;returnValue( $user ) );

    $pass_encoder	= $this-&gt;getMock( 'PasswordEncoder' );
    $pass_encoder
	-&gt;expect( $this-&gt;once() )
        -&gt;method( 'hash' )
	-&gt;with( $password )
	-&gt;will( $this-&gt;returnValue( $encoded_pass ) );

    $auth_service 	= new AuthService( $user_repo, $pass_encoder );

    $this-&gt;assertEquals(
	$user,
	$auth_service-&gt;authenticate( $email, $password ),
	'Must return the user when credentials are valid.'
    );
}</code></pre>

A pesar de que apenas hay lógica que testear, y gracias en parte a lo incómodo de PHPUnit (tenéis que probar <a title="PHPSpec" href="http://www.phpspec.net/" target="_blank">PHPSpec</a>, en serio), los tests empiezan a crecer, **obligándonos a leer un montón de líneas cada vez que queramos entender qué hace esto**.

Muchos programadores intentan solventar este problema extrayendo un método para, por ejemplo, la creación de mocks. Queda algo tal que así.

<pre><code data-lang="php">public function testShouldReturnAuthenticatedUserForValidCredentials()
{
    $email 		= 'john@doe.com';
    $encoded_pass 	= 'encoded_pass';
    $password 	= 'decoded_pass';
    $user 		= new User( $email, $encoded_pass );

    $this-&gt;createRepositoryMock( $email, $encoded_pass );
    $this-&gt;createEncoderMock( $password, $encoded_pass );

    $auth_service 	= new AuthService( $user_repo, $pass_encoder );

    $this-&gt;assertEquals(
	$user,
	$auth_service-&gt;authenticate( $email, $password ),
	'Must return the user when credentials are valid.'
    );
}

private function createRepositoryMock( $email, $password )
{
    $user 		= new User( $email, $password );
    $user_repo 	= $this-&gt;getMock( 'UserRepository' );
    $user_repo
	-&gt;expect( $this-&gt;any() )
	-&gt;method( 'findByEmail' )
	-&gt;will( $this-&gt;returnValue( $user ) );

    return $user_repo;
}

private function createEncoderMock( $password, $encoded_pass )
{
    $pass_encoder	= $this-&gt;getMock( 'PasswordEncoder' );
    $pass_encoder
	-&gt;expect( $this-&gt;once() )
	-&gt;method( 'hash' )
	-&gt;with( $password )
	-&gt;will( $this-&gt;returnValue( $encoded_pass ) );

    return $pass_encoder;
}</code></pre>

Aunque a priori pueda parecer que esto ha mejorado la legibilidad de nuestros tests, nada más lejos de la realidad. Lo único que hemos hecho es reducir el número de líneas del test, pero sigo teniendo que leerme todo cada vez que necesite entenderlo.

**El refactoring de extraer método no sirve para ahorrar líneas, si no para que el método sea más fácil de leer.** El nombre _createRepositoryMock()_ no me dice absolutamente nada sobre por qué lo estamos haciendo.

### Un enfoque mejor

Si el objetivo es que se entienda mejor el código, ¿por qué no hace todo explícito? Vamos a hacer lo mismo, pero pensando en el por qué de las cosas.

<pre><code data-lang="php">public function testShouldReturnAuthenticatedUserForValidCredentials()
{
    $email 		= 'john@doe.com';
    $encoded_pass 	= 'encoded_pass';
    $password 	= 'decoded_pass';
    $user 		= $this-&gt;createExistingUser( $email, $encoded_password );
    $user_repo 	= $this-&gt;createUserRepoWhenUserExists( $user );
    $pass_encoder	= $this-&gt;createEncoderWhenEncoderMustEncodePassword( $password, $encoded_pass );

    $auth_service 	= new AuthService( $user_repo, $pass_encoder );

    $this-&gt;assertEquals(
	$user,
	$auth_service-&gt;authenticate( $email, $password ),
	'Must return the valid user when credentials are valid.'
    );
}

private function createUserRepoWhenUserExists( $user )
{
    $user_repo = $this-&gt;getMock( 'UserRepository' );
    $user_repo
	-&gt;expect( $this-&gt;any() )
	-&gt;method( 'findByEmail' )
	-&gt;will( $this-&gt;returnValue( $user ) );

    return $user_repo;
}

private function createExistingUser( $email, $password )
{
    return new User( $email, $password );
}

private function createEncoderWhenEncoderMustEncodePassword( $password, $encoded_pass )
{
    $pass_encoder = $this-&gt;getMock( 'PasswordEncoder' );
    $pass_encoder
	-&gt;expect( $this-&gt;once() )
	-&gt;method( 'hash' )
	-&gt;with( $password )
	-&gt;will( $this-&gt;returnValue( $encoded_pass ) );

    return $pass_encoder;
}
</code></pre>

Estos nombres no tienen por qué ser los mejores, pero la idea es esa: hacer explícito el motivo de todo, para que cuando volvamos a leer el código (o algún compañero nuestro), todo lo que está ocurriendo se entienda a la perfección.

Este primer enfoque es bastante básico, y si vemos que se complica, siempre podemos acudir a patrones de creación como el patrón builder o factories para los mocks.

> Revelar la intención de nuestro código y el objetivo del test: lo más importante

Haciendo que el objetivo de cada línea sea revelar con claridad la intención de nuestro código y qué caso concreto estamos testeando en ese momento, reduciremos muchísimo el tiempo necesario para entenderlo. **Da igual que ocupe más líneas, o que tarde dos microsegundos más en ejecutarse**. Los objetivos de los tests son los que vimos al principio de este artículo, no son el rendimiento o tener menos líneas de código.

## Luchando contra la duplicación

He visto desarrolladores que detectan que necesitarán un stub en todos los test cases, así que sacan la creación del stub al método de setUp, para ahorrar las líneas de configuración del stub.

Volvemos a lo mismo. Debemos preguntarnos si eso mejora la legibilidad del test. En mi opinión, normalmente no lo hace, porque _esconde_ una colaboración, que puede que pasemos por alto ya que está en el setUp y no en el test case.

_“Pero entonces tendremos duplicación!”_, me dirán los lectores más atentos. Pero yo no digo que dupliquemos, sino que lo hagamos explícito. Si vemos que por hacerlo explícito vamos a duplicar código, saquemos la duplicación a un método privado que contenga la lógica que se duplica, y llamemos a este método privado desde el test case. El nombre de ese método privado nos dirá el motivo de por qué está ese código ahí.

## Conclusión

Nunca debemos olvidar que **los tests son nuestra primera documentación**. Es la mejor forma de ver cómo funciona un componente, cómo se configura, y qué valores debo esperar que devuelva. Cuando tengo que modificar código ya existente, entender qué es lo que hace es vital. Si no hago que mis tests **GRITEN** el comportamiento de mis clases, me estoy perdiendo otra de las grandes ventajas de los tests.