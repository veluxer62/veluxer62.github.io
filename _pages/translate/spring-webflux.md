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
