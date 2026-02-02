---
layout: single
title: "Spring WebFlux - Annotated Controller with reactor Tutorial"
categories:
  - TUTORIALS
tags:
  - Spring WebFlux
  - kotlin
  - Annotated Controller
  - Reactor

toc: true
toc_sticky: true
excerpt: 이번 글에서는 이전에 작성한 Spring WebFlux - Functional Endpoint With Coroutine Tutorial에 이어 Spring WebFlux를 이용한 또다른 방식의 API를 만들어보는 글을 작성해 보겠다. 이전 글에서는 함수형 앤드포인트방식을 이용하여 API를 만들었는데 이 글에서는 Annotation 기반 컨트롤러방식으로 API를 구현하는 방법에 대해서 다루어 보겠다.
---

## Introduction

이번 글에서는 이전에 작성한 [Spring WebFlux - Functional Endpoint With Coroutine Tutorial](/tutorials/spring-web-flux-functional-endpoint-api/){: target="\_blank" }에 이어 [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }를 이용한 또다른 방식의 API를 만들어보는 글을 작성해 보겠다.

이전 글에서는 [함수형 앤드포인트](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-fn){: target="\_blank" }방식을 이용하여 API를 만들었는데 이 글에서는 [Annotation 기반 컨트롤러](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-controller){: target="\_blank" }방식으로 API를 구현하는 방법에 대해서 다루어 보겠다.

## Annotated Controller

**Annotated Controller**방식은 [Spring MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc){: target="\_blank" }로 개발을 해본 개발자라면 아주 익숙한 방식으로 개발할 수 있다. 다만 차이가 있다면 반환 값이 [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html){: target="\_blank" } 또는 [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html){: target="\_blank" }로 감싸져 있다는 것이다. 자세한 내용은 [링크](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-controller){: target="\_blank" }를 참고하자.

## Reactor

[Reactor](https://projectreactor.io/){: target="\_blank" }는 JVM에서 반응형 스트림을 기반으로 비 차단 어플리케이션을 구축하기 위한 반응형 라이브러리이다. 0..1개의 시퀀스를 처리하기 위한 **Mono**와 0..N개의 시퀀스를 처리하기 위한 **Flux** 타입이 있다.

## Getting Start

이제 **주석 기반 컨트롤러** 방식으로 간단한 Web API를 만들어 보자. 프로젝트 생성은 [이전 글의 Getting Start](http://localhost:4000/tutorials/spring-data-r2dbc-tutorial/#getting-start){: target="\_blank" }에서 생성한 프로젝트에서 아래의 의존성을 추가하여 진행하면 된다.

### Gradle

**주석 기반 컨트롤러**는 아래와 같이 WebFlux를 사용하기 위한 의존성만 추가하면 된다.

```gradle
implementation("org.springframework.boot:spring-boot-starter-webflux")
```

<br />

### Controller

**Controller**를 추가하자.

먼저 `PersonController` 클래스를 생성하자. 우리는 API를 위한 Controller를 추가하는 것이므로 `@RestController` 어노테이션을 추가하면 된다. 그런다음 기본 라우팅 경로를 설정하기 위해 `@RequestMapping` 어노테이션을 추가한다. 데이터베이스에 요청받은 데이터를 저장하기 위해 `PersonRepository`를 생성자로 주입받는다.

```kotlin
@RestController
@RequestMapping("/persons")
class PersonController(private val repository: PersonRepository)
```

<br />

저장 요청을 다루는 함수를 추가한다. 사용자의 저장요청을 **Repository**에 전달하고 HTTP 응답을 반환한다. Spring MVC와 다른점은 `@RequestBody`의 타입이 `Mono<Person>`이라는 점과 반환값의 타입이 `Mono<ResponseEntity<Unit>>`이라는 점이다.

```kotlin
@PostMapping
fun create(@RequestBody requestBody: Mono<Person>): Mono<ResponseEntity<Unit>> {
    return requestBody
        .flatMap { repository.save(it) }
        .map { ResponseEntity.created(URI.create("/persons")).build<Unit>() }
}
```

<br />

모든 데이터 목록 조회 요청을 다루는 함수를 추가한다. 사용자의 조회 요청을 받으면 **Repository**에서 데이터를 조회하고 조회한 결과와 함께 HTTP 응답을 반환한다. 목록에 대한 응답값 유형으로 `Flux<Person>`을 사용하였다.

```kotlin
@GetMapping
fun findAll(): Flux<Person> {
    return repository.findAll()
}
```

<br />

단일 데이터 조회 요청을 다루는 함수를 추가한다. URI의 경로에서 ID값을 가져와서 **Repository**에서 데이터를 조회하고 조회한 결과와 함께 HTTP 응답을 반환한다. 만약 ID가 존재하지 않는다면 Bad Request 응답을 반환한다. 반환값의 타입은 `Mono<ResponseEntity<Person>>`을 사용하였다.

```kotlin
@GetMapping("/{id}")
fun findById(@PathVariable("id") id: Long): Mono<ResponseEntity<Person>> {
    return repository.findById(id)
        .map { ResponseEntity.ok(it) }
        .defaultIfEmpty(ResponseEntity.badRequest().build())
}
```

<br />

수정 요청을 다루는 함수를 추가한다. URI의 경로에서 ID값을 가져와서 **Repository**에서 데이터를 조회하고 데이터가 존재한다면 해당 데이터를 수정 하는 함수를 호출한다. 만약 ID가 존재하지 않는다면 Bad Request 응답을 반환한다. 반환값의 타입은 `Mono<ResponseEntity<Unit>>`을 사용하였다.

```kotlin
@PutMapping("/{id}")
fun update(
    @PathVariable("id") id: Long,
    @RequestBody requestBody: Mono<Person>
): Mono<ResponseEntity<Unit>> {
    return repository.existsById(id)
        .zipWith(requestBody)
        .flatMap {
            if (it.t1)
                Mono.just(it.t2.copy(id = id))
            else
                Mono.empty()
        }
        .flatMap { repository.save(it) }
        .map { ResponseEntity.ok().build<Unit>() }
        .defaultIfEmpty(ResponseEntity.badRequest().build())
}
```

<br />

삭제 요청을 다루는 함수를 추가한다. URI의 경로에서 ID값을 가져와서 **Repository**에서 데이터를 조회하고 데이터가 존재한다면 해당 데이터를 삭제하는 함수를 호출한다. 만약 ID가 존재하지 않는다면 Bad Request 응답을 반환한다. 반환값의 타입은 `Mono<ResponseEntity<Unit>>`을 사용하였다.

```kotlin
@DeleteMapping("/{id}")
fun delete(@PathVariable("id") id: Long): Mono<ResponseEntity<Unit>> {
    return repository.existsById(id)
        .flatMap {
            if (it)
                repository.deleteById(id)
                    .then(Mono.just(ResponseEntity.ok().build<Unit>()))
            else
                Mono.just(ResponseEntity.badRequest().build())
        }
}
```

<br />

### Testing

테스트는 **함수형 앤드포인트** 방식의 [Testing](http://localhost:4000/tutorials/spring-web-flux-functional-endpoint-api/#testing){: target="\_blank" }과 동일한 코드를 사용하면 된다. 다만 사용하는 빈이 달라졌으므로 `@WebFluxTest`의 매개변수를 아래와 같이 변경해주어야 한다.

```kotlin
@ExtendWith(SpringExtension::class)
@WebFluxTest(PersonController::class)
internal class PersonRouteTest
```

<br/>

나머지 테스트 함수들은 변경하지 않고 그대로 테스트를 실행하면 된다.

## GIT

해당 튜토리얼에 대한 전체 코드는 Github을 참고하기 바란다. ([https://github.com/veluxer62/toy/tree/webflux-annotated-controller-with-reactor-and-r2dbc/spring-data-r2bc-example](https://github.com/veluxer62/toy/tree/webflux-annotated-controller-with-reactor-and-r2dbc/spring-data-r2bc-example){: target="\_blank" })

## Wrap up

지금까지 Kotlin의 Coroutine을 이용하여 Spring WebFlux의 Annotated Controller 방식의 API를 구현해 보았다. Annotated Controller 방식을 사용하면 Spring MVC에 익숙한 개발자라면 손쉽게 WebFlux API 개발에 적응 할 수 있다. 이러한 장점은 스프링으로 개발하는 개발자가 느낄 수 있는 큰 장점이 아닐까 라는 생각이든다. 또 나는 **Reactor**를 이용한 튜토르리얼을 진행하였는데, 원한다면 Annotated Controller와 Coroutine을 함께 사용할 수도 있다. Coroutine에 대한 의존성을 추가한 후 suspend function만 사용하면 가능하다. 이 부분은 앞에서 보여준 **함수형 앤드포인트** 방식을 참고해서 작성하면 되므로 생략하고자 한다.

이후 글에서는 WebFlux에서 [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc){: target="\_blank" }보다 보편적으로 사용되고 있는 [ReactiveMongoRepository](https://docs.spring.io/spring-data/mongodb/docs/2.2.6.RELEASE/reference/html/#mongo.reactive.repositories){: target="\_blank" }에 대해서 알아보고자 한다.

> 이전 글 보기 : [Spring WebFlux - Functional Endpoint With Coroutine Tutorial](/tutorials/spring-web-flux-functional-endpoint-api/){: target="\_blank" }

<br/>

> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-fn](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-fn){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-controller](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-controller){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc){: target="\_blank" }
>
> [https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html){: target="\_blank" }
>
> [https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html){: target="\_blank" }
>
> [https://projectreactor.io/](https://projectreactor.io/){: target="\_blank" }
>
> [https://spring.io/projects/spring-data-r2dbc](https://spring.io/projects/spring-data-r2dbc){: target="\_blank" }
