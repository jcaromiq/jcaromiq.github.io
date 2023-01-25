---
title: "Rust"
date: 2020-09-15T22:05:31+02:00
description: "Breve introducción al lenguaje de programación Rust"
categories: ["Rust"]
tags: ["Rust"]
toc: false

canonicalURL: "https://apiumhub.com/es/tech-blog-barcelona/descubriendo-rust/"
image: "rust.jpg"

---
Seguramente has escuchado hablar de [Rust](https://www.rust-lang.org/), de todas sus bondades y su quizás compleja curva de aprendizaje.
En este articulo no voy a profundizar en que es Rust, si no que voy a detallar las características que en el poco tiempo que llevo usándolo he visto que hacen de Rust un lenguaje muy atractivo e interesante de usar.

<!--more-->

## Por que Rust?

Como developers es fácil subirse rápido al tren del hype e intentar aprender o incluso utilizar aunque sea en pet projects las últimas librerías, frameworks y porque no lenguajes de programación.
En un sector donde todo evoluciona a una velocidad muy alta, a veces cuesta mantener el foco y no volverse loco con los constantes cambios.
En mi caso, me gusta leer y practicar diferentes lenguajes de programación y contra más distintos sean entre ellos mejor, ya que aportan diferentes visiones de como resolver problemas e incluso te dan más herramientas para tu día a día.


En uno de los lunch & learn que hacemos en Apiumhub, con un compañero salió la idea migrar un cli interno que lo tenemos en puro bash a algún otro lenguaje y como candidatos estaban Go y Rust, como entre nosotros no nos pusimos de acuerdo, decidimos que cada uno elegiría un lenguaje y haría su port.
El resultado fue que ese cli no se migró nunca, pero por el camino, ambos pudimos descubrir las ventajas e inconvenientes de lenguajes como GO y Rust.
Por mi parte no se quedo en una simple prueba de concepto, desde aquel momento, Rust me engancho a la vez que desespero y desde entonces, cualquier proyecto personal o prueba de concepto o idea que se me ocurra, la intento picar primero en Rust.



## Que es Rust

Rust se basa en tres pilares:

* Performance, rápido y eficiente con la memoria, sin runtime ni Garbaje collector

* Fiable, el modelo de ownership garantiza memory-safety and thread-safety 

* Productivo, gran documentación, un compilador que en los errores la gran mayoría de veces te dice como has de hacerlo y te da referencias a la documentación para que entiendas el problema y una herramienta que sirve de package manager, formateador de código, mejora y sugerencias.

Rust empezó definiéndose como un lenguaje de sistemas, pero con el tiempo esa definición ha ido desapareciendo, cada vez hay más librerías para [web](https://www.arewewebyet.org/), para [GUI](https://areweguiyet.com/), para [juegos](https://arewegameyet.rs/) y por supuesto para sistemas!

Lo primero que piensas cuando lees "lenguaje de sistemas" es.. voy a tener que tratar con punteros, voy a tener que trabajar liberando y asignando memoria, voy a tener que trabajar a muy bajo nivel sin abstracciones y claro.. si no es tu día a día pues da miedo volver a los orígenes de C C++, pero la gran mayoría de estos miedos están muy bien resueltos y por el compilador! con lo que nos facilita mucho el camino.

En este articulo veremos algunos conceptos básicos de rust y algunas de sus features que bajo mi punto de vista hacen que sea un lenguaje muy atractivo.



## Conceptos 

* Rust tiene un amplio catálogo de [tipos primitivos](https://doc.rust-lang.org/std/index.html#primitives)

* Rust apuesta por la inmutabilidad por lo tanto, todas las variables declaradas son inmutables a no ser que a la hora de declararlas se especifique con el keyword mut.

* Lo mismo pasa con la visibilidad, todo es privado y en el caso que se quiera hacer publico se hará con el keyword pub.
* En el caso de las funciones podemos especificar el keyword return para indicar que es el valor a devolver de la función o si la ultima sentencia no incluye el ; se convertirá en el return de la función.

* Rust tiene una inferencia de tipos muy completa y pocas veces nos hará falta especificar el tipo de variable que estamos creando.
* Otras funcionalidades que le dan a Rust un plus es el que tiene tipos Genéricos, Pattern matching, sistema de Macros y muchos conceptos de la programación funcional como first-class functions,  closures, Iterators

```rust
let name = "Jon".to_string(); // String type
let year = 2020; // i32 type
let age: u8 = 35;

let mut index = 0; //mutable variable
index = index +=1; 

let add_one = |x: i32| x + 1; //closure
let two = add_one(1);

let numbers = vec![1, 2, 3, 4, 5, 6];
let pairs: Vec<i32> = numbers.into_iter()
		.filter(|n| n % 2 == 0)
		.collect(); // [2, 4, 6]

fn greet(name: String) {
    println!("hello {}", name);
}
pub fn plus_one(x: i32) -> i32 {
    return x + 1; 
}
pub fn plus_two(x: i32) -> i32 {
    x + 2 
}
```



### Ownership, borrowing y lifetimes

Estos tres conceptos son la mayor complejidad que nos encontraremos en Rust, ya que son conceptos que en lenguajes con GC es la propia GC la que los trata haciéndolo transparente para el desarrollador.

Al no tener un runtime asociado ni un Garbage Collector que vaya liberando de la memoria los objetos que no usa, todo eso lo maneja el compilador con la ayuda del ownership,
Aunque el concepto da para más de un articulo, vamos a ver lo básico del concepto con algún ejemplo.

Cada valor tiene una variable asignada (owner) y sólo puede haber un owner al mismo tiempo, cuando ese owner esta fuera del scope, el valor será liberado.

Con lo cual tenemos el siguiente ejemplo:

```rust
fn main() {
    let name = "Jon".to_string();
    greet(name);
    println!("goodbye {}", name); //^^^^ value borrowed here after move
}

fn greet(name:String) {
    println!("hello {}", name);
}
```

El compilador nos esta avisando que el owner de la variable name ha sido pasado a la función greet, con lo cual después de ejecutarse greet ya no esta en ese scope.

Para solucionarlo es tan sencillo como indicarle que lo que queremos es prestarle el owner, para que cuando la función termite, vuelva a obtener el owner, y eso se indica con el &

```rust
fn main() {
    let name = "Jon".to_string();
    greet(&name);
    println!("goodbye {}", name);
}

fn greet(name:&String) {
    println!("hello {}", name);
}
```

El lifetime es un check de la gestión del ownership y borrow de los valores, la mayoría de las veces la sabe interpretar el propio compilador, pero a veces hay que detallarlo. Sin entrar mucho en detalle ya que lo mismo que el ownership y borrowing, da para una serie de artículos.



### Structs

Las structs existen para poder definir nuestros propios tipos, se crean con el keyword struct y no tienen comportamiento asociado.

Para darle comportamiento a un struct se hará mediante el keyword impl.

```rust
struct Point {
    x: f32,
    y: f32,
}
impl Point {
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}
let point1 = Point {
  x: 10.0,
  y: 20.0,
};

let point2 = Point {
  x: 5.0,
  y: 1.0,
};

let point3 = point1.add(point2); //Point { x: 15.0, y: 21.0 }
```

En la implementación de Point el método add recibe como parámetro self, esto es porque ese método es de instancia, si no quisiéramos hacerlo de instancia es tan sencillo como quitar el self

### Traits

Los Traits en Rust son una colección de métodos definidos para ser implementados por Structs, son similares a las interfaces de otros lenguajes.

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```



### Enums 

Podemos crear enums, sin valor, con valor o incluso que cada enum tenga valores de tipos diferentes

```rust
enum IpAddrKind {
    V4,
    V6,
}

enum IpAddr {
    V4(String),
    V6(String),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

Aquí es donde el pattern matching y destructuring juegan un valor muy importante para poder manejar los enums



### Alias

En rust podemos crear nuestros alias para los tipos existentes

```rust
type Second = u64;
let seconds: Second = 10;
```



### Añadir funcionalidad a clases una vez definidas

En algunos lenguajes de programación es posible añadir métodos (extension methods) a clases una vez definidas, en Rust no iba a ser menos!

Primero nos crearemos el trait con el método que queremos utilizar.

Seguidamente implementaremos el Trait antes creado al tipo que queramos extender.

```rust
pub trait Reversible {
    fn reverse(&self) -> Self;
}

impl Reversible for String {
    fn reverse(&self) -> Self {
        self.chars()
            .rev()
            .collect::<String>()
    }
}

let hello = String::from("hello");
println!("{}",hello.reverse()); // olleh
```



### Sobrecarga de operadores

Imaginar poder aplicar operaciones aritméticas a vuestros propios tipos, siguiendo el ejemplo antes del struct Point, poder sumar Points con el simple operador +.

Pues en Rust esto es posible sobrecargando los operadores, que vuelve a ser lo mismo, implementar traits para los tipos.

[Aquí están todos los posibles operadores a sobrecargar](https://doc.rust-lang.org/std/ops/index.html#traits)

```rust
struct Point {
    x: f32,
    y: f32,
}

impl std::ops::Add for Point { // implement Add trait for Point
    type Output = Self;

    fn add(self, other: Self) -> Self { //implement add function
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

let point1 = Point { x: 10.0, y: 20.0};

let point2 = Point {x: 5.0, y: 1.0};

let p = point1 + point2; // apply add trait
```



### Question mark operator

Es muy comun ver el manejo de errores en Rust mediante el tipo Result<T,E> siendo T el valor del OK y E el del error, y el manejo de este tipo se realiza mediante pattern matching.

Imaginemos que queremos leer el contenido de un fichero, para ello utilizaremos la función read_to_string del modulo fs, el resultado de esa función es un Result<String, std:io::error::Error>.

```rust
let content = fs::read_to_string("filename");
match content {
	Ok(file_content) => { println!("{}", file_content) }
	Err(e) => { println!("Error reading file: {}", e) }
}
```

Mediante pattern matching hemos tratado ambos posibles casos del result. Para este ejemplo solo hemos querido printarlo por consola, pero imaginar que queréis tratar ese contenido, el código se vuelve algo más complejo, para estos casos en Rust existe el operador ? con el cual directamente obtenemos el valor del Result si ha ido bien y si no directamente se devolverá el error como return de la función.

Pero para poder usar el operador ? la firma del método ha de devolver un tipo Result<T,E>

```rust
fn get_content() ->  Result<String, Error>{
    let content = fs::read_to_string("filename")?;
    println!("{}", content);
    Ok(content)
}
```

Si la lectura del fichero falla, automáticamente devolverá el trait Error, si no el código seguirá ejecutándose hasta devolver el Result ok del content.



### Conversion de tipos

Otra feature muy interesante es la conversion de tipos que se puede aplicar en Rust únicamente implementando el Trait std::convert::From

El caso de uso más claro es una función donde manejas diferentes tipos de errores pero el retorno de tu función quieres que sea en el caso de error un error tuyo, de tu dominio, mediante pattern matching podríamos ir cazando todos los results y creando los Results de nuestro tipo, pero haría nuestro código difícil de mantener y poco legible.

Mediante el operador ? y la conversion de tipos quedaría de la siguiente manera

```rust
fn get_and_save() -> Result<String, DomainError> {
    let content:  Result<String,HttpError> = get_from_http()?; 
    let result: Result<String,DbError> = save_to_database(content)?; 
  	// with ? operator in case of error, the return of the function will be Result of Error
    Ok(result)
}

pub struct DomainError {
    pub error: String,
}

impl std::convert::From<DbError> for DomainError {
    fn from(_: DbError) -> Self {
        DomainError {
            error: (format!("Error connecting with database")),
        }
    }
}

impl std::convert::From<HttpError> for DomainError {
    fn from(_: HttpError) -> Self {
        DomainError {
            error: (format!("Error connecting with http service")),
        }
    }
}
```

### Cargo y utilidades

Y si hablamos de rust, no podemos dejar de lado su package manager cargo, el cual viene con la propia instalación de Rust.

Con cargo podemos crear un proyecto desde cero, gestionar las dependencias, generar la release, lanzar los tests, generar la documentación, publicar el package al registry...

Además hay una gran lista de comandos de terceros [disponibles](https://github.com/rust-lang/cargo/wiki/Third-party-cargo-subcommands)



## Recursos

Aunque Rust fue creado por Mozilla, es mantinido por la comunidad, es la propia comunidad la que va proponiendo los cambios y adaptando el lenguaje a las necesidades.

Algunos de los enlaces más interesantes a seguir para estar al tanto de las novedades:

[rust-lang.slack](rust-lang.slack.com),  Slack en el cual se tratan todos los temas referentes al lenguaje con mucha ayuda a los que se inician en el lenguaje.

[Weekly](https://this-week-in-rust.org/) Newsletter semanal con las novedades tanto a nivel de cambios del lenguaje, como crates interesantes, artículos y conferencias / charlas

[youtube](https://www.youtube.com/channel/UCaYhcUwRBNscFNUKTjgPFiA) canal oficial de rust donde se cuelgan conferencias, los meetings de los diferentes grupos de trabajo formados para el desarrollo del lenguaje.

[Discord](https://discord.gg/rust-lang) Servidor de Discord donde se coordinan la mayoría de los grupos de trabajo para mantener el lenguaje

[El libro de rust](https://doc.rust-lang.org/book/) Documentación oficial sobre Rust, todo lo que debes de saber sobre el lenguaje está en el libro.

Y un par de proyectos personales con los que voy poniendo en práctica todo lo que voy leyendo sobre Rust

[Adh](https://github.com/jcaromiq/adh-rust) es un cli para docker, portado del original de [ApiumHub](https://github.com/ApiumhubOpenSource/adh)  con los comandos que más utilizo en el día a día, aún faltan muchas funcionalidades por añadir.

[Covid-bot](https://gitlab.com/jcaromiq/corona-bot) es un bot para telegram desarrollado en rust para estar informado de los casos de covid-19


[Artículo publicado en Apiumhub](https://apiumhub.com/es/tech-blog-barcelona/descubriendo-rust/)
