---
layout: single
title: "Spring WebFlux - Functional Endpoint With Coroutine Tutorial"
categories:
  - TUTORIALS
tags:
  - Spring WebFlux
  - kotlin
  - Functional EndPoint
  - Coroutine

toc: true
toc_sticky: true
excerpt: 이번 글에서는 이전에 작성한 Spring WebFlux - Spring Data R2DBC tutorial에 이어 Spring WebFlux를 이용한 API를 만들어보는 글을 작성해 보겠다. Spring WebFlux에서는 Web API를 위한 2가지 방식의 프로그래밍 모델을 제공해준다. 하나는 Annotation 기반 컨트롤러방식이고 다른 하나는 함수형 앤드포인트방식이다. 먼저 이번글에는 **함수형 앤드포인트**방식으로 API를 어떻게 구현하는지를 먼저 보여주고자 한다. 이후 글에서 **Annotation 기반 컨트롤러**를 이용한 API 구현 방법에 대해서도 다루어 보겠다.
---

## Introduction

이번 글에서는 이전에 작성한 [Spring WebFlux - Spring Data R2DBC tutorial](/tutorials/spring-data-r2dbc-tutorial/){: target="\_blank" }에 이어 [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }를 이용한 API를 만들어보는 글을 작성해 보겠다. Spring WebFlux에서는 Web API를 위한 2가지 방식의 프로그래밍 모델을 제공해준다. 하나는 [Annotation 기반 컨트롤러](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-controller){: target="\_blank" }방식이고 다른 하나는 [함수형 앤드포인트](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-fn){: target="\_blank" }방식이다.

먼저 이번글에는 **함수형 앤드포인트**방식으로 API를 어떻게 구현하는지를 먼저 보여주고자 한다. 이후 글에서 **Annotation 기반 컨트롤러**를 이용한 API 구현 방법에 대해서도 다루어 보겠다.

## Functional Endpoint

Spring WebFlux에서는 [Spring MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc){: target="\_blank" }로 개발을 해본 개발자라면 익숙한 **Annotation 기반 컨트롤러**방식 뿐만 아니라 **함수형 앤드포인트**방식의 라우팅을 제공해준다. (Spring MVC에서도 [함수형 앤드포인트](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#webmvc-fn){: target="\_blank" }방식 또한 제공해준다.)

`Controller`가 라우팅와 데이터 처리를 함께 하는 **Annotation 기반 컨트롤러** 방식과 달리 **함수형 앤드포인트**방식은 라우팅을 위한 `Router Function`과 데이터 처리를 위한 `Handler Function`으로 나뉘어져 있다.

## Kotlin Coroutine

Spring 5.2 버전부터 공식적으로 Spring WebFlux에서 Kotlin의 [Coroutine](https://kotlinlang.org/docs/reference/coroutines-overview.html){: target="\_blank" }을 이용한 `Router Function` 기능을 지원한다. 또한 `awaitBody`, `bodyToFlow`, `coRouter` 등 Coroutine을 위한 다양한 함수들을 지원하고 있어 편리하게 Coroutine을 사용할 수 있다.

## Getting Start

이제 **함수형 앤드포인트** 방식으로 간단한 Web API를 만들어 보자. 프로젝트 생성은 [이전 글의 Getting Start](http://localhost:4000/tutorials/spring-data-r2dbc-tutorial/#getting-start){: target="\_blank" }에서 생성한 프로젝트에서 아래의 의존성을 추가하여 진행하면 된다.

### Gradle

먼저 WebFlux를 사용하기 위한 의존성을 추가한다.
그다음 Coroutine 사용과 테스팅을 위한 의존성도 함께 추가해 준다.

```gradle
implementation("org.springframework.boot:spring-boot-starter-webflux")
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
```

<br />

### Handler Function

**Handler Function**을 추가하자. 여기서 **Handler Function**의 역할은 **Repository**를 주입 받아서 사용자의 요청 데이터를 다루는 것이다.

먼저 `PersonHandler` 클래스를 생성하자. `PersonHandler`클래스는 `PersonRepository`를 주입받는다. (원래라면 **Handler FUnction**이 **Repository**를 직접 의존하고 있는 구조는 좋지않은 설계이다. **Service** 역할을 하는 객체를 의존하는게 더 나은 설계라 할 수 있지만 튜토리얼에서는 해당 부분을 중요하게 다루지 않고 **Service** 객체를 생성할 필요성이 없다고 판단해서 **Repository**를 바로 참고하는 구조로 소개한다.) 추가로 `@Component` 어노테이션을 추가하여 빈으로 등록한다.

```kotlin
@Component
class PersonHandler(private val repository: PersonRepository)
```

<br />

저장 요청을 다루는 함수를 추가한다. 사용자의 저장요청을 **Repository**에 전달하고 HTTP 응답을 반환한다. 여기에서 우리는 Coroutine을 이용할 것이기 때문에 [suspend function](https://kotlinlang.org/docs/reference/coroutines/composing-suspending-functions.html)을 사용해야 한다는 점을 유의해야 한다.

```kotlin
suspend fun create(request: ServerRequest) =
    repository
        .save(request.awaitBody<Person>())
        .awaitSingle()
        .let {
            created(URI("/users"))
                .buildAndAwait()
        }
```

<br />

모든 데이터 목록 조회 요청을 다루는 함수를 추가한다. **Handler Function**은 사용자의 조회 요청을 받으면 **Repository**에서 데이터를 조회하고 조회한 결과와 함께 HTTP 응답을 반환한다. 저장 함수와 마찬가지로 **suspend function**을 사용한다.

```kotlin
suspend fun findAll(request: ServerRequest) =
    ok()
        .contentType(MediaType.APPLICATION_JSON)
        .bodyAndAwait(repository.findAll().asFlow())
```

<br />

단일 데이터 조회 요청을 다루는 함수를 추가한다. URI의 경로에서 ID값을 가져와서 **Repository**에서 데이터를 조회하고 조회한 결과와 함께 HTTP 응답을 반환한다. 만약 ID가 존재하지 않는다면 Bad Request 응답을 반환한다. 저장 함수와 마찬가지로 **suspend function**을 사용한다.

```kotlin
suspend fun findById(request: ServerRequest) =
    repository
        .findById(request.pathVariable("id").toLong())
        .awaitFirstOrNull()
        ?.let {
            ok()
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValueAndAwait(it)
        }
        ?: badRequest().buildAndAwait()
```

<br />

수정 요청을 다루는 함수를 추가한다. URI의 경로에서 ID값을 가져와서 **Repository**에서 데이터를 조회하고 데이터가 존재한다면 해당 데이터를 수정 하는 함수를 호출한다. 만약 ID가 존재하지 않는다면 Bad Request 응답을 반환한다. 저장 함수와 마찬가지로 **suspend function**을 사용한다.

```kotlin
suspend fun update(request: ServerRequest): ServerResponse {
    val id = request.pathVariable("id").toLong()
    val isExist = repository.existsById(id).awaitSingle()

    return if (isExist) {
        val person = request.awaitBody<Person>().copy(id = id)
        repository.save(person).awaitSingle()

        ok().buildAndAwait()
    } else {
        badRequest().buildAndAwait()
    }
}
```

<br />

삭제 요청을 다루는 함수를 추가한다. URI의 경로에서 ID값을 가져와서 **Repository**에서 데이터를 조회하고 데이터가 존재한다면 해당 데이터를 삭제하는 함수를 호출한다. 만약 ID가 존재하지 않는다면 Bad Request 응답을 반환한다. 저장 함수와 마찬가지로 **suspend function**을 사용한다.

```kotlin
suspend fun delete(request: ServerRequest): ServerResponse {
    val id = request.pathVariable("id").toLong()
    val isExist = repository.existsById(id).awaitSingle()

    return if (isExist) {
        repository.deleteById(id).awaitFirstOrNull()
        ok().buildAndAwait()
    } else {
        badRequest().buildAndAwait()
    }
}
```

<br />

### Router Function

**Router Function**을 추가한다. **Router Function**을 사용하기 위해서는 `@Configuration` 어노테이션을 사용해야 한다.

```kotlin
@Configuration
class Routes
```

<br />

Router를 빈으로 등록한다. Coroutine을 사용하기 위해서 [coRouter](https://docs.spring.io/spring/docs/current/kdoc-api/spring-framework/org.springframework.web.reactive.function.server/co-router.html){: target="\_blank" }를 사용한다.
각 요청 경로에 맞는 handler를 매핑한다.

```kotlin
@Bean
fun personRoutes(handler: PersonHandler) = coRouter {
    "/persons".nest {
        POST("", handler::create)
        GET("", handler::findAll)
        GET("/{id}", handler::findById)
        PUT("/{id}", handler::update)
        DELETE("/{id}", handler::delete)
    }
}
```

<br />

### Testing

먼저 테스트 클래스를 생성하자. Spring Boot 2.2 버전에서는 테스트용 도구로 [junit5](https://junit.org/junit5/docs/current/user-guide/){: target="\_blank" }를 사용하고 있다.
테스트 클래스임을 설정하기 위해 `@ExtendWith(SpringExtension::class)` 주석을 추가한다. 다음으로 [`WebTestClient`](https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#webtestclient){: target="\_blank" }를 이용한 테스트를 위해 `@WebFluxTest(Routes::class, PersonHandler::class)` 주석을 추가한다. 매개변수로 테스트할 `Router Function`과 `Handler Function`을 추가해주어야 한다.

```kotlin
@ExtendWith(SpringExtension::class)
@WebFluxTest(Routes::class, PersonHandler::class)
internal class PersonRouteTest
```

<br/>

테스트를 담당한 테스트용 클라이언트 도구인 `WebTestClient` 객체를 추가하고 `PersonRepository`는 이번 테스트 대상에서 벗어나니 Mock을 이용하였다.

```kotlin
@Autowired
private lateinit var client: WebTestClient

@MockBean
private lateinit var personRepository: PersonRepository
```

<br/>

테스트 전에 테스트 코드를 줄여줄 공통 함수를 하나 추가한다. 해당 함수는 데이터베이스에 데이터를 저장하기 위한 랜덤 데이터를 생성해 준다.

```
private fun generateRandomPerson(id: Long? = null) = Person(
    id = id,
    firstname = UUID.randomUUID().toString(),
    lastname = UUID.randomUUID().toString()
)
```

<br />

목록 조회 API에 대한 테스트를 추가한다. `PersonRepository`에서 DB에서 조회한 값을 가상으로 제공 해준다면 목록을 올바르게 반환하는지 테스트한다.

```kotlin
@Test
fun `test get all persons`() {
    Mockito.`when`(personRepository.findAll())
        .thenReturn(
            listOf(
                generateRandomPerson(),
                generateRandomPerson(),
                generateRandomPerson()
            ).toFlux()
        )

    client.get()
        .uri("/persons")
        .exchange()
        .expectStatus().isOk
        .expectHeader().contentType(MediaType.APPLICATION_JSON)
        .expectBody()
        .jsonPath("$").isArray
        .jsonPath("$.length()").value(Matchers.`is`(3))
}
```

<br />

단일 항목 조회 API에 대한 테스트를 추가한다. `PersonRepository`에서 ID를 제공하여 DB에서 조회한 값을 가상으로 제공해 준다면 데이터를 올바르게 반환하는지 테스트한다.

```kotlin
@Test
fun `test get one person`() {
    val id = Random.nextLong()
    val person = generateRandomPerson(id)
    Mockito.`when`(personRepository.findById(id))
        .thenReturn(person.toMono())

    client.get()
        .uri("/persons/$id")
        .exchange()
        .expectStatus().isOk
        .expectHeader().contentType(MediaType.APPLICATION_JSON)
        .expectBody<Person>().isEqualTo(person)
}
```

<br />

단일 항목에 대한 API 호출 시 조회를 할 수 없는 경우에 대한 테스트를 추가한다. `PersonRepository`에서 ID를 제공받아 DB에서 데이터를 조회하였지만 존재하지 않는 ID의 경우 API에서 정해진 규칙에 맞게(여기에서는 400 Bad Request 응답을 반환하도록 하였다.) 응답하는지 테스트한다.

```kotlin
@Test
fun `test get one person if it is not exists`() {
    val id = Random.nextLong()
    Mockito.`when`(personRepository.findById(id))
        .thenReturn(Mono.empty())

    client.get()
        .uri("/persons/$id")
        .exchange()
        .expectStatus().isBadRequest
}
```

<br/>

데이터를 저장하는 API 테스트를 추가한다. `PersonRepository`에서 가상으로 데이터를 저장하는 함수를 호출하도록 한 후 저장을 위한 API 호출 후 정상적으로 저장을 위한 함수가 호출되는 지를 테스트 한다.

```kotlin
@Test
fun `test post person`() {
    val person = generateRandomPerson()

    Mockito.`when`(personRepository.save(person))
        .thenReturn(person.toMono())

    client.post()
        .uri("/persons")
        .body(BodyInserters.fromValue(person))
        .exchange()
        .expectStatus().isCreated

    Mockito.verify(personRepository).save(person)
}
```

<br />

데이터를 수정하는 API 테스트를 추가한다. `PersonRepository`에서 주어진 ID에 대한 데이터를 가상으로 조회하고 저장하도록 한 후 수정을 위한 API 호출 시 정상적으로 데이터를 수정하기 위한 함수가 호출되는 지를 테스트 한다.

```kotlin
@Test
fun `test put person`() {
    val id = Random.nextLong()
    val person = generateRandomPerson(id)

    Mockito.`when`(personRepository.existsById(id))
        .thenReturn(true.toMono())
    Mockito.`when`(personRepository.save(person))
        .thenReturn(person.toMono())

    client.put()
        .uri("/persons/$id")
        .bodyValue(person)
        .exchange()
        .expectStatus().isOk

    Mockito.verify(personRepository).save(person)
}
```

<br />

데이터 수정 API 호출 시 존재하지 않는 데이터의 수정요청에 대한 테스트를 추가한다. `PersonRepository`에서 주어진 ID에 대한 데이가 없는 경우를 가상으로 제공하고 API가 정해진 규칙에 맞게(여기에서는 400 Bad Request 응답을 반환하도록 하였다.) 응답하는지 테스트한다.

```kotlin
@Test
fun `test put person if it is not exist`() {
    val id = Random.nextLong()
    val person = generateRandomPerson(id)

    Mockito.`when`(personRepository.existsById(id))
        .thenReturn(false.toMono())

    client.put()
        .uri("/persons/$id")
        .bodyValue(person)
        .exchange()
        .expectStatus().isBadRequest

    Mockito.verify(personRepository, never()).save(person)
}
```

<br />

데이터를 삭제하는 API 테스트를 추가한다. `PersonRepository`에서 주어진 ID에 대한 데이터가 존재하면 해당 데이터를 삭제하도록 하는 가상의 함수를 생성한 후 삭제 API를 호출하면 실제로 삭제 함수가 실행되는지 테스트한다.

```kotlin
@Test
fun `test delete person`() {
    val id = Random.nextLong()

    Mockito.`when`(personRepository.existsById(id))
        .thenReturn(true.toMono())
    Mockito.`when`(personRepository.deleteById(id))
        .thenReturn(Mono.empty())

    client.delete()
        .uri("/persons/$id")
        .exchange()
        .expectStatus().isOk

    Mockito.verify(personRepository).deleteById(id)
}
```

<br />

데이터 삭제 API 호출 시 존재하지 않는 데이터의 삭제요청에 대한 테스트를 추가한다. `PersonRepository`에서 주어진 ID에 대한 데이가 없는 경우를 가상으로 제공하고 API가 정해진 규칙에 맞게(여기에서는 400 Bad Request 응답을 반환하도록 하였다.)

```kotlin
@Test
fun `test delete person if it is not exist`() {
    val id = Random.nextLong()
    Mockito.`when`(personRepository.existsById(id))
        .thenReturn(false.toMono())

    client.delete()
        .uri("/persons/$id")
        .exchange()
        .expectStatus().isBadRequest

    Mockito.verify(personRepository, never()).deleteById(id)
}
```

## GIT

해당 튜토리얼에 대한 전체 코드는 Github을 참고하기 바란다. ([https://github.com/veluxer62/toy/tree/wbflux-functional-endpoint-with-coroutine-and-r2dbc/spring-data-r2bc-example](https://github.com/veluxer62/toy/tree/wbflux-functional-endpoint-with-coroutine-and-r2dbc/spring-data-r2bc-example){: target="\_blank" })

## Wrap up

지금까지 Kotlin의 Coroutine을 이용하여 Spring WebFlux의 Functnional Endpoint 방식의 API를 구현해 보았다. Coroutine을 사용하면 [Reactor](https://projectreactor.io/){: target="\_blank" }의 `Flux`나 `Mono`를 사용하여 객체를 감싸지 않고도 suspend fun 키워드를 사용하여 비동기적인 작업을 수행할 수 있다. 마치 javascript의 [async function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function)과 유사하다.

이 후 글에서는 **Annotation 기반 컨트롤러**와 **Reactor**를 이용한 API 구현방법에 대해서도 이야기 해보고 **함수형 앤드포인트** API와 어떤점이 다른지 알아보겠다.

> 이전 글 보기 : [Spring WebFlux - Spring Data R2DBC tutorial](/tutorials/spring-data-r2dbc-tutorial/){: target="\_blank" }

<br/>

> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-controller](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-controller){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-fn](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-fn){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#webmvc-fn](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#webmvc-fn){: target="\_blank" }
>
> [https://kotlinlang.org/docs/reference/coroutines-overview.html](https://kotlinlang.org/docs/reference/coroutines-overview.html){: target="\_blank" }
>
> [https://kotlinlang.org/docs/reference/coroutines/composing-suspending-functions.html](https://kotlinlang.org/docs/reference/coroutines/composing-suspending-functions.html){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/kdoc-api/spring-framework/org.springframework.web.reactive.function.server/co-router.html](https://docs.spring.io/spring/docs/current/kdoc-api/spring-framework/org.springframework.web.reactive.function.server/co-router.html){: target="\_blank" }
>
> [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function){: target="\_blank" }
