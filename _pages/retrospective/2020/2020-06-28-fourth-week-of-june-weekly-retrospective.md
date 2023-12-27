---
layout: single
title: "2020년 6월 4주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-fourth-week-of-june-weekly-retrospective/
---

새로운 회사에 출근한지 2주가 지났다. 첫 주에 비해 마음도 한결 편안해 지고 같은 부서의 개발자들과는 다른분들에 비해 가까워 진것 같다. 이번주에는 리더의 요청으로 지난주부터 작업했던 대사 작업에 대한 리뷰를 동료 백엔드 개발자들 앞에서 리뷰하는 시간을 가졌었다. 주로 어떻게 설계했는지와 테스트 코드를 어떻게 작성하였는지 공유하는 내용으로 구성하여 발표를 진행하였었다. 발표기간 동안 동료 개발자분들이 재미있게 들어주셨고 토의할만한 주제로 많은 질문을 주셔서 정말 기분 좋은 시간이었고 좋은 경험이었다. 개발 구성원들이 새로운 구성원과 기술에 대한 태도와 열의는 내가 본받아야 할 자세인 것 같다.

아래는 내가 이번주 동안 새롭게 배우거나 사용했던 기술들에 대한 정리이다.

## Spring Boot의 Configuration 설정 시 주의사항

프로젝트 코드에서 간혹가다 보면 `@SpringBootApplication`이 선언되어 있는 Application 클래스에 `@EnableJpaAuditing`과 같이 추가 라이브러리를 사용하기 위한 설정 어노테이션을 함께 적용해 두는 경우들을 볼 수 있다. 이번주에 작업하면서 발생했던 이슈중에 하나가 Controller Unit Testing을 위해 `@WebMvcTest`를 선언해서 테스트 코드를 작성하려고 하였는데 JPA와 관련한 오류가 계속 발생하는 이슈를 겪었었다. JPA와 관련된 빈을 테스트 빈에 추가하지도 않았는데 관련 오류가 발생하길래 이유를 찾아보니 `@SpringBootApplication`과 함께 `@EnableJpaAuditing`이 함께 선언되어 있는 것을 볼 수 있었다. 해당 문제는 Application 클래스에서 별도로 `@Configuration`을 선언한 설정 클래스를 분리하고 `@EnableJpaAditing`을 선언해 주니 오류가 발생하는 테스팅을 해결할 수 있었다. 문제가 발생한 이유를 명확하게 파악하지는 못했지만 아마도 Unit Testing시 `@SpringBootApplication`을 필수적으로 실행시켜야 하는데 그 클래스에 추가적인 라이브러리를 사용하기 위한 어노테이션을 함께 사용하고 있어서 불필요한 설정이 함께 적용되면서 발생했던 문제였던 것으로 예상된다. 앞으로 SpringBoot 설정 시 `@SpringBootApplication`이 선언된 Application 클래스에 설정관련 코드를 넣지 말고 별도의 `@Configuration`이 선언된 설정 클래스를 새롭게 추가해서 사용하는 습관을 가져야 겠다.

## RestTemplate Mocking

Unit Testing을 위해 RestTemplate를 `Mockito.when`을 사용하는 경우 파라미터 중에 equals를 지원하지 않는 파라미터가 있을 수 있다. (예를 들어 `HttpEntity`객체) 그래서 테스트 시 모의 객체의 응답값에서 `NullPointerExcepion`이 발생하지 않으려면 `ArgumentMatchers` 객체를 이용해서 파라미터를 잘 전달해 주어야 한다. 아래는 `ArgumentMatchers`를 사용하여 사용한 예제이다.

```java
Mockito.when(nicePayRestTemplate.exchange(ArgumentMatchers.anyString(), ArgumentMatchers.any(), ArgumentMatchers.any(), ArgumentMatchers.eq(Object.class)))
```

그리고 동일한 모의 객체를 여러번 호출하고 데이터를 반환해야 하는 경우 `thenReturn`을 여러번 사용하면 된다. 아래는 그 예제 코드이다.

```java
Mockito.when(nicePayRestTemplate.exchange(ArgumentMatchers.anyString(), ArgumentMatchers.any(), ArgumentMatchers.any(), ArgumentMatchers.eq(Object.class)))
    .thenReturn(new Object())
    .thenReturn(new Obejct());
```

## ZonedDateTime

사내에서 글로벌 시간 이슈를 대응하기 위해 `ZonedDateTime`을 사용하기로 하였다. 그동안 `LocalDateTime`으로 사용해 왔었는데, `LocalDateTime`객체는 JSON Serialize 시 타임존을 기본적으로는 표기하지 않아 클라이언트에서 시간을 표시할 때 문제가 발생할 가능성이 높다. 그래서 사내에서 `ZonedDateTime`을 기본적으로 사용하기로 결정하였고 자바로 개발한 서버들은 점진적으로 `ZonedDateTime`클래스로 변경해나가고 있다. 앞으로 `ZonedDateTime`클래스를 사용하면서 겪은 문제나 구현내용들에 대해 다루는 글을 적어보는 것도 좋을 것 같다.

## 협력 다이어그램 (Collaboration Diagram)

[오브젝트](http://www.yes24.com/Product/Goods/74219491){: target="\_blank" }책을 읽으면서 처음으로 [협력 다이어그램](https://www.visual-paradigm.com/guide/uml-unified-modeling-language/what-is-uml-collaboration-diagram/){: target="\_blank" }에 대해서 알게 되었다. 기능을 처음 구현할 때 클래스 다이어그램보다 간단하게 다이어그램을 정의하면서 요구사항에 대한 어떤 인터페이스를 정의하면 좋은지와 어떻게 상호작용하면 좋은지 작성해 보기에 좋을 것 같아서 앞으로도 자주 사용하게 될 다이어그램일 것 같다.

## 루시드 차트 (Lucidchart)

개발팀에서 다이어그램을 작성하기 위한 툴로 [루시드 차트](https://app.lucidchart.com/){: target="\_blank" }를 사용하기로 하였다. 이번 주 발표 때문에 협력 다이어그램과 클래스 다이어그램을 그려보았는데, [PlantUML](https://plantuml.com/){: target="\_blank" } 보다 훨씬 그리기 쉽고 다양한 기능을 제공해 주어서 앞으로 유용하게 사용할 것 같다.

## ReentrantLock

분산환경에서 함수 도는 객체에 대한 동기화를 위해서 `syncronized` 키워드나 `Atomic` 객체를 사용하면 된다는 것은 알고 있었으나 이번에 `ReentrantLock`이라는 객체를 사용하는 방법에 대해서 처음 알게 되었다. `syncronized`와 유사하게 사용할 수 있는데 차이점은 lock과 unlock을 명시적으로 다룰 수 있다는 장점이 있다.

<br/>

다음주에는 내가 운영 관련된 이슈를 담당하는 담당자로 지정되어 한주간 운영관련 요구사항에 대한 대응을 해야 한다. 아직 시스템에 대한 이해도가 부족해서 잘할지 걱정이긴 하지만 최대한 동료들이 불편하지 않도록 잘 해보아야 겠다. Typescript + GraphQL + MongoDB로 되어 있는 서버에 대한 개발 일정도 함께 있다. 세가지 기술 전부 사용경험이 없지만 새로운 기술을 사용해보고 경험을 가질 수 있다는 점에서 설레리도 한다. 차근차근 적응해 가서 바쁜 동료의 업무를 분산할 수 있었으면 좋겠다.
