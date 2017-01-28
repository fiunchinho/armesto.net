---
author: fiunchinho
categories:
- Code
date: 2017-01-28T20:00:31Z
tags:
- code
- ddd
- performance
title: Stackless Exceptions
url: /stackless-exceptions/
thumbnail: "images/stack_trace.png"
---

Este es un post corto donde os enseñaré qué son y por qué son utiles las Stackless Exceptions, o excepciones que no contienen stack trace.

<!--more-->

## Controlando el flujo de la aplicación con excepciones
Digo que es un post corto porque podríamos entrar en el debate sobre si es buena práctica o no el utilizar excepciones en el código para dirigir el flujo de la aplicación. Con esto me refiero, por ejemplo, a lanzar una excepción cuando el usuario intenta gastarse más dinero del que tiene en la cuenta, y que una clase de otra capa se encargue de capturar esa excepción.
Yo es un patrón que sí utilizo, aunque soy consciente de que mucha gente cree que es un anti patrón, argumentando que es más difícil seguir el flujo del programa, y que es más caro lanzar excepciones que simplemente devolver errores. De hecho, [el lenguaje Golang no permite lanzar excepciones](https://golang.org/doc/faq#exceptions), sino que cada llamada a una función devuelve dos valores: el valor en caso de éxito, y el valor en caso de error. Con esto se intenta conseguir que el flujo de la aplicación sea lineal y fácil de seguir.

Aunque como digo, no voy a entrar en ese debate, sí es cierto que una de los argumentos [es el coste de lanzar una excepción](http://stackoverflow.com/a/567593/563072).
Si esto es lo que nos preocupa, hay una manera de lanzar excepciones que las hace practicamente gratis: las **stackless exceptions**.

Muchas veces no nos importa dónde se haya lanzado una excepción. Lo que nos importa realmente es que se ha lanzado. Siguiendo con el ejemplo antes mencionado, imaginad una excepción de negocio que se lanza cuando el usuario intenta realizar un pago pero no tiene suficiente dinero en su cuenta.
Normalmente lanzamos esa excepción para capturarla en una capa superior de la aplicación, pero no es una excepción que necesitemos saber dónde se ha lanzado. Por tanto, puedo ahorrarme el stack trace, que, precisamente, es lo que más cuesta a la hora de construir excepciones.

Esta excepción es igual que cualquier otra, con la única diferencia de que no tiene stack trace.

## Creando Stackless Exceptions
Aquí vemos un ejemplo de una excepción que no tendrá información de stack

```java
public class NotEnoughMoneyException extends Exception {

    public StacklessException(String message) {
        super(message);
    }

    /**
     * Does not fill in the stack trace for this exception
     * for performance reasons.
     *
     * @return this instance
     * @see java.lang.Throwable#fillInStackTrace()
     */
    @Override
    public Throwable fillInStackTrace() {
        return this;
    }
}

```

Lo que hacemos para crear la excepción `NotEnoughMoneyException`, es sobreescribir el método `fillInStackTrace()`, que es el encargado de guardar esa información. Al sobreescribirlo y que no haga nada, no tendremos stack trace.

Incluso podrías tener la excepción llamada `StacklessException`, y hacer que las excepciones de negocio donde no te interesa el stack trace, hereden de ésta, y no repetir esto en cada excepción. Aunque esto ya es al gusto de cada uno.

## Conclusión
El debate sobre si utilizar excepciones para controlar el flujo de la aplicación sigue ahí, pero el argumento del coste de las excepciones desaparece con esta técnica, donde quitamos el peso del stack trace.
