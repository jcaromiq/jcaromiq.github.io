---
title: "Funciones como ciudadanos de primera clase"
date: 2018-09-10
description: "Como componer funciones en diferente lenguajes de la jvm"
categories: ["General"]
tags: ["java", "Clojure","Kotlin", "functional programming"]
draft: true
image: "mathematics.jpg"

---

Una de las conceptos de la programación funcional es que las funciones son tratadas como "ciudadanos de primera clase", esto quiere decir que son tratadas como el resto de las variables, objectos. Se pueden asignar a una variable, pueden pasarse como argumento, incluso, una funcion puede devolver otra funcion.

en este articulo vamos a ver diferentes implementaciones en lenguajes de la jvm tan dispares como java, Kotlin o Clojure

<!--more-->

## Java

Hasta java 8 una funcion era de tercera clase, unicamente era posible definirla y llamarla.

En java 8 se introdujo algo del paradigma funcional al lenguaje, haciendo que las funciones pasarán a ser ciudadanos de primera introduciendo las [Interfaces Funcionales](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html) de las cuales las básicas son:

* Consumer -> Funcion que recibe un parametro y no devuelve nada
* Function -> Funcion que recibe un parámetro y devuelve un resultado
* Supplier -> Funcion que no recibe parametros y devuelve un resultado
* Predicado -> Funcion recibe un parametro y devuelve un booleano.

A partir de aquí, surgen diferentes combinaciones como los BiXXX que reciben dos parametros, o los XXToYY que transforman un tipo a otro o las especificaciones como IntConsumer/Supplier, LongConsumer/Supplier etc etc..

Siendo el limite 2 argumentos como entrada a no ser que se usen librerias como [vavr](https://apiumhub.com/tech-blog-barcelona/functional-java-with-vavr/) que permiten ampliar hasta 8

### Ejemplos

```java
// Asignar una funcion a una variable
Function<String, String> applyFormat = (it) ->
	it.trim().replaceAll("_", " ").toLowerCase();

String hello = applyFormat.apply("   Hello_World  ");

Assertions.assertThat(hello).isEqualTo("hello world");
```

El siguiente ejemplo es una posible implementacion del calculo de movimiento de la kata mars Rover, en la cual creariamos un map con la orientacion y la funcion a ejecutar en cada caso, ademas de que la funcion que se guarda esta parcialmente aplicada.

``````java
Map<Orientation, Function<Point, Point>> gps = new HashMap<>();
Function2<Integer, Point, Point> moveYBy = (i, p) -> new Point(p.x, p.y + i);
Function2<Integer, Point, Point> moveXBy = (i, p) -> new Point(p.x + i, p.y);

gps.put(Orientation.NORTH, moveYBy.apply(1));
gps.put(Orientation.EAST, moveXBy.apply(1));
gps.put(Orientation.SOUTH, moveYBy.apply(-1));
gps.put(Orientation.WEST, moveXBy.apply(-1));

....
    
Point point = gps.get(Orientation.EAST).apply(new Point(0, 0));

Assertions.assertThat(point).isEqualTo(new Point(1, 0));
``````

```java
//Pasar una funcion como argumento

```

