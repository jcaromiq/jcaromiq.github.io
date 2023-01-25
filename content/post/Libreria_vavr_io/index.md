---
title: "Vitaminando la programación funcional en java 8 con Vavr"
date: 2018-08-21T01:05:31+02:00
description: "Que nos aporta vavr sobre la programación funcional en java 8"
categories: ["Java"]
tags: ["java", "vavr", "functional programming"]
image: "pexels.jpg"
---

Con la release de java 8 se abrió un nuevo paradigma en el desarrollo con java, pero es suficiente con lo que trae de serie? Y si pudiéramos tener otras funcionalidades de lenguajes más puramente funcionales en java?

<!--more-->

Para suplir estas necesidades nació [vavr](http://www.vavr.io/) con la misión de reducir el código haciéndolo mas legible y añadir robustez con la inmutabilidad de los datos.

Vavr entre otras cosas incluye inmutabilidad en listas y funciones para trabajar con ellas, incluye algunos de los monads más utilizados en otros lenguajes más funcionales, currying y partial aplication en funciones.

## Funciones

### Composición

Con la llegada de java 8 se incluyó el concepto de Function y BiFunction, con el cual podemos definir funciones de uno o dos parámetros de entrada. por ejemplo:

```java
Function<Integer, Integer> pow = (n) -> n * n;
assertThat(pow.apply(2)).isEqualTo(4);

BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
assertThat(multiply.apply(10, 5)).isEqualTo(50);
```

Con vavr podemos tener funciones hasta 8 parámetros con los tipos Function*N*

```java
Function1<Integer, Integer> pow = (n) -> n * n;
assertThat(pow.apply(2)).isEqualTo(4);

Function3<Integer, Integer, Integer, Integer> multiply = (n1, n2, n3) -> n1 * n2 * n3;
assertThat(multiply.apply(5, 4, 3)).isEqualTo(60);
```

Ademas de poder crear funciones de hasta 8 parámetros de entrada, también nos ofrece la composición de funciones con las operaciones *.andThen* *.apply* y *.compose*

```java
Function1<String, String> toUpper = String::toUpperCase;
Function1<String, String> trim = String::trim;
Function1<String, String> cheers = (s) -> String.format("Hello %s", s);
assertThat(trim
			.andThen(toUpper)
			.andThen(cheers)
			.apply("   john")).isEqualTo("Hello JOHN");

Function1<String, String> composedCheer = cheers.compose(trim).compose(toUpper);
assertThat(composedCheer.apply(" steve ")).isEqualTo("Hello STEVE");
```

### Lifting

Con lifting lo que conseguimos es tratar con las excepciones a la hora de componer las funciones, con lo que la función devolverá un Option.none en el caso de que se haya producido una excepción y un Option.some en el caso de que haya ido todo correctamente.

Esto es muy útil a la hora de componer funciones que usan librerías de terceros que pueden devolver excepciones.

```Java
Function1<String, String> toUpper = (s) -> {
    if (s.isEmpty()) throw new IllegalArgumentException("input can not be null");
    return s.toUpperCase();
};
Function1<String, String> trim = String::trim;
Function1<String, String> cheers = (s) -> String.format("Hello %s", s);
Function1<String, String> composedCheer = cheers.compose(trim).compose(toUpper);

Function1<String, Option<String>> lifted = Function1.lift(composedCheer); assertThat(lifted.apply("")).isEqualTo(Option.none());
assertThat(lifted.apply(" steve ")).isEqualTo(Option.some("Hello STEVE"));
```

### Aplicación parcial

Con la aplicación parcial podemos crear una nueva función fijándole n parámetros a una ya existente, donde n siempre será inferior a la aridad de la función original y el retorno será una función de aridad original - parámetros fijados

```java
Function2<String, String, String> cheers = (s1, s2) -> String.format("%s %s", s1, s2);
Function1<String, String> sayHello = cheers.apply("Hello");
Function1<String, String> sayHola = cheers.apply("Hola");

assertThat(sayHola.apply("Juan")).isEqualTo("Hola Juan");
assertThat(sayHello.apply("John")).isEqualTo("Hello John");
```

Hemos definido una función *cheers* genérica que acepta dos parámetros de entrada, hemos derivado esta a otras dos nueva *sayHello* y *sayHola* aplicándola parcialmente, y ya tenemos dos más especifica para saludar y podríamos derivar en más casos si los necesitáramos.

### Currying

Currying es la técnica de descomponer una función de múltiples argumentos en una sucesión de funciones de un argumento.

```java
 Function3<Integer, Integer, Integer, Integer> sum = (a, b, c) -> a + b + c;

        Function1<Integer, Function1<Integer, Integer>> add2 = sum.curried().apply(2);

        Function1<Integer, Integer> add2And3 = add2.curried().apply(3);
        assertThat(add2And3.apply(4)).isEqualTo(9);
```

### Memoization

Una de las premisas de la programación funcional es tener funciones puras, sin side effects, esto básicamente es que a una función pasándole los mismo argumentos siempre ha de devolver el mismo resultado.

Por lo tanto, si siempre devolverá lo mismo, porque no cachearlo? pues esta es la misión de memoization, cachear las entradas y salidas de las funciones para sólo lanzarlas una vez.

```java
void memoization() {
        Function1<Integer, Integer> calculate = 
        	Function1.of(this::aVeryExpensiveMethod).memoized();

        long startTime = System.currentTimeMillis();
        calculate.apply(40);
        long endTime = System.currentTimeMillis();
        assertThat(endTime - startTime).isGreaterThanOrEqualTo(5000l);

        startTime = System.currentTimeMillis();
        calculate.apply(40);
        endTime = System.currentTimeMillis();
        assertThat(endTime - startTime).isLessThan(5000l);


        startTime = System.currentTimeMillis();
        calculate.apply(50);
        endTime = System.currentTimeMillis();
        assertThat(endTime - startTime).isGreaterThanOrEqualTo(5000l);

    }

    private Integer aVeryExpensiveMethod(Integer number) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return number * number;
    }
```

## Monads

### Try

El monad Try engloba una ejecución capturando una posible excepción, sus dos posibles valores de retorno son el caso de fallo por excepción o el valor resultado si ha ido bien.

Algunos métodos útiles del Try son:

.isSuccess() -> como el propio nombre indica, devuelve un boolean marcando si es un success.

.isFailure() -> devuelve un boolean marcando si es un failure.

get() -> obtiene el valor en el caso de que haya ido correctamente, si se hace un get y no se comprueba de si esto se hace sin comprobar si es success, soltará la excepción.

map() -> map sobre el valor en el caso de que haya ido bien, si es failure no se ejecutará.

getOrElse(T) -> el cual permite devolver un valor por defecto en el caso de error.

getOrElse(Supplier) -> el cual permite pasarle otra función en el caso de error.

recover( throwable -> {} ) -> Igual que el getOrElse pero en este caso tendremos la excepción que se ha lanzado para poder lograrla o poder devolver diferentes valores dependiendo del tipo de excepción.

```java
Function2<Integer, Integer, Integer> divide = (n1, n2) -> n1 / n2;

assertThat(Try.of(() -> divide.apply(10, 0)).isFailure()).isTrue();

assertThat(Try.of(() -> divide.apply(10, 5)).isSuccess()).isTrue();

assertThat(Try.of(() -> divide.apply(10, 5)).get()).isEqualTo(2);

assertThat(Try.of(() -> divide.apply(10, 0)).getOrElse(0)).isEqualTo(0);
```

### Lazy

Lazy es un monad sobre un Supplier al cual se le aplica memoization la primera vez que es evaluado.

```java
Lazy<List<User>> lazyOperation = Lazy.of(this::getAllActiveUsers);
assertThat(lazyOperation.isEvaluated()).isFalse();
assertThat(lazyOperation.get()).isNotEmpty();
assertThat(lazyOperation.isEvaluated()).isTrue();

```

### Either

Either representa un valor de dos tipos, Left y Right siendo por convención, poner el valor en el Right cuando es correcto y en el Left cuando no lo es.

Siempre el resultado será un left o un right, nunca podrá darse el caso de que sean las dos.



## Estructuras de datos

### Listas inmutables

Si uno de los principios de la programación funcional es la inmutabilidad, que pasa cuando definimos una lista y le añadimos ítems? pues que la estamos mutando.

Vavr proporciona una especialización de List a la cual una vez creada ya no puede ser modificada, cualquier operación de añadir, eliminar, reemplazar, nos dará una nueva instancia con los cambios aplicados



```java
import io.vavr.collection.List;
...

//Append
List<Integer> original = List.of(1,2,3);
List<Integer> newList = original.append(4);
assertThat(original.size()).isEqualTo(3);
assertThat(newList.size()).isEqualTo(4);

//Remove
List<Integer> original = List.of(1, 2, 3);
List<Integer> newList = original.remove(3);
assertThat(original.size()).isEqualTo(3);
assertThat(newList.size()).isEqualTo(2);

//Replace
List<Integer> original = List.of(1, 2, 4);
List<Integer> newList = original.replace(4,3);
assertThat(original).contains(1,2,4);
assertThat(newList).contains(1,2,3);
```

Ademas de la inmutabilidad también proporciona métodos directos para operar con la lista sin pasar por el stream, obtener el valor mínimo, máximo, medio.. para más información sobre lo que ofrece esta lista podéis consultarlo en su [javadoc](https://www.javadoc.io/doc/io.vavr/vavr/0.9.2)



Y estas son las principales características que nos ofrece Vavr, pero ofrece muchas más que nos ayudan a poder ser más funcionales en un lenguaje como java.

[Artículo publicado en Apiumhub](https://apiumhub.com/es/tech-blog-barcelona/java-con-vavr/)
