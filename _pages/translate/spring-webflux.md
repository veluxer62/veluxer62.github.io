---
permalink: /translate/spring-webflux/
title: Spring Webflux 번역
---

이 문서는 Netty, Undertow 및 Servlet 3.1+ 컨테이너들과 같은 비차단 서버들의 실행을 위한 [Reactive Streams](https://www.reactive-streams.org/) API 기반 반응형 스텍 웹 어플리케이션에 대한 지원을 다룬다. 개별 장에서는 Spring WebFlux 프레임워크, 반응형 `WebClient`, [테스트](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-test) 지원, 그리고 [반응형 라이브러리들](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-reactive-libraries)을 다룬다. Servlet-stack 웹 어플리케이션을 위해, [Web on Servlet Stack](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#spring-web)를 참고하라

# 1. Spring WebFlux

Spring Framework에 포함된 원래 웹 프레임워크인 Spring Web MVC는 Servlet API와 Servlet 컨테이너를 위한 목적으로 제작되었다. 반응형 스텍 웹 프레임워크인 Spring WebFlux는 버전 5.0 이후에 추가되었다. 완전한 비차단이고, Reactive Streams 백 프레셔 지원하며, 그리고 Netty, Undertow, 그리고 Servlet 3.1+ 컨테이너들과 같은 서버에서 실행된다.

두 웹 프레임워크는 소스모듈([spring-webmvc](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc) 및 [spring-webflux](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux))을 반영하고 Spring Framework에서 나란히 공존한다. 각 모듈은 선택사항이다. 어플리케이션은 하나 또는 다른 모듈로 사용될 수 있으며, 경우에 따라, 예를 들어, 반응형 `WebClient`가 있는 Spring MVC 컨트롤러와 같이 둘다 사용될 수 있다.

## 1.1. 개요

왜 Spring WebFlux가 만들어졌나?

하나의 답변은 소수의 스레드와의 동시성을 처리하고 더 적은 하드웨어 리소스로 확장할 수 있는 비차단 웹 스택의 필요성이다. Servlet 3.1은 비차단 I/O를 위한 API를 제공했다. 그러나 이것을 사용하면 계약이 동기식(`Filter`, `Servlet`) 또는 차단(`getParameter`, `getPart`)인 나머지 서블릿 API에서 멀어지게 된다. 이는 새로운 공통 API가 실행시점의 모든 비차단 작업에 기반역할을 하게된 동기였다. 그것은 비동기, 비차단 공간에 잘 구축된 서버(Netty와 같은 ) 때문에 중요하다.

다른 대답은 함수형 프로그래밍이다. Java 5에 어노테이션을 추가하여 기회(어노테이션이 달린 REST 컨트롤러 또는 유닛 테스트와 같은)가 생긴 것과 마찬가지로, Java 8에 람다식을 추가함으로써 Java의 함수형 API에 대한 기회가 생겨났다. 이는 비동기식 로직을 선언적으로 구성할 수 있는 비차단 어플리케이션과 연속 스타일 API (`CompletableFuture`와 [ReactiveX](http://reactivex.io/)에서 널리 사용되는)의 하나의 유행이다. 프로그래밍 모델 수준에서, Java 8은 Spring WebFlux가 어노테이션이 선언된 컨트롤러와 함께 함수형 웹 앤드포인트를 제공할 수 있도록 했다.

## 1.1.1. "Reactive" 정의

우리는 "비차단"과 "함수형"에 대해 언급했지만, 반응형의 의미는 무엇인가?

"반응형"이라는 용어는 I/O 이벤트에 반응하는 네트워크 컴포넌트, 마우스 이벤트에 반응하는 UI 컨트롤러 등의 변경에 반응하기 위해 주위에 구축된 프로그래밍 모델을 만한다.

그런 의미에서, 비차단은 반응형이다, 왜냐하면, 차단되는 대신, 우리는 이제 작업이 완료되거나 데이터가 이용가능해짐과 같은 알림에 반응하는 방식에 있기 때문이다.

스프링 팀에서 우리가 "반응형"과 연관되어 있는 또 다른 중요한 메커니즘이 있는데 그것은 비차단 백 프레셔이다. 동기, 명령형 코드, 차단 요청은 호출자가 기다리게 강제하는 자연적 형태의 백 프레셔를 제공한다. 비차단 코드에서는, 빠른 프로듀서가 목적지에서 어쩔줄 몰라 하지 않도록 이벤트의 속도를 조절하는 것이 중요해진다.

Reactive Streams는 백 프레셔와 함께 비동기 구성요소 간의 상호작용을 정의하는 [작은 사양](https://github.com/reactive-streams/reactive-streams-jvm/blob/master/README.md#specification)(Java 9에서도 [채택](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html))이다. 예를 들어 데이터 레파지토리([Publisher](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Publisher.html)로 작동)는 HTTP 서버([Subscriber](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Subscriber.html)로 동작)가 응답에 쓸 수 있는 데이터를 생성할 수 있다. Reactive Streams의 주요 목적은 구독자가 게시자가 데이터를 얼마나 빨리 또는 얼마나 느리게 생산하는지 제어할 수 있도록 하는 것이다.

> **일반적인 질문: 만약 게시자가 속도를 늦출 수 없다면?**<br/>
> Reactive Streams의 목적은 단지 메커니즘과 결계를 설정하는 것이다. 만약 게시자가 속도를 늦출 수 없다면, 버퍼, 드랍, 또는 실패 여부를 결정해야 한다.

## 1.1.2. Reactive API

Reactive Streams는 중요한 역할을 수행한다 / 상호작용을 위한 /
그것은 라이브러리들과 인프라스트럭처 컴포넌트 들에 흥미를 가진다 / 하지만 유용함이 적다 / 어플리케이션 API로는 / 왜냐하면 그것은 너무 낮은 수준이기 때문이다 /
어플리케이션들은 높은 수준과 풍부함, 비동기 로직을 구성한 함수형 API를 원한다 / Java8 `Stream` API와 비슷한 / 하지만 단지 collections를 위한 것이 아닌 /
이것은 역할이다 / reactive 라이브러리 행위 /

Reactor는 Spring WebFlux의 선택된 reactive library이다 / 그것은 `Mono`와 `Flux` API 유형을 제공한다 / 0..1(`Mono`)과 0..N(`Flux`)의 데이터 시퀀스를 사용하기 위해 / 풍부한 오퍼레이터 세트를 통해 / ReactiveX 오퍼레이터 문법이 적용된 /
Reactor is a Reactive Streams 라이브러리 이다 / 그리고, 그러므로, 모든 오퍼레이터들은 non-blocking 백 프레셔를 지원한다 /
Reactor는 server-side Java에 강하게 관심이 집중되어 있다 / 그것은 Spring과 긴밀하게 협력하여 개발되었다 /

WebFlux는 Reactor를 요구한다 / 주요한 의존성으로 / 하지만 그것은 다른 reactive 라이브러리와 상호운용가능 하다 / Reactive Streams를 통해서 /
일반적인 규칙에서 / WebFlux API는 분명한 `Publisher`를 받아들인다 / 입력으로써 / 그것을 Reactor 유형에 적응시키다 / 내부적으로 / 그것을 사용하다 / 그리고 `Flux` 또는 `Mono`로 반환하다. / 출력으로써 / 그래서 어떤 `Publisher`도 통과할 수 있다 / 입력으로써 / 그리고 출력 동작을 적용할 수 있다 / 하지만 출력에 적응할 필요가 있다 / 사용하기 위해 / 다른 reactive 라이브러리와 함께 / 언제든 실행가능한 (예를 들어 주석이달린 컨트롤러들), WebFlux는 투명하게 적응하다 / RxJava 또는 다른 reactive 라이브러리의 사용에 / Reactive Libraries를 보아라 / 더 자세한 것을 위해 /

> Reactive API에 추가로 / WebFlux는 Coroutines API들과 함께 사용될 수 있다 / 코틀린에서 / 좀더 필수적인 형태의 프로그래밍을 제공하는 / 이후의 Kotlin 코드 예시들은 Coroutines API로 제공될 것이다. /

## 1.1.3. 프로그래밍 모델들

`spring-web` 모듈은 반응형 토대를 포함하고 있다 / Spring WebFlux의 기초가 되는 / HTTP 추상화, 서버와 코덱들을 지원하는 Reactive Streams 어뎁터들, 그리고 주요 `WebHandler` API / Servlet API와 비교가능한 / 하지만 non-blocking 계약과 함께

이 토대로 / Spring WebFlux는 2가지 프로그래밍 모델들의 선택을 제공한다. /

- 주석처리된 컨트롤러: / Spring MVC와 일치한다 / 그리고 같은 주석들에 기반한다 / `spring-web` 모둘로부터 /
  Spring MVC와 WebFlux 컨트롤러들 둘다 반응형 (Reactor와 RxJava) 반환 유형을 지원한다. / 그리고, 그 결과 / 그것들을 쉽게 떨어뜨려서 말하는게 쉽지 않다. /
  하나의 주목할만한 차이점은 / WebFlux는 또한 반응형 `@RequestBody` 인자들을 지원한다 /

- 함수형 끝점: / 람다-기반, 가벼움, 그리고 함수형 프로그래밍 모델 /
  이것을 생각할 수 있다 / 작은 라이브러리 또는 유틸리티들의 세트로 / 어플리케이션이 경로를 잡거나 요청을 다루는데 사용할 수 있다 /
  주석처리된 컨트롤러와 큰 다른점은 / 그 어플리케이션은 요청을 다루는 것에 대한 책임이 있다 / 의도를 정의하는 것과 대비해서 / 주석과 콜백을 통한

## 1.1.4. 적용가능성

Spring MVC 또는 WebFlux? /
당연한 질문 / 물어보는 것 / 하지만 한가지 / 부적절한 이분법을 세우는 /
사실 / 둘다 동작한다 / 가능한 선택 범위를 확장하기 위해 /
둘다 디자인 되었다 / 연속성과 일관성을 위해 / 각각 함께 / 그것들은 나란히 가능하다 / 그리고 응답한다 / 쌍방의 이익을 도모 /
다음의 다이어그램은 보여준다 / 어떻게 두개가 관계되는지 / 무엇을 그들이 일반적으로 가지고 있는지 / 무엇이 각각 유일하게 지원하는지

![spring-mvc-and-webflux-venn](/asserts/images/translate/spring-webflux/spring-mvc-and-webflux-venn.png)

우리는 제안한다 / 당신이 구체적인 포인트들을 고려하기를: /

- 만약 당신이 Spring MVC 어플리케이션을 가지고 있다면 / 잘 동작하는 / 변경할 필요 없다 /
  필수적인 프로그래밍은 가장 쉬운 방법이다 / 작성, 이해, 그리고 코드를 디버깅 하기 위한 /
  당신은 라이브러리의 최대 선택을 가지고 있다 / 이례로 / 역사적으로 / 대부분은 blocking이다. /

- 만약 당신이 이미 non-blocking 웹 스텍을 선택했다면 / Spring WebFlux는 동일한 실행 모델 혜택을 제공한다 / 이 공간에서 / 그리고 또한 서버의 선택을 제공한다 (Netty, Tomcat, Jetty, Undertow, 그리고 Servlet 3.1+ 컨테이너들), / 프로그래밍 모델들의 선택 (주석이달린 컨트롤러들, 함수형 웹 끝점) / 그리고 반응형 라이브러리의 선택 (Reactor, RxJava, 또는 다른것들). /

- 만약 당신이 가벼운것에 관심이 있다면, / 함수형 웹 프레임워크 / 사용을 위한 / Java 8 람다 또는 Kotlin과 함께 / 당신은 Spring WebFlux 함수형 웹 끝점을 사용할 수 있다. / 그것은 또한 좋은 선택이다 / 더 작은 어플리케이션 또는 마이크로서비스 / 적은 복잠도의 요구사항과 함께 / 혜택을 줄수 있다 / 더 큰 투명성과 통제로 부터 /

- 마이크로서비스 아키텍쳐에서 / 당신은 섞인 어플리케이션을 가질 수 있다 / Spring MVC 또는 Spring WebFlux 컨트롤러와 함께 / 또는 Spring WebFlux 함수형 끝점과 함께 / 지원을 가지는 것 / 동일한 주석기반 프로그래밍 모델을 위한 / 프레음워크 둘다에서 / 그것을 쉽게 만든다 / 지식을 재사용할 수 있도록 / 또한 제대로된 도구를 사용하는것 동안 / 제대로된 일을 위해 /

- 단순한 방법 / 어플리케이션을 평가하는 / 그것들의 의존성을 확인하는 것이다. / 만약 당신ㅇ blocking 영속성 API들 (JPA, JDBC) 또는 네트워킹 API들 / 사용할 수 있는 / Spring MVC는 가장좋은 선택이다 / 일반적인 아키텍처를 위한 / 적어도 / 그것은 기술적으로 실행가능하다 / Reactor와 RxJava 둘다 함께 / blocking 요청을 수행하기 위해 / 분리된 thread에서 / 하지만 당신은 대부분의 non-blocking 웹 스텍을 만들지 않을 것이다.

- 만약 Spring MVC 엎ㄹ리케이션을 가지고 있다면 / 원격 서비스 호출과 함께 / 반응형 `WebClient`를 시도해봐라 /
  당신은 반응형 타입(Reactor, RxJava, 또는 다른것들)을 직접적으로 반환받을 수 있다 / Spring MVC 컨트롤러 함수로부터 /
  더큰 응답 요청과 상호운용성 / 요청 중에 / 좀더 드라마틱한 혜택 /
  Spring MVC 컨트롤러들은 또한 다른 반응형 컨포턴트들을 호출할 수 있다 /

- 만약 당신이 큰 팀을 가지고 있다면 / 가파른 학습 곡선을 생각해야 한다 / 변경하는것을 / non-blocking, 함수형, 그리고 선언적 프로그래밍으로 /
  하나의 부분적인 방법은 / 시작하는 것 / 전체 변경없이 / 반응형 `WebClient`를 시작하는 것이다. /
  넘어서 / 작게 시작하고 이익을 측정하는 것을 /
  우리는 예상하다 / 넓은 범위의 어플리케이션을 위해 / 변경은 불필요하다 / 만약 당신이 확실하지 않다면 / 무엇이 찾을 수 있는 혜택인지 / 기대어 출발하다 / 어떻게 non-blocking I/O가 동작하는지 (예를들어 동시성 / 단일 스레드 node.js 에서) / 그리고 그것의 효과

## 1.1.5. 서버들

Spring WebFlux는 지원된다 / Tomcat, Jetty, Servlet 3.1 컨테이너에 대해 / 실행시 서블릿을 사용하지 않는 것 뿐만 아니라 / Netty와 Undertow와 같이 /
모든 서버는 낮은 수준과 일반적인 API에 적응한다 / 그래서 고수준 프로그래밍 모델들은 전체 서버에 걸체 지원을 받는다 /

Spring WebFlux는 서버를 시작하고 종료하는 기능이 내장되어 있지 않다. / 하지만 / 어플리케이션을 조립하는 것은 쉽다 / 스프링 설정과 WebFlux 기반 / 그리고 그것을 실행한다 / 적은 줄의 코드와 함께

Spring Boot는 WebFlux starter를 가지고 있다 / 이 절차를 자동화하는 / 기본값으로 / starter는 Netty를 사용한다 / 하지만 Tomcat, Jetty, 또는 Undertow로 변경하는 것은 쉽다 / Maven 또는 Gradle 의존성을 변경하는 것에 의해서 / Spring Boot는 Netty를 기본으로 한다 / 왜냐하면 그것은 비동기, non-blocking 공간에서 좀더 널리 사용되고 있기 때문이다 / 그리고 클라이언트와 서버가 자원을 공유하도록 한다 /

Tomcat과 Jetty는 Spring MVC와 WebFlux 둘다 사용되어 진다 /
명심해라 / 하지만 / 그 방식은 / 그들이 사용되는 / 아주 다르다 /
Spring MVC는 Servlet blocking I/O에 의존한다 / 그리고 어플리케이션들이 Servlet API를 직접적으로 하용하도록 한다 / 만약 그들이 필요로 한다면 /
Spring WebFlux는 Servlet 3.1 non-blocking I/O에 의존한다 / 그리고 Servlet API를 사용한다 / 저수준 어뎁터 뒤에서 / 그리고 노출하지 않는다 / 직접 사용하기 위한 /

Undertow를 위해 / Spring WebFlux는 Undertow API들을 직접적으로 사용한다 / Servlet API 없이 /

## 1.1.6. 성능

성능은 많은 특징과 의미를 가지고 있다 /
반응형과 non-blocking은 일반적으로 어플리케이션을 빠르게 동작하도록 만들지 않는다.
그들은 할수있다 / 약간의 사례에서 / (예를 들어, 만약 `WebClient`를 사용하는 것 / 원격요청을 실행하기위해 / 병렬로) / 대체로 / 그것은 더 많은 일을 요청한다 / non-blocking 방법을 하기위한 / 그리고 요구된 프로세싱 시간이 약간 증가될 수 있다 /

열쇠는 / 반응형과 non-blocking의 혜택이 예측된 / 작고, 정해진 쓰레드 수 그리고 적은 메모리의 확장 능력이다. / 그것은 어플리케이션이 로딩된 상태에서 좀더 탄력성있도록 만들어준다 / 왜냐하면 그들은 좀더 예측가능한 방법으로 규모를 축소시킨다 / 이 혜택을 관찰한 것에 따라 / 하지만 / 당신은 약간의 대기시간을 필요로 한다 / (느리고 예측불가능한 네트워크 I/O가 섞인 것을 포함해서) / 그것이 어디에서 반응형 스텍이 시작되고 / 그 강점을 보여주기 위해 / 그리고 다른점들이 트라마틱할 수 있는 지

## 1.1.7. 동시성 모델

Spring MVC와 Spring WebFlux 둘다 주석이 달린 컨트롤러들을 지원한다 / 하지만 열쇠가 있다 / 다른 동시성 모델과 기본적인 가정 / blocking과 쓰레드를 위한 /

Spring MVC에서 / (그리고 일반적인 서블릿 어플리케이션에서) / 가정한다 / 어플리케이션은 최근 쓰레드를 차단할수 있다 / (예를들어, 원격 요청을 위해) / 그리고 / 이러한 이유로 / 서블릿 컨테이너들은 큰 쓰레드 풀을 이용한다. / 잠재적인 차단을 흡수하기 위해 / 요청을 다루는 동안 /

Spring WebFlux에서 / (그리고 일반적인 비차단 서버들에서) / 가정한다 / 어플리케이션은 차단하지 않는다 / 그리고 / 그러므로 / 비 차단 서버들은 작고 정해진 크기의 쓰레드 풀을 사용한다 (이벤트 반복 워커들) / 요청을 다루기 위해

> "규모를 키우는 것"과 "적은 숫자의 쓰레드"는 모순되는 것 처럼 들린다 / 하지만 최신 쓰레드를 절대 차단하지 않는 것은 / (그리고 콜백에 대신 의존하는 것은 ) / 의미한다 / 당신이 추가적인 쓰레드가 필요하지 않다는 것을 / 차단하지 않는 요청과 같은 / 흡수하기 위한

### 차단 API를 발동하는 것

만약 당신이 차단 라이브러리를 필요로 한다면? / Reactor와 RxJava 둘다 `publishOn` 오퍼레이터를 제공한다 / 다른 쓰레드에서 계속 수행하기 위해 /
그것은 의미한다 / 출구로 쉽게 탈출할 수 있음을 /
명심해라 / 하지만 / 그 차단 API들은 잘 어울리지 않는다 / 이 동시성 모델을 위해

### 변경가능한 상태

Reactor와 RxJava에서 / 당신은 논리를 선언한다 / 오퍼레이터를 통해서 / 그리고 / 실행시점에 / 반응형 파이프라인은 형성된다 / 어디에 데이터가 순차적으로 처리되고 있는지 / 구별된 스테이지들에서 /
이것의 주된 이점은 / 그것은 어플리케이션을 자유롭게 한다 / 가지는 것으로 부터 / 변경 가능한 상태를 지키이 위해 / 왜냐하면 어플리케이션의 코드는 / 파이프라인 이내의 / 절대 동시에 발동되지 않는다.

### 쓰레딩 모델

어떤 쓰레드가 당신이 예상할 수 있는가? / 보기 위해 / 서버에서 / Spring WebFlux와 함께 동작하는 /

- "vanilla" Spring WebFlux 서버에서 / (예를 들어, / 데이터 접근이 없고 다른 선택적인 의존석이 없는), 당신은 하나의 쓰레드를 예상할 수 있다 / 서버와 여러 다른것들을 위해 / 요청 프로세싱을 위한 / (전통적으로 CPU 코어 개수 만큼) /
  서블릿 컨테이너들, / 하지만, / 아마 좀더 많은 쓰레드로 시작할 것이다 / (예를들어, 톰켓에서 10개) / 서블릿(차단)과 서블릿3.1(비차단) I/O 사용 둘다의 지원에서 /

- 반응형 `WebClient`는 이벤트 순환 스타일로 동작한다 /
  그래서 당신은 작고, 정해신 실행 쓰레드 개수를 볼 수 있다 / 관련된 / (예를들어, `reactor-http-nio-` / Reactor Netty 커넥터와 함께) /
  하지만, / 만약 Reactor Netty가 사용된다면 / 클라이언트와 서버 둘다를 위해, / 두개는 이벤트 순환 자원을 공유한다 / 기본적으로 /

- Reactor와 RxJava는 쓰레드 풀 추상화를 제공한다, / Schedulers라고 불리는 / 사용하기 위해 / `publishOn` 오퍼레이터와 함께 / 사용되는 / 진행을 바꾸기 위해 / 다른 쓰레드 풀로 /
  스케줄러는 이름을 가지고 있다 / 특정한 동시성 전략을 제안하는 / 예를 들어, / "paralled" / (CPU에 묶인 작업 / 제한된 숫자의 쓰레드와 함께) / 또는 "elastic" / (I/O 작업에 묶인 / 많은 숫자의 쓰레드와 함께) / 만약 이런 쓰레드들을 본다면 / 그것은 의미한다 / 어떤 코드가 특정한 쓰레드 풀 `Scheduler` 전략을 사용하고 있는 것이라는 것을 /

- 데이터 접근 라이브러리와 다른 써드파티 의존성들은 그들이 가진 쓰레드들을 또한 생성하고 소유할 수 있다.

### 설정

Spring Framework는 지원을 제공하지 않는다 / 서버의 시작과 종료를 / 서버를 위한 쓰레딩 모델을 설정하는 것은, / 당신이 특정 서버 설정 API들을 사용하는것이 필요하다, / 또는 / 만약 Spring Boot를 사용한다면, / Spring Boot 설정 옵션을 확인해라 / 각각의 서버를 위해 /
당신은 `WebClient`를 직접 설정할 수 있다 /
다른 모든 라이브러리들을 위해 / 그들의 존중할만한 문서를 봐라 /

## 1.2. Reactive Core

`spring-web` 모듈은 다음의 기반 지원을 포함한다 / 반응형 웹 어플리케이션들을 위한 /

- 서버 요청 프로세싱을 위해 / 두개의 지원 수준이 있다 /

  - HttpHandler: 기본적인 계약 / HTTP 요청을 다루기 위한 / 비 차단 I/O와 반응형 스트림 백 프레셔, / 어뎁터들과 함께 / Reactor Netty, Undertow, Tomcat, Jetty, 그리고 아무 Servlet 3.1+ 컨테이너 /
  - WebHandler API: 약간 높은 수준, / 일반적인 목적의 웹 API / 요청을 다루기 위한 / 구체적인 프로그래밍 모델들 위에 / 주석이 달린 컨트롤러들 그리고 함수형 끝점과 같은 / 구축되어 졌다. /

- 클라이언트를 위해 / 기본적인 `ClientHttpConnector` 계약이 있다 / HTTP 요청들을 수행하기 위해 / 비 차단 I/O와 반응형 스트림 백 프레셔와 함께, / 어뎁터들과 함께 / Reactor Netty를 위해 / 그리고 반응형 Jetty HttpClient를 위해. /
  높은 수준의 WebClient는 / 어플리케이션에서 사용된 / 이 기본적인 계약을 구축한다. /

- 클라이언트와 서버를 위해 / 코덱을 만들다 / 사용하기 위한 / HTTP 요청ㅇ과 응답 컨텐트를 직렬화와 역직렬화를 위해 /

## 1.2.1. `HttpHandler`

HttpHandler는 단순한 계약이다 / 하나의 함수를 가진 / 요청과 응답을 다루기 위한 /
그것은 의도적으로 작게, / 그리고 그것의 메인, 그리고 단지 목적은 작은 추상화가 되는 것이다 / 다른 HTTP 서버 API들 이면에 /

아래 테이블은 지원되는 서버 API들을 설명한다.

| Server name           | Server API used                                                                  | Reactive Streams support                                            |
| :-------------------- | :------------------------------------------------------------------------------- | :------------------------------------------------------------------ |
| Netty                 | Netty API                                                                        | Reactor Netty                                                       |
| Undertow              | Undertow API                                                                     | spring-web: Undertow to Reactive Streams bridge                     |
| Tomcat                | Servlet 3.1 non-blocking I/O; Tomcat API to read and write ByteBuffers vs byte[] | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Jetty                 | Servlet 3.1 non-blocking I/O; Jetty API to write ByteBuffers vs byte[]           | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Servlet 3.1 container | Servlet 3.1 non-blocking I/O                                                     | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |

아래의 테이블은 서버 의존성들을 설명한다 (또한 지원버전을 보라): /

| Server name   | Group id                | Artifct name               |
| :------------ | :---------------------- | :------------------------- |
| Reactor Netty | io.projectreactor.netty | reactor-netty              |
| Undertow      | io.undertow             | undertow-core              |
| Tomcat        | org.apache.tomcat.embed | tomcat-embed-core          |
| Jetty         | org.eclipse.jetty       | jetty-server,jetty-servlet |

아래의 코드 조각들은 `HttpHandler` 어뎁터들의 사용을 보여준다 / 각각의 서버 API들과 함께: /

### Reactor Netty

```
val handler: HttpHandler = ...
val adapter = ReactorHttpHandlerAdapter(handler)
HttpServer.create().host(host).port(port).handle(adapter).bind().block()
```

### Undertow

```
val handler: HttpHandler = ...
val adapter = UndertowHttpHandlerAdapter(handler)
val server = Undertow.builder().addHttpListener(port, host).setHandler(adapter).build()
server.start()
```

### Tomcat

```
val handler: HttpHandler = ...
val servlet = TomcatHttpHandlerAdapter(handler)

val server = Tomcat()
val base = File(System.getProperty("java.io.tmpdir"))
val rootContext = server.addContext("", base.absolutePath)
Tomcat.addServlet(rootContext, "main", servlet)
rootContext.addServletMappingDecoded("/", "main")
server.host = host
server.setPort(port)
server.start()
```

### Jetty

```
val handler: HttpHandler = ...
val servlet = JettyHttpHandlerAdapter(handler)

val server = Server()
val contextHandler = ServletContextHandler(server, "")
contextHandler.addServlet(ServletHolder(servlet), "/")
contextHandler.start();

val connector = ServerConnector(server)
connector.host = host
connector.port = port
server.addConnector(connector)
server.start()
```

### Servlet 3.1+ Container

WAR를 어느 Servlet 3.1+ 컨테이너로 배포하기 위해, / 당신은 `AbstractReactiveWebInitializer`를 확장하고 포함할 수 있다 / WAR안에 /
그 클레스는 `ServletHttpHandlerAdapter`를 `HttpHandler`로 감싼다 / 그리고 그것을 `Servlet`으로 등록한다

## 1.2.2. `WebHandler` API

`org.springframework.web.server` 패키지는 `HttpHandler` 계약을 기반으로 한다 / 일반적인 목적의 웹 API를 제공하기 위해 / 요청을 수항하는 것을 위한 / 여러 `WebExceptionHandler`, 여러 `WebFiler` 체인과, 그리고 단일 `WebHandler` 컴포넌트를 통해 /
그 체인은 결합될 수 있다 / `WebHttpHandlerBuilder`와 함께 / Spring `ApplicationContext`를 단순히 가리키는 것에 의해 / 컴포넌트들이 자동으로 감지된 / 그리고/또는 / 컴포넌트들이 등록된 것에 의해 / 빌더와 함께 /

`HttpHandler`가 단순한 목표를 가지는 동안 / 다른 HTTP 서버의 사용을 추상화한, / `WebHandler` API는 겨냥한다 / 더 넓은 특징들의 세트를 제공하기 위해 / 일반적으로 웹 어플리케이션에서 사용되는 / 다응과 같은 :

- 사용자 세션 / 속성과 함께
- 속성의 요청
- 해결된 `Locale` 또는 `Principal` / 요청을 위한
- 파싱되고 캐싱된 폼 데이터에 접근
- 추상화 / 멀티파트 데이터를 위한
- 기타..

### 특별한 빈 유형

아래 테이블은 컴포넌트들을 나열한다 / `WebHttpHandlerBuilder`가 Spring ApplicationContext가 자동으로 감지할 수 있는, / 또는 그것과 함께 직접적으로 등록될 수 있는 /

| Bean Name                    | Bean type                    | Count | Description                                                                                                                                                                           |
| :--------------------------- | :--------------------------- | :---- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `<any>`                      | `WebExceptionHandler`        | 0..N  | 다루는 것을 제공한다 / 예외들을 위한 / `WebFilter` 인스턴스들과 타겟 `WebHandler`의 체인으로 부터 ./ 더 자세한 정보를 위해, Exceptions를 봐라                                         |
| `<any>`                      | `WebFilter`                  | 0..N  | 가로채는 형태의 로직을 필터체인과 타겟 `WebHandler`의 rest 전과 후에 적용한다. / 더 자세한 정보를 위해, Filters를 봐라                                                                |
| `webHandler`                 | `WebHandler`                 | 1     | 요청을 위한 헨들러                                                                                                                                                                    |
| `webSessionManager`          | `WebSessionManager`          | 0..1  | 매니저 / `WebSession` 인스턴스를 위한 / 노출된 / 함수를 통해 / `ServerWebExchange` 위에. / `DefaultWebSessionManger`가 기본이다.                                                      |
| `serverCodecConfigurer`      | `ServerCodecConfigurer`      | 0..1  | `HttpMessageReader` 인스턴스에 접근하기 위해 / 폼 데이터와 멀티파트 데이터를 분석하기 위한 / 노출된 / `ServerWebExchange` 함수를 통해. / `ServerCodecConfigurer.create()`가 기본이다. |
| `localeContextResolver`      | `LocaleContextResolver`      | 0..1  | 리졸버 / `LocaleContext`를 위한 / 노출된 / `ServerWebExchange` 함수를 통해. / `AcceptHeaderLocaleContextResolver`가 기본이다.                                                         |
| `forwardedHeaderTransformer` | `ForwardedHeaderTransformer` | 0..1  | 전달된 유형의 헤더들을 수행하기 위해, / 그것들을 추출하고 제거하는 것 중 하나 또는 그것들을 제거하는 것만 . / 사용되지 않는 것이 기본이다.                                            |

### Form Data

`ServerWebExchange`는 다음 함수를 노출한다 / 폼 데이터를 접근하기 위해

```
suspend fun getFormData(): MultiValueMap<String, String>
```

`DefaultServerWebExchange`는 설정된 `HttpMessageReader`를 사용한다 / 폼 데이터를 분석하기 위해 / (`application/x-www-form-urlencoded`) / `MultiValueMap`으로. /
기본적으로, / `FormHttpMessageReader`는 설정된다 / 사용을 위해 / `ServerCodecConfigurer` 빈에 의해 / (Web Handler API를 보라)

### Multipart Data

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-multipart)

`ServerWebExchange` 이후의 함수로 노출된다. / 멀티파트 데이터에 접근하기 위한: /

```
suspend fun getMultipartData(): MultiValueMap<String, Part>
```

`DefaultServerWebExchange`는 설정된 `HttpMessageReader<MultiValueMap<String, Part>>`로 사용된다 / `multipart/form-data`를 `MultiValueMap`으로 분석하기 위해 /
현재 / Synchronoss NIO Multipart가 유일한 서드파티 라이브러리이다 / 지원되는 / 그리고 유일한 라이브러리이다 / 우리가 비차단을 위한 것으로 알고 있는 / 멀티파트 요청의 분석하는 /
그것은 가능하다 / `ServerCodeConfigurer` 빈을 통해 / (Web Handler API를 보라)

스트리밍 문화에서 멀티파트 데이터를 분석하기 위해, / 당신은 `Flux<Part>`를 사용할 수 있다 / `HttpMessageReader<Part>`로 부터 반환된 / 대신에 /
예를 들어, / 주석이 달린 컨트롤러인 경우에 / `@RequestPart`의 사용은 / `Map`과 같은것으로 구현된 / 개별적인 파트들에 접근한다 / 이름에 의해 / 그리고, 그러므로, / 분석한 멀티파트 데이터를 요청한다 / 완전히 /
대조적으로 / 당신은 `@RequestBody`를 사용할 수 있다 / 그 내용을 `Flux<Part>`로 해석하기 위해 / `MultiValueMap`을 수집하지 않고 /

### Forwarded Headers

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-multipart)

요청이 가는것 처럼 / 프록시 (로드 밸런서와 같이), 호스트, 포트, 그리고 스키마를 통해 / 아마 변경한다 / 그리고 그것을 도전하게 만든다 / 클라이언트 관점으로 부터 / 링크를 만들기 위해 / 올바른 호스트, 포트, 그리고 스키마를 가리키기 위한 /

RFC 7239는 `Forwared` HTTP 헤더의 정의를 내린다 / 프록시가 사용 가능한 / 정보를 제공하기 위해 / 원래 요청에 대한 /
다른 비 표준적인 헤더들 또한 있다 / `X-Forwarede-Host`, `X-Forwarded-Port`, `X-Forwarded-Proto`, `X-Forwarded-Ssl`, 그리고 `X-Forwarded-Prefix`를 포함해서 /

`ForwardedHeaderTransformer`는 컴포턴트이다 / 요청의 호스트, 포트, 그리고 스키마를 수정하는 / forwarded 헤더를 기반으로, / 그리고 그들은 그것들의 헤더를 제거한다. /
당신은 그것을 빈으로 선언할 수 있다 / `forwardedHeaderTransformer`의 이름과 함께, / 그리고 그것은 탐색되고 사용된다 /

보안 고려사항이 있다 / forwarded 헤더를 위한, / 어플리케이션이 알지 못했던 이례로 / 만약 헤더가 더해진 경우, / 프록시에 의해 / 계획했던 대로, / 또는 악의적인 클라이언트에 의해 /
이것은 왜 프록시가 신뢰구간에서 설정되어야 하는 이유이다 / 신뢰할 수 없는 forwarded 트래픽을 제거하기 위해 / 외부에서 들어온 /
당신은 또한 `ForwardedHeaderTransformer`로 설정할 수 있다 / `removeOnly=true`와 함께 / 사용되지 않는 헤더들을 제거하는 것인

> 5.1에서 `ForwordedHeaderFilter`는 사용되지 않는다 / 그리고 대체된다 / `ForwardedHeaderTransformer`에 의해 / 그래서 forwarded 헤더는 일찍 실행된다 / 교환이 발생하기 전에 /
> 만약 필터가 설정되면, / 그것은 필터 목록에서 제외된다 / 그리고 `ForwardedHeaderTransformer`는 대신에 사용된다. /

## 1.2.3. Filters

`WebHandler` API에서, / 당신은 `WebFilter`를 사용할 수 있다 / 가로채는 스타일 논리를 적용하기 위해 / filter와 타겟 `WebHandler`의 실행 체인의 rest 전과 후에 /
WebFlux Config를 사용할 때, / `WebFilter`를 등록하는 것은 Spring 빈과 같은 것을 정의하는 것 만큼 단순하다 / 그리고 (선택적으로) `@order`를 사용하는 것에 의해 우선순위를 표현하는 것도 / 빈을 정의하거나 `Ordered`를 구현하는것에 의해

### CORS

Spring WebFlux는 CORS 설정을 위한 결이고운 지원을 제공한다 / 컨트롤러의 주석을 통해 /
하지만, / 당신이 Spring Security와 함께 사용한다면, / 우리는 내장된 `CorsFilter`에 의존하기를 조언한다, / 그것은 반드시 Spring Security의 필터 체인에 앞서 정렬되어 진 /

더 자세한것을 위해 CORS 섹션과 webflux-cors.html을 봐라. /

## 1.2.4. Exceptions

`WebHandler` API에서 / 당신은 `WebExceptionHandler`를 사용할 수 있다 / 예외를 다루기 위해 / `WebFilter` 인스턴스 체인과 타겟 `WebHandler`로 부터. /
WebFlux Config를 사용할 때, / `WebExceptionHandler`를 등록하는 것은 Spring 빈을 등록하는 것 만큼 쉽다 / (선택적으로) `@Order`를 사용해 우선순위를 표현하는 것도 / 빈을 정의하는 것 또는 `Ordered`를 구현하는것에 의해 /

아래 테이블은 가능한 `WebExceptionHandler` 구현들을 설명한다:

| Exception Handler                       | Description                                                                                                                                                                  |
| :-------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ResponseStatusExceptionHandler`        | `ResponseStatusException`유형의 예외를 다루는 것을 제공한다 / 예외의 HTTP 상태 코드 응답을 설정하는 것에 의해                                                                |
| `WebFluxResponseStatusExceptionHandler` | `ResponseStatusExceptionHandler`의 확장이다 / `@ResponseStatus` 주석의 HTTP 상태 코드 또한 결정할 수 있는 / 어떤 예외도 / <br/><br/> 이 핸들러는 WebFlux Config에 정의된다 / |

## 1.2.5. Codecs

`spring-web`과 `spring-core` 모듈은 바이트 내용을 직렬화 및 역직렬화 하는 것을 제공한다 / 고수준 객체를 주고받는 / Reactive Streams 백 프레셔와 같은 비 차단 I/O를 통해서 /
다음은 이 지원에 대해 설명한다 /

- `Encoder`와 `Decoder`는 저수준 계약이다 / HTTP 메시지 내용을 인코딩 및 디코딩 하기 위해 /
- `HttpMessageReader`와 `HttpMessageWriter`는 계약이다 / HTTP 메시지 내용을 인코딩 및 디코딩 하기 위해 /
- `Encoder`는 `EncoderHttpMessageWriter`로 감싸질 수 있다 / 웹 어플리케이션에서 그것을 사용에 맞게 개조하기 위해 / `Decoder`가 `DecoderHttpMeassageReader`로 감싸질 수 있는 동안 /
- `DataBuffer`는 다른 바이트 버퍼 표현을 추상화 한다 (예: Neety `ByteBuf`, `java.nio.ByteBuffer`, 기타 등등) / 그리고 전체 코덱이 수행하는 것이다. / "Spring Core" 섹션안의 Data Buffers와 Codecs를 보라 / 이 주제의 더 많은 것을 위해 /

`spring-core`모듈은 `byte[]`, `ByteBuffer`, `DataBuffer`, `Resource`, 그리고 `String` 인코더와 디코더 구현을 제공한다. /
`spring-web`모듈은 Jackson JSON, Jackson Smile, JAXB2, Protocol Buffers 및 다른 인코더와 디코더를 제공한다 / 웹만을 위한 HTTP 메시지 리더와 라이터 구현체와 함께 / 폼 데이터, 멀티파트 내용, 서버가 보낸 이벤트들 그리고 다른것들을 위해

`ClientCodecConfigurer`와 `ServerCodecConfigurer`는 전통적으로 사용된다 / 코덱들을 설정 및 커스터마이징 하기 위해 / 어플리케이션의 사용을 위한 /
HTTP 메시지 코덱을 설정하는 섹션을 보라 /

### Jackson JSON

JSON과 binary JSON (Smile)은 둘다 지원된다 / Jackson 라이브러리가 나타날 때 /

`Jackson2Decoder`는 다음과 같이 동작한다:

- Jackson의 비동기, 비 차단 파서는 사용된다 / 스트림 바이트 덩어리를 모으기 위해 / JSON 객체로 각각 표현된 `KokenBuffer` 안으로 /
- 각 `TokenBuffer`는 Jackson의 `ObjectMapper`에게 전달된다 / 고수준 객체를 생성하기 위해 /
- 단일값 퍼블리셔 (예: `Mono`)로 해석할 때 / 하나의 `TokenBuffer`가 있다. /
- 여러값 퍼블리셔 (예: `Flux`)로 해석할 때 / 각 `TokenBuffer`는 `ObjectMapper`로 전달된다 / 가능한한 충분한 바이트를 수신하여 / 완전히 형성된 객체를 위해 / 입력된 내용은 JSON 배열이 될 수 있다 / 또는 라인으로 구분된 JSON으로 될 수 있다 / 만약 컨텐트 타입이 "application/stream+json" 이면 /

`Jackson2Encoder`는 다음과 같이 동작한다:

- 단일값 퍼블리셔 (예: `Mono`)를 위해, / `ObjectMapper`를 통해서 그것을 단순히 직렬화 한다. /
- "application/json"과 함께하는 여러값 퍼블리셔를 위해 / `application/stream+json` 또는 `application/stream+x-jackson-smile`과 같은 / 각 값을 개별정으로 인코딩, 작성 또는 플러시 한다. / 인라인 구분 JSON 형식을 사용해서 /
- SSE를 위해서 / `Jackson2Encoder`는 각 이벤트를 호출한다 / 그리고 출력은 플러시된다 / 지연 없는 전달을 확실히 하기 위해 /

> 기본적으로 / `Jackson2Encoder`와 `Jackson2Decoder`는 `String` 유형의 엘리먼트를 지원하지 않는다 /
> 대신에 기본 가정은 / string 또는 string 시퀀스는 직렬화된 JSON 내용을 표현한다, / `CharSequenceEncoder`에 의해 랜더링 되기 위해 / 만약 당신이 `Flux<String>`으로 부터 JSON 배열을 랜더링하기 위한 무엇을 원한다면, / `Flux#collectToList()`를 사용하고 `Mono<List<String>>`으로 인코딩해라

### Form Data

`FormHttpMessageReader`와 `FormHttpMessageWriter`는 "application/x-www-form-urlencoded" 내용을 인코딩 및 디코딩하는 것을 지원한다. /

서버쪽에서 / 어디서 내용이 자주 필요로 한다 / 여러 장소로 부터 접속되기 위해, / `ServerWebExchange`는 전용 `getFormData()` 함수를 제공한다. / `FormHttpMessageReader`를 통해 내용을 분석하는 / 그리고 반복적인 접속을 위한 결과를 캐시한다. /
`WebHandler` API 섹션안의 Form Data를 봐라 /

한번 `getFormData()`가 사용되면, / 원래의 가공되지 않은 내용은 request body로부터 더이상 읽혀지지 않는다. /
이러한 이유로, / 어플리케이션은 `ServerWebExchange`를 지속적으로 겪을 것으로 예상된다 / 캐시된 폼 데이테로의 접근을 위해 / 가공되지 않는 requst body로 부터 읽는 것을 대비해서 /

### Multipart

`MultipartHttpMessageReader`와 `MultipartHttpMessageWriter`는 "multipart/form-data" 내용을 디코딩 및 인코딩 하는것을 지원한다.
결국 `MultipartHttpMessageReader`는 다른 `HttpMessageReader`에게 위임한다 / `Flux<Part>`를 실제로 분석하기 위해 / 그리고 일부분들을 `MultivalueMap`으로 단순히 수집한다. /
현재 시점에 Synchronoss NIO Multipart가 실제적인 분석을 위해 사용된다. /

서버쪽에서 / 어디서 멀티파트 폼 내용을 필요로 한다. / 다양한 장소에서 접속되기 위해, / `ServerWebExchange`는 전용의 `getMultipartData()` 함수를 제공한다. / `MultipartHttpMessageReader`를 통해 내용을 분석하는 / 그리고 반복적인 접속을 위해 결과를 캐시한다. /
`WebHandler` API 섹션의 Multipart Data를 봐라 /

한번 `getMultipartData()`가 사용되면, / 원래의 가공되지 않은 내용은 request body로부터 더이상 읽혀지지 않는다. /
이러한 이유로 / 어플리케이션은 `getMultipartData()`를 지속적으로 사용해야 한다 / 일부로의 반복되고, 맵과 같은 접근을 위해 / 또는 그렇지 않으면 `SynchronossPartHttpMessageReader`에 의존한다 / `Flux<Part>`로 일시적인 접근을 위해 /

### Limits

`Decoder`와 `HttpMessageReader` 구현 / 입력 흐름의 일부 또는 전체를 완화하는 / 제한으로 구성될 수 있다 / 바이트의 최대 숫자에서 / 메모리 상에서 완충하기 위해. /
일부 유형에서 버퍼링은 나타난다 / 왜냐하면 입력은 단일 객체로 모여지고 나타내어 지기 때문이다 / 예를 들어 / 컨트롤러 함수 / `RequestBody byte[]`, `x-www-form-urlencoded` 데이터 등등과 함께 /
버퍼링은 또한 스트리밍과 함께 나타난다 / 입력 흐름을 나눌 때 / 예를 들어, / 별개의 문자, JSON 객체의 흐름 등등. /
그것들의 스트리밍 유형들을 위해, / 제한은 바이트의 숫자에 적용한다 / 하나의 객체와 어울려서 / 흐름 내에서 /

완충제의 크기를 설정하기 위해, / 당신은 확인할 수 있다 / 만약 주어진 `Decoder` 또는 `HttpMessageReader`가 `maxInMemorySize` 특성에 노출되면 / 그리고 Javadoc 기본 값의 상세를 가지게 될 것이다. /
WebFlux에서, `ServerCodecConfigurer`는 단일 장소를 제공한다 / 모든 코덱들을 설정할 수 있는 장소로 부터, / 기본 코덱들을 위한 `maxInMemorySize` 특성을 통해서. /
클라이언트 쪽에서, / 제한은 WebClient.Builder로 바뀐다. /

Mulipart 분석을 위해 / `maxInMemorySize` 특성은 비 파일 부분의 크기를 제한한다. /
파일 부분을 위해 / 그것은 한계점을 결정한다 / 부분이 디스크에 기록된 곳에 /
디스크에 기록된 파일 부분을 위해 / 추가적인 `maxDiskUsagePerPart` 특성이 있다 / 부분 마다 디스크 공간의 양을 제한하기 위해 /
또한 `maxParts` 특성이 존재한다 / 부분의 전반적인 숫자를 제한하기 위해 / 멀티파트 요청에서 /
3가지 모두를 설정하기 위해 / WebFlux에서 / 당신은 사전에 설정된 `MultipartHttpMessageReader`의 인스턴스를 `ServerCodecConfigurer`에 공금할 필요가 있을 것이다. /

### Streaming

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-async-http-streaming)

HTTP 응답 (예를 들어, `text/event-stream`, `application/stream+json`)으로 스트리밍 할때, 연결 되지 않은 클라이언트를 보다 빨리 안정적으로 감지하기 위해서는 데이터를 주기적으로 전송하는 것이 중요하다. 그러한 전송은 코멘트 전용, 빈 SSE 이벤트 또는 실질적으로 하트비트 역할을 할 수 있는 기타 "no-op" 데이터일 수 있다.

### `DataBuffer`

`DataBuffer`는 WebFlux에서 바이트 버퍼를 위한 표현이다. 참고자료의 Spring Core 부분은 [Data Buffers 와 Codecs](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#databuffers)에 관한 섹션에 더 많은 내용을 담고 있다. 요점은 Netty와 같은 일부 서버에서 바이트 버퍼들은 풀링되고 참조카운트 되며, 메모리 누수를 피하기 위해 소비하였을 때 반드시 풀려야 한다는 것을 이해하는 것이다.

WebFlux 어플리케이션은 일반적으로 데이터 버퍼를 직접 소비 및 생산하지 않는 한, 고수준 객체를 변환하거나 사용자 지정 코덱을 생성하지 않는 한 이러한 문제에 대해 걱정할 필요가 없다. 이러한 경우에는 [Data Buffers 및 Codecs](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#databuffers) 정보, 특히 [Using DataBuffer](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#databuffers-using) 섹션을 검토하라.

## 1.2.6. Logging

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-logging)

Spring WebFlux에서 DEBUG 수준의 로깅은 작고, 최소한이고, 인간 친화적으로 설계되어 있다. 특정 이슈를 디버깅할 때만 유용한 다른 정보와 비교해서 계속 유용한 고부가가치 비트에 초점을 맞춘다.

TRACE 수준 로깅은 일반적으로 DEBUG와 동일한 원칙을 따라가지만 (그리고 예를 들어 불쏘시개가 되어서는 안된다.) 모든 이슈를 디버깅하기 위해 사용될 수 있다. 또한 일부 로그 메시지는 TRACE와 DEBUG에서 다른 수준의 상세를 보여준다.

좋은 로깅은 로그의 사용 경험에서 비롯된다. 만약 명시된 목표에 부합하지 않는 것이 발견되면 우리에게 알려달라.

### Log Id

WebFlux에서 단일 요청은 여러개의 쓰레드로 실행할 수 있으며 쓰레드 ID는 특정 요청에 속하는 상관관계가 있는 로그 메시지를 위해서는 유용하지 않다. WebFlux 로그 메시지가 기본적으로 요청별 ID를 접두사로 가지고 있는 이유가 여기에 있다.

서버측에서 로그 ID는 해당 ID를 기반으로 완전 포멧된 접두사가 `ServerWebExchange#getLogPrefix()`에서 사용가능한 동안 `ServerWebExchange` 속성(`LOG_ID_ATTRIBUTE`)에 저장된다. `WebClient`쪽에서는 로그 ID는 완전히 정제된 접두사가 `ClientRequest#logPrefix()`에서 사용가능하는 동안 `ClientRequest` 속성 (`LOG_ID_ATTRIZBUTE`)에 저장된다.

### Sensitive Data

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-logging-sensitive-data)

`DEBUG`와 `TRACE`로깅은 민감정보를 로깅할 수 있다. 이 때문에 매개변수와 헤더는 기본적으로 가려지며, 전체 로그는 `DispatcherServlet`의 `enableLoggingRequestDetails` 속성을 통해 명시적으로 활성화 되어야 한다.

다음 예제는 Kotlin 설정을 통해 이를 수행하는 방법을 보여준다:

```kotlin
class MyInitializer : AbstractAnnotationConfigDispatcherServletInitializer() {

    override fun getRootConfigClasses(): Array<Class<*>>? {
        return ...
    }

    override fun getServletConfigClasses(): Array<Class<*>>? {
        return ...
    }

    override fun getServletMappings(): Array<String> {
        return ...
    }

    override fun customizeRegistration(registration: ServletRegistration.Dynamic) {
        registration.setInitParameter("enableLoggingRequestDetails", "true")
    }
}
```

### Custom codecs

어플리케이션은 사용자 정의 코덱들을 추가적인 미디어 타입의 지원, 또는 기본 코덱이 지원하지 않는 특별한 행동을 위해 등록할 수 있다.

개발자에 의해 표현되는 일부 설정 옵션들은 기본 코덱에서 시행된다. 사용자 정의 코덱은 [버퍼링 한계 적용](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-codecs-limits) 또는 [민감한 정보 로깅](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-logging-sensitive-data)과 같은 선호를 맞추기 위한 기회를 얻는 것을 원할 수 있다.

다음 예제는 클라이언트 측의 요청에 대해 수행 방법을 보여준다:

```kotlin
val webClient = WebClient.builder()
        .codecs({ configurer ->
                val decoder = CustomDecoder()
                configurer.customCodecs().registerWithDefaultConfig(decoder)
         })
        .build()
```

## 1.3. `DispatcherHandler`

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-servlet)

Spring MVC와 유사하게 Spring WebFlux는 중앙 `WebHandler`, `DispatcherHandler`가 실제 작업을 설정가능하고 대리 구성요소에 의해 수행되는 동안 요청 처리를 위해 공유된 알고리즘을 제공하는 프론트 컨트롤러 패턴 중심으로 설계되었다. 이 모델은 유연하고 다양한 업무흐름을 지원한다.

`DispatcherHandler`는 스프링 설정에서 필요로 하는 위임된 구성요소를 발견한다. 또한 Spring 빈 그 자체로 설계되었으며 실행 컨텍스트에 접근하기 위해 `ApplicationContextAware`를 구현한다. `DispatcherHandler`가 `webHandler`라는 이름의 빈으로 선언되면, [`WebHandler` API](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-web-handler-api)에 기술된 바와 같이 요청 처리 체인을 결합한 `WebHttpHandlerBuilder`에 의해 발견된다.

WebFlux 어플리케이션에 일반적으로 구성된 스프링 설정:

- `webHandler`라는 이름을 가진 `DispatcherHandler`
- `WebFilter` 및 `WebExceptionHandler` 빈
- [`DispatcherHandler` 특별 빈](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-special-bean-types)
- 기타

설정은 처리 체인을 구축하기 위해 `WebHttpHandlerBuilder`로 제공되며, 다음과 같이 보여준다:

```kotlin
val context: ApplicationContext = ...
val handler = WebHttpHandlerBuilder.applicationContext(context).build()
```

`HttpHandler`의 결과는 [server adapter](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-httphandler)와 함께 사용할 준비가 된다.

## 1.3.1 Special Bean Types

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-servlet-special-bean-types)

`DispatcherHandler`는 요청 처리 및 적절한 응답을 위해 특별한 빈들을 위임한다. "특별한 빈"이란 WebFlux 프레임워크 계약을 이행하는 스프링이 관리하는 `Object` 인스턴스를 의미한다. 그것들은 주로 내장된 계약을 수반하지만 특성들을 개인화, 확장, 또는 교체할 수 있다.

다음 테이블은 `DispatcherHandler`에 의해 감지된 특별한 빈들의 목록을 보여준다. 다음 표에는 `DispatcherServlet`에서 찾은 특별한 빈들이 나열되어 있다.

| Bean Type                                                                                                                                                                                                                                                                    | Explanation                                                                                                                                                                                                                                                                                                                                                                                                                           |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `HandlerMapping`                                                                                                                                                                                                                                                             | 요청을 핸들러에 연결한다. 연결하는 것은 일부 기준, `HandlerMapping` 구현체의 다양한 상세, 주석이 달린 컨트롤러, 단순한 URL 패턴 매핑 등등을 기반으로한다. <br/><br/> 주요 `HandlerMapping` 구현체는 `@requestMapping` 주석이 달린 함수를 위한 `RequestMappingHandlerMapping`, 함수형 끝점 라우트들을 위한 `RouterFunctionMapping`, 그리고 URI 경로 패턴들과 `WebHandler` 인스턴스의 명시적 등록을 위한 `SimpleUrlHandlerMapping`이다. |
| `HandlerAdapter`                                                                                                                                                                                                                                                             |
| 어떻게 핸들러가 실제로 호출되는지에 상관없이 요청에 연결된 핸들러를 호출하기 위해 `DispatcherHandler`를 지원한다. 예를 들어, 주석이 달린 컨트롤러를 호출하려면 주석을 해결해야 한다. `HandlerAdapter`의 주요 목적은 `DispatcherHandler`를 이러한 상세로부터 보호하는 것이다. |
| `HandlerResultHandler`                                                                                                                                                                                                                                                       | 핸들러 호출의 결과 처리 및 응답을 완료한다. [Result Handling](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-resulthandling)을 봐라                                                                                                                                                                                                                                                  |

## 1.3.2. WebFlux Config

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-servlet-config)

어플리케이션은 요청 처리를 위한 기반 시설 용 빈 ([Web Handler API](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-web-handler-api-special-beans) 및 `DispatcherHandler`에 열거된) 을 선언할 수 있다. 하지만 대부분의 경우, [WebFlux Config](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-config)는 최고의 시작 점이다. 그것은 필수 빈을 선언하고 개인화를 위해 고수준 설정 콜백 API를 제공한다.

> Spring Boot는 Spring WebFlux를 설정하기 위해 WebFlux 설정에 의존하고 또한 많은 추가적인 편의 옵션들을 제공한다.

## 1.3.3. Processing

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-servlet-sequence)

`DispatcherHandler`는 요청을 다음과 같이 처리한다:

- 각 `HandlerMapping`는 일치하는 핸들러를 찾으라고 요청되며, 처음 매칭된 것이 사용된다.
- 핸들러를 찾지 못했다면, 적절한 `HandlerAdapter`가 실행되며, 이 어뎁터는 `HandlerRequlst`와 같은 실행으로 부터 반환 값을 노출한다.
- `HnadlerResult`는 응답을 기록하거나 랜더링할 뷰의 사용을 처리를 완료하기 위해 적절한 `HandlerResultHandler`에게 주어진다.

## 1.3.4. Result Handling

핸들러의 호출로 부터의 반환 값은 `HandlerAdapter`를 통해 일부 추가적인 컨텍스트와 함께 `HandlerResult`로 포장되며, 그에 대한 지원을 요구하는 최초의 `HandlerResultHandler`에게 전달된다. 다음 테이블은 가능한 `HandlerResultHandler` 구현체들을 보여준다, 모든 것은 [WebFlux Config](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-config)에 선언되어 있다.

| Result Handler 유형           | 반환 값                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | 기본 순서           |
| :---------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------ |
| `ResponseEntityResultHandler` | `ResponseEntity`, 일반적으로 `@Controller` 인스턴스에서 제공                                                                                                                                                                                                                                                                                                                                                                                                                                            | 0                   |
| `ServerResponseResultHandler` | `ServerReponse`, 일반적으로 함수형 끝점에서 제공                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 0                   |
| `ResponseBodyResultHandler`   | `@responseBody` 함수 또는 `@ResponseController` 클래스에서 제공하는 반환 값을 다룬다.                                                                                                                                                                                                                                                                                                                                                                                                                   | 100                 |
| `ViewResolutionResultHandler` | `CharSequence`, `View`, [Model](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/ui/Model.html), `Map`, [Rendering](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/web/reactive/result/view/Rendering.html), 또는 다른 `Object`는 모델 속성으로 취급된다. <br/><br/> [View Resolution](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-viewresolution)을 참고하라 | `Integer.MAV_VALUE` |

## 1.3.5. Exceptions

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-exceptionhandlers)

`HandlerAdapter`로 부터 반환된 `HandlerResult`는 일부 핸들러 별 매커니즘을 기반으로 오류를 다루는 기능을 노출할 수 있다. 이런 에러 기능은 호출되어 진다 :

- 핸들러 (예를 들어, `@Controller`) 호출이 실패한다면.
- `HandlerResultHandler`를 통한 핸들러 반환 값을 다루는 것이 실패한다면.

핸들로에서 반환된 반응형 타입 핸들러가 데이터 항목을 생산하기 전에 오류 신호가 발생하는 한, 오류 함수는 응답(예를 들어, 오류 상태)를 변경할 수 있다.

`@Controller` 클래스의 `@ExceptionHandler` 함수는 이렇게 지원된다. 대조적으로 Spring MVC에서 동일한 지원은 `HandlerExceptionResolver`에 구축되어 있다. 이것은 일반적으로 문제가 되지 않는다. 하지만, WebFlux에서는 핸들러가 선택되기 이전에 발생하는 예외를 다루기 위한 `@ControllerAdvice`는 사용할 수 없다는 것을 명심해야 한다.

"Annotaged Controller" 섹션의 [Managing Exceptions](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-controller-exceptions) 또는 WebHandler API 섹션의 [Exceptions](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-exception-handler)를 봐라

## 1.3.6. View Resolution

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-viewresolver)

뷰 리솔루션은 특별한 뷰 기술 없이도 HTML 템플릿과 모델로 브라우저로 랜더링 할 수 있게 해준다. Spring WebFlux에서 뷰 리솔루션은 문자열(논리적 뷰의 이름을 나타내는)을 `View` 인스턴스로 매핑하기위해 `ViewResolver` 인스턴스를 사용하는 전용 [HandlerResultHandler](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-resulthandling)를 통해 지원된다. `View`는 그런다음 응답을 랜더링 하는데 사용된다.

### Handling

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-handling)

`ViewResolutionResultHandler`에 전달된 `HandlerResult`는 요청을 다루는 동안 추가된 속성을 담고 있는 핸들러와 모델의 반환 값을 담고있다. 반환 값은 다음 중 하나로 처리된다:

- `String`, `CharSequence`: 구현된 `ViewResolver` 구현체 목록을 통해 `View`로 해석될 논리적 뷰 이름.
- `void`: 요청 경로를 기반으로 앞과 뒤의 슬래시를 제거하고 기본 뷰 이름을 선택한 다음 `View`로 해결한다. 뷰 이름이 제공되지 않거나 (예를 들어 모델 속성이 반환되거나) 또는 값이 비동기적으로 반환(예를 들어 `Mono`가 빈값으로 완료될때) 될 때 동일한 현상이 발생한다.
- [Rendering](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/web/reactive/result/view/Rendering.html): 뷰 리솔루션 시나리오를 위한 API. IDE의 코드완성 옵션을 살펴보자.
- `Model`, `Map`: 요청에 대해 모델에 추가할 추가 모델 특성
- 기타 사항: 다른 모든 반환 값([BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)에 의해 결정된 단순 유형을 제외)은 모델에 추가할 모델 속성으로 처리된다. 속성 이름은 핸들러 함수 `@ModelAttribute` 주석이 없는 한 [규칙](https://docs.spring.io/spring-framework/docs/5.2.5.RELEASE/javadoc-api/org/springframework/core/Conventions.html)을 사용한 클래스 이름에서 파생된다.

모델은 비동기성, 반응형 타입을 포함할 수 있다.(예를 들어 Reactor 또는 RxJava) 랜더링에 앞서, `AbstractView`는 이러한 모델 특성을 구체적인 값으로 결정하고 모델을 업데이트한다. 단일 값 반응형 타입은 단일 값 또는 값이 없이 (만약 비어있다면) 해결되고, 다중 값 반응형 타입은 (예를 들어 `Flux<T>`)는 `List<T>`로 수집되고 해결된다.

뷰 리솔루션을 설정하는 것은 `ViewResolutionResultHandler` 빈을 Spring 설정에 추가하는 것 만큰 간단하다. [WebFlux Config](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-config-view-resolvers)는 뷰 리솔루션을 위한 전용 설정 API를 제공한다.

Spring WebFlux와 통합된 뷰 기술에 대한 자세한 내용은 [View Technologies](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-view)를 참조하라.

### Redirecting

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-redirecting-redirect-prefix)

뷰 이름에서 특수 `redirect:` 접두사를 사용하면 리다이렉션을 수행할 수 있다. `UrlBasedViewResolver` (그리고 하위 클래스들) 이 리다이렉션이 필요되는 구문으로 인식한다. 뷰 이름의 나머지 부분은 리다이렉션 URL이다.

인터넷 효과는 컨트롤러가 `RedirectView` 또는 `Rendering.redirectTo("abc").build()`를 반환해 온 경우와 같다, 하지만 지금은 컨트롤러가 스스로 논리적인 뷰 이름 관점에서 수행할 수 있다. `redirect:/some/resource`와 같은 뷰 이름은 현재 어플리케이션과 상대적인 반면 `redirect:https://example.com/arbitrary/path`와 같은 뷰 이름은 절대 URL 경로로 리다이렉트 한다.

### Content Negotiation

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-multiple-representations)

`ViewResolutionResultHandler`는 컨텐츠 협상을 지원한다. 그것은 요청 미디어 타입과 각 선택된 `View`에서 지원하는 미디어 타입을 비교한다. 요청된 미디어 유형이 지원하는 첫번째 `View`가 사용된다.

JSON 및 XML과 같은 미디어 타입을 지원하기 위해 Srping WebFlux는 [HttpMessageWriter](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-codecs)를 통해 랜더링 되는 특별한 `View`인 `HttpMessageWriterView`를 제공한다. 일반적으로, [WebFlux Configuration](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-config-view-resolvers)을 통해 기본 뷰로 이것들을 설정할 수 있다. 요청된 미디어 유형이 일치한다면 기본 뷰는 항상 선택되고 사용된다.

## 1.4. Annotated Controllers

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-controller)

Spring WebFlux는 `@Controller` 및 `@RestController` 컴포넌트들은 어노테이션으로 사용하여 요청 매핑, 요청 입력, 예외 처리 등등을 표현하기 위한 어노테이션 기반 프로그래밍 모델을 제공한다. 어노테이션 기반 컨트롤러들은 유연한 함수 시그니처를 가지고 있고 기본 클래스를 확장하거나 특별한 인터페이스를 구현할 필요가 없다.

다음 목록은 기본적인 예를 보여준다:

```kotlin
@RestController
class HelloController {

    @GetMapping("/hello")
    fun handle() = "Hello WebFlux"

}
```

앞의 예에서 함수는 응답 본문에 쓸 `String`을 반환한다.

1.4.1. `@Controller`

[Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-controller)

표준 스프링 빈 정의를 사용하여 컨트롤러 빈을 정의할 수 있다. `@Controller` 스테레오타입은 자동 탐지를 허용하고 클래스경로에서 `@Component` 클래스들을 감지하는 것과 그것들을 위한 빈 정의를 자동으로 등록하기 위한 스프링 일반 지원과 함께 정렬된다. 또한 주석이 달린 클래스를 위한 스테레오타입으로 작용하여 웹 펌포넌트와 같은 역할을 나타낸다.

이러한 `@Controller`빈의 자동 탐지를 활성화 하려면 다음 예체와 같이 자바 설정에 컴포넌트 스캐닝을 추가할 수 있다:

```kotlin
@Configuration
@ComponentScan("org.example.web") // (1)
class WebConfig {

    // ...
}
```

(1) `org.example.web` 패키지를 스캔한다.

`@RestController`는 `@Controller`와 `@responseBody` 메타 어노테이션이 달린 [구성된 어노테이션](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-meta-annotations)이고, 모든 함수가 유형 수준 `@ResponseBody` 어노테이션을 상속한 컨트롤러를 나타내며, 따라서 뷰 리솔루션 및 HTML 템플릿을 랜더링하는 것 대신 응답 본문을 직접 작성한다.
