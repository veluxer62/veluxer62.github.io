---
layout: single
title: "Synchronous, Asynchronous, Blocking, Non-Blocking"
categories:
  - REFERENCE
tags:
  - synchronous
  - sync
  - asynchronous
  - async
  - blocking
  - non-blocking
toc: true
toc_sticky: true
excerpt: 얼마전에 Blocking과 Non-Blocking에 대해 설명을 해야 할 일이 있었는데, 설명을 Synchronous와 Asynchronous에 대해서 설명한 일이 있었다. 현재 내가 개발을 하면서 많은 곳에서 사용하고 있지만 제대로 알고 사용하고 있지 않다는 것을 알게 되었고, 이에 대해 정리하고자 이 글을 작성한다.
---

## Intro

얼마전에 Blocking과 Non-Blocking에 대해 설명을 해야 할 일이 있었는데, 설명을 Synchronous와 Asynchronous에 대해서 설명한 일이 있었다. 현재 내가 개발을 하면서 많은 곳에서 사용하고 있지만 제대로 알고 사용하고 있지 않다는 것을 알게 되었고, 이에 대해 정리하고자 이 글을 작성한다.

## Synchronous

컴퓨터용어에서 Synchronous란 아래와 같이 설명할 수 있다.

> 현재 작업의 응답과 다음 작업 요청의 타이밍을 맞추는 방식

Synchronous라는 단어에 대한 의미를 찾아보면 수많은 블로그 해석 글들을 볼 수 있었다. 나름대로 여러 글을 읽어보며 가장 잘 해석했다고 생각하는 글을 인용하였다. [관련 블로그 참고](https://evan-moon.github.io/2019/09/19/sync-async-blocking-non-blocking/){: target="\_blank" }

동기적인 코드를 적어보면 아래와 같다.

```kotlin
fun main() {
   println("first")
   println("second")
   println("third")
}
```

즉, 하나의 작업 실행 결과 시점이 다음 작업의 시작 시점이 된다. 그러므로 `first`와 `second`와 `third`는 코드에 적힌 순서대로 실행된다.

## Asynchronous

Asynchronous는 Synchronous와는 반대되는 의미로 해석할 수 있는데 아래와 같이 설명할 수 있다.

> 현재 작업의 응답은 다음 작업 요청의 타이밍이 맞지 않아도 되는 방식

비 동기적 코드를 적어보면 아래와 같다.

```kotlin
import java.util.concurrent.CompletableFuture.runAsync

fun main() {
    println("first")
    runAsync {
        println("second")
    }
    println("third")
}
```

즉, 하나의 작업 실행 결과 시점이 다음 작업의 시작 시점이 되지 않는다. 그러므로 `first`다음 `second`가 아닌 `third`가 먼저 실행되고 `second`가 실행되는 것을 확인할 수 있다. 이는 `third`의 코드가 `second` 코드의 실행 응답 시점에 실행 요청하는 것을 보장하지 않기 때문이다.

## Blocking

컴퓨터 용어에서 Blocking은 아래와 같이 설명할 수 있다.

> 직접 제어할 수 없는 대상의 작업이 끝날 때까지 그 제어권을 넘겨받지 않고 기다리는 방식

대표적으로 I/O를 예로 들 수 있다.

코드를 적으면 아래와 같다.

```kotlin
import java.util.*

fun main() {
    val write = Scanner(System.`in`).nextLine()
    println(write)
}
```

해당 코드는 사용자가 입력을 마칠때까지 아래 출력함수는 동작하지 않을 것이고 사용자 입력이 끝나야지만 프로그램은 입력값을 출력한 후 종료된다.

## Non-Blocking

Non-Blocking은 Blocking과 반대되는 의미로 해석할 수 있는데 아래와 같이 설명할 수 있다.

> 직접 제어할 수 없는 대상의 작업 여부와 관계없이 다음 작업을 수행하는 방식

Multi-Thread를 이용한 처리 방식이 대표적인 Non-Blocking 방식이라 할 수 있다.

Non-Blocking에 대한 코드를 적어보면 아래와 같다.

```kotlin
import java.util.*
import kotlin.concurrent.thread

fun main() {
    thread {
        val write = Scanner(System.`in`).nextLine()
        println(write)
    }
    println("Hello, World!")
}
```

해당 코드를 실행해보면 사용자 입력을 기다리는 동안에도 `Hello, World!`문구를 표시하는 코드는 실행됨을 알 수 있다. 이렇듯 Non-Blocking의 경우 제어할 수 없는 대상의 작업 여부와 관계없이 다음 작업을 수행한다.

## Wrap up

지금까지 **Synchronous**와 **Asyncronous**, **Blocking**과 **Non-Blocking**에 대해서 알아보았다. 많은 블로그에서 이 네가지를 조합한 2:2 매트릭스 예제를 보여주는데 정확히 이해하고 나만의 방식으로 정리하기에는 아직 어렵다고 생각해서 잘 정리해주신 [블로그 링크](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/){: target="\_blank" }로 대체하고자 한다.

관련 내용을 찾다가 **Synchronous**와 **Asyncronous**, **Blocking**과 **Non-Blocking**의 관심사 차이에 대한 내용을 보았는데, 해당 내용을 인용하면서 이 글을 마무리하고자 한다.

> Blocking과 Non-Blocking은 호출되는 함수가 바로 리턴되느냐 마느냐가 관심사이다.

> Synchronous와 Asynchronous는 호출되는 함수의 작업 완료 여부를 누가 신경 쓰느냐가 관심사이다.

<br/>

> [https://evan-moon.github.io/2019/09/19/sync-async-blocking-non-blocking/](https://evan-moon.github.io/2019/09/19/sync-async-blocking-non-blocking/){: target="\_blank" }
>
> [https://velog.io/@codemcd/Sync-VS-Async-Blocking-VS-Non-Blocking-sak6d01fhx](https://velog.io/@codemcd/Sync-VS-Async-Blocking-VS-Non-Blocking-sak6d01fhx){: target="\_blank" }
