---
title: "Feature Flag"
date: 2018-12-26T01:05:31+02:00
description: "Patron Feature Flag"
categories: ["Java"]
tags: ["java", "kotlin"]
image: "apple.jpg"
---

## Introducción
Es muy común ver proyectos que utilizan el mecanismo de Feature Branch para poder tener todo el desarrollo de una feature aislada y una vez finalizada poder integrarla en el flujo de release y poder generar version.

Sin entrar en debate sobre pros y contras entre Feature Branch y Master Only, la estrategia de Feature Branch tiene un handicap muy caro, el feedback, si la feature tarda por ejemplo un mes en estar finalizada, mínimo estará ese tiempo sin poder ser mergeada con el resto de código, sin poder ser probada con el resto de funcionalidades etc etc.

Si, está claro, con Feature Branch también podemos tener feedback "rápido", pero para ello necesitaríamos ajustar nuestra pipeline para que estuviera al tanto de cualquier rama que se creara y poder levantar una entorno completamente nuevo por rama, con el coste que esto supone, no sólo a nivel de dinero, si no de preparar los entornos, datos etc etc y aun así, tampoco estaríamos probando la integración de esa feature con el resto.


Imaginemos que podemos integrar nuestro código y poder meterlo en el ciclo de release de nuestra aplicación incluso sin tener la feature acabada, una de las ventajas más obvias es el feedback rápido tanto en la integración de nuestro código con el resto de la aplicación, como en minimizar los temidos conflictos futuros de merge.

## Feature flag al rescate!
Feature Flag, también llamado Feature Toogle, es el patrón perfecto para poder aplicar integración continua a nuestras features de larga duración.

Mediante una condición en nuestro código y un fichero de configuración podríamos activar o no funcionalidad de nuestra aplicación en runtime y una vez finalizada la feature podríamos activarla.

La implementación más sencilla sería una tabla en nuestra base de datos con el nombre de feature y el estado(Activo o no) y en nuestro código tener un if(activo) muestro else noMuestro.

Pero este patrón va mucho más allá, permitiendo poder activar una feature para un grupo de usuarios, país, navegador del usuario, comportamiento o aleatoriamente.

Cada una de estas implementaciones tienen nombres diferentes, AbTesting, Canary Release.. pero todo trata sobre lo mismo, exponer o no funcionalidad dependiendo de condiciones.

## Feature Flag For Java

Si nuestro stack tecnológico esta formado por java o cualquier lenguaje de la jvm estamos de suerte.

[FF4J](https://ff4j.github.io/) es la implementación del patron Feature Toggle en todo su esplendor. Sin apenas esfuerzo tenemos integración con diferentes tipos de orígenes de datos para poder guardar nuestras features con su estado, también nos da auditoria de cuantas veces se han aplicado las features, tanto por número total, por usuario, por origen..

También tiene diferentes estrategias de decisión, WhiteList, BlackList, ClientFilter, ServerFilter, Ponderación, etc... además nos permiten también crearnos nuestra propia estrategia.

Y todo esto configurable mediante una consola web o mediante API, con lo que facilita mucho la creación y mantenimiento de las Features.

## Show me the code!
Aunque en el repositorio del proyecto hay muchos ejemplos conectando con diferentes orígenes para las features (jdbc, cassandra, mongo etc etc) me ha parecido interesante crear un ejemplo que no existe.

Supongamos que tenemos un servicio para la configuración de todas las features y demás servicios se conectan a este por http para consultar las features activas.
En esté ejemplo, he creado un multimodule con dos servicios:

### ffconsole
Este servicio únicamente tiene la consola de administración para crear las features y un plugin para exponerlo mediante rest.
Para simplificar el ejemplo, está configurado para que el storage sea in memory y sin nada de seguridad (no aplicar en entornos productivos 😜).

La única configuración que hay es en la clase FF4jServletConfiguration.kt en la cual registramos el bean de FF4J, el servlet y la consola web y anotamos la clase para que importe los beans necesarios para exponer la api.
También para simplificar el ejemplo, hago que la feature ya se de de alta, esto debería de hacerse mediante el panel de administración.

```Kotlin
@Bean
fun getFF4j(): FF4j = FF4j().apply {
		createFeature("AwesomeFeature", true)
	}

@Bean
fun getFF4JServlet(): FF4jDispatcherServlet =
	FF4jDispatcherServlet().apply { ff4j = getFF4j() }

@Bean
fun ff4jDispatcherServletRegistrationBean(): ServletRegistrationBean =
	ServletRegistrationBean(getFF4JServlet(), "/ff4j-web-console/*")

```


### my-awesome-web
Una aplicación spring boot con un controlador que aplica una lógica u otra dependiendo si la feature esta o no activa.
Esta aplicación se conectará a ffconsole mediante http.
La configuración es tan sencilla como decirle cual es el storage:

```Kotlin
@Bean
fun ff4j(): FF4j =
	FF4j().apply {
		featureStore = FeatureStoreHttp("http://ff4j-console:8080/api/ff4j")
	}
```
y en el controlador tenemos un endpoint el cual devolverá un HTTP_200 en caso de que la feature esté activa y un HTTP_404 en caso de que no lo este.

```Kotlin
@GetMapping("/")
fun home(): ResponseEntity<String> =
	ff.operateWith("AwesomeFeature",
		ResponseEntity("Not available", HttpStatus.NOT_FOUND),
		ResponseEntity("This is the awesome Feature", HttpStatus.OK))
```
Este es el único código que una vez la feature ya esté implementada y probada lo suficiente en el entorno productivo, se debería de limpiar y aplicarlo siempre sin depender de si el flag esta activo o no.


Aquí es mi única pega de esta librería, la api para trabajar con las features peca un poco de funcionalidad, en el caso de que no exista lanza una runtimeException por ejemplo, o tratar con las features es algo imperativo, por lo que me he tomado la libertad para este ejemplo de hacer un pequeño wrapper y hacerlo un poco más "funcional", con lo que expongo tres métodos para trabajar con las features:

```Kotlin
fun enabled(featureID: String): Boolean =
	getFeature(featureID)
		.map { it.isEnable }
		.orElse(false)

fun getFeature(featureID: String): Optional<Feature> =
	try {
		Optional.of(ff4j.getFeature(featureID))
	} catch (nf: FeatureNotFoundException) {
		Optional.empty()
	}

fun <T> operateWith(featureId: String, nonExistsValue: T, existsValue: T): T =
	getFeature(featureId)
		.filter { it.isEnable }
		.map { existsValue }
		.orElse(nonExistsValue)

fun <T> operateWith(featureId: String, nonExistsMapper: () -> T, existsMapper: () -> T): T =
	getFeature(featureId)
		.filter { it.isEnable }
		.map { existsMapper() }
		.orElse(nonExistsMapper())
```
El proyecto está montado con un docker compose el cual con un ```make all``` genera los artefactos y levanta dos dockers para poder hacer pruebas.
la url de la parte de adminstracion es http://localhost:9090/ff4j-web-console/
y la url para la feature http://localhost:8080/

El proyecto al completo está en [github](https://github.com/jcaromiq/ff4j-sample).



## Conclusión
Este patrón no ha de ser utilizado como un martillo para todos los casos.

La mejor estrategia siempre es poder partir las funcionalidades / historias de usuario en más pequeñas para poder integrarlas lo antes posible, pero siempre hay casos que la feature no puede ser entregada hasta que no esté finalizada al 100% y es allí donde aplica.

Importante la fase de limpiar el código una vez la feature esta desplegada y probada durante el tiempo acordado, para la correcta evolución y mantenimiento de la aplicación.

## Bibliografia

* [FeatureToogle](https://martinfowler.com/bliki/FeatureToggle.html)
* [FF4J wiki](https://github.com/ff4j/ff4j/wiki)
