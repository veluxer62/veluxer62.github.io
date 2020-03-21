---
permalink: /translate/spring-webflux/
title: Spring Webflux
---

이문서는 지원을 덮는다 / reactive-stack 웹 어플리케이션을 위해 / Reactive Stream API 위에 만들어진 / non-blocking 서버를 실행하기 위한 / Netty, Undertow, 그리고 Servlet 3.1+ 컨테이너와 같은 /
개별 챕터는 Spring WebFlux 프레임워크를 덥는다 / reactive `WebClient`, testing 지원, 그리고 reactive 라이브러리들 /
Servlet-stack 웹 어플리케이션을 위해 / Web on Servlet Stack을 봐라 /

# 1. Spring WebFlux

원조 웹 프레임워크는 / Spring Framework, Spring Web MVC를 포함한 / Servlet API와 Servlet 컨테이너들을 위한 목적으로 만들어졌다. /
reactive-stack 웹 프레임워크, Spring WebFlux는 추가됬다 / 버전 5.0 이후에 /
그것은 완전히 non-blocking이다 / Reactive Streams 백 프레셔를 지원하는 / 그리고 Netty, Undertow, and Servlet 3.1+ 컨네이너들과 같은 서버를 실행하는 /

둘다의 웹 프레임워크들은 소스모듈들(spring-webmvc와 spring-webflux)의 이름을 반영한다 그리고 나란히 함께 존재한다 스프링 프레임워크에서 /
각 모듈들은 선택적이다. /
어플리케이션들은 하나 또는 다른 하나로 사용될 수 있다 / 일부의 경우에는 / 둘다 / 예를 들어 / Spring MVC 컨트롤러들 / reactive WebClient와 함께

## 1.1. 개요

왜 Spring WebFlux가 만들어졌나?

한편의 대답은 필요이다 / non-blocking web stack을 위한 / 동시성을 다루기 위해 / 적은 수의 thread들과 스케일과 함께 / 적은 하드웨어 리소스와 함께 /
Servlet 3.1은 API 를 제공한다 / non-blocking I/O를 위한
하지만 / 그것을 사용하는 것은 서블릿의 rest를 떠나도록 이끈다 / 계약들은 동기성(`Filter`, `Servlet`) 또는 blocking(`getParameter`, `getPart`)이다 /
그것은 동기이다 / 새로운 공통 API를 위한 / 기초를 제공하기 위한 / 어떤 non-blocking에 걸쳐 / 실행시점에 /
그것은 중요하다 / 서버들(Netty와 같은) 때문에 / 잘 설립된 / 비동기, non-blocking 공간에서 /

다른 대답은 함수형 프로그래밍이다. / 많음 / 어노테이션의 추가와 같은 / Java 5에서 / 기회를 만들었다 (어노테이션이 적용된 REST 컨트롤러 또는 유닛 테스트와 같은), / 람다 표현식의 추가 / Java 8에서 / 기회를 만들었다 / 함수형 API들을 위한 / Java에서 /
그것은 하나의 유행이다 / non-blocking 어플리케이션과 continuation-style API들을 위한 (`CompletableFuture`와 ReactiveX에 의해 대중화된 것과 같은) / 그것은 비동기 로직의 선언된 구성을 허락한다 /
프로그래밍 모델 레벨에서, / Java 8은 Spring Webflux를 가능하게 하다 / 함수형 웹 끝점을 제공하도록 / 어노테이션이 적용된 컨트롤러들 옆에 /

## 1.1.1. "Reactive" 정의

우리는 "non-blocking"과 "함수형"에 대해 언급했다. / 하지만 reactive는 무슨 의미인가?

"reactive" 용어는 프로그래밍 모델을 나타낸다 / 반응하는 것을 기반으로 한 / 변경하기 위해 / 네트워크 컴포넌트들 / IO 이벤트들에 응답하는 / UI 컨트롤러들 / 마우스 이벤트에 반응하는 / 그리고 다른것들
그런의미에서 / non-blocking은 reactive 이다 / 왜냐하면 / block 되는 것 대신에 / 우리는 / 지금 / 알림을 응답하는 것의 방식에 있다 / 실행 완료 또는 데이터가 사용가능한 것과 같은

또 다른 중요한 메커니즘이 있다 / 우리는 / 스프링팀에서 / "reactive"와 어울린다 / 그리고 그것은 non-blocking 백프레셔이다. /
동기화에서 / 필수적인 코드 / blocking 호출은 백 프레셔의 자연적인 형식을 제공한다 / 요청자가 기다리도록 강제하는 /
non-blocking 코드에서 / 그것은 중요하게 된다 / 이벤트의 비율을 다루는것이 / 그래서 빠른 생산자는 그것의 목적지에서 어쩔 줄 모르게 만들지 않는다 /

Reactive Streams 는 작은 스펙이다 (또한 적응되어있다 / Java 9에) / 그것은 정의한다 상호작용을 / 비동기 컨포넌트 사이에서 / 백 프레셔와 함께 /
예를 들어 데이터 레파지토리 (Publisher로 행동하는) 데이터를 생산할 수 있다 / HTTP 서버가 (Subscriber로 행동하는) 응답을 작성할수 있다./
Reactive Stream들의 주요 목표는 구독자를 이끄는 것이다 / 다룰수 있도록 / 어떻게 빠르게 / 또는 어떻게 느리게 Publisher가 데이터를 생산하는지

> **일반적인 질문: 만약 Publisher가 속도를 줄일 수 없다면?**
> Reactive Stream들의 목적은 단지 매커니즘과 바운더리를 설립하는 것이다 / 만약 Publisher가 속도를 줄일 수 없다면, / 버퍼나, 드랍, 실패를 사용하는 것을 결정해야만 한다.

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
