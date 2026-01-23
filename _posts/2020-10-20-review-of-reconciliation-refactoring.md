---
layout: single
title: "The review of reconciliation refactoring"
categories:
  - EXPLANATION
tags:
  - DDD
  - kotlin
  - Event Driven
  - JPA

toc: true
toc_sticky: true
excerpt: 이 글은 내가 회사에서 대사 기능을 리펙토링하면서 배운것들을 동료들에게 공유하기 위해 정리했던 내용을 옮긴 글이다. 피쳐를 쳐내기 바쁘다는 핑계로 대대적인 리펙토링을 할 기회가 많지 않았는데 이번 리펙토링을 하면서 크게 스프링의 빈을 정의하고 어떻게 주입하는 지에 대해 좀더 배울 수 있게 되었고 특히 TDD를 할때 어떻게 테스트 코드를 작성해야 되는지를 많이 배울 수 있었다. 그럼 본 내용을 시작해 보자.
---

## Intro

이 글은 내가 회사에서 대사 기능을 리펙토링하면서 배운것들을 동료들에게 공유하기 위해 정리했던 내용을 옮긴 글이다. 피쳐를 쳐내기 바쁘다는 핑계로 대대적인 리펙토링을 할 기회가 많지 않았는데 이번 리펙토링을 하면서 크게 스프링의 빈을 정의하고 어떻게 주입하는 지에 대해 좀더 배울 수 있게 되었고 특히 TDD를 할때 어떻게 테스트 코드를 작성해야 되는지를 많이 배울 수 있었다. 그럼 본 내용을 시작해 보자.

## 왜 대사 기능을 리펙토링 하게 되었나?

네이버 페이 거래 비중이 높아 네이버 페이 대사 기능이 추가되어야 했다.

![slack-message](/assets/images/posts/review-of-reconciliation-refactoring/slack-message.png)

처음 나이스페이 대사 기능 개발 시 나름대로 확장성을 고려한답시고(?) 모듈별로 인터페이스를 두고 개발을 하였고, 간단히 네이버페이 수집기 구현체만 추가하면 될거라 생각했다.

![meme-1](/assets/images/posts/review-of-reconciliation-refactoring/meme-1.jpg)

하지만 우리의 Spring Boot는 특정 인터페이스에 여러개의 구현체가 존재하는 경우 별도의 설정이 없다면 하나의 구현체를 명시해줘야 정상적으로 빈을 주입할 수 있다. (여러개를 주입받으려면 Map이나 List 타입으로 받을 수 있다.) [스프링 문서 참고](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-definition){: target="\_blank" }

그래서 어노테이션 기반으로 빈을 생성하는 방식으로 한다면 네이버 페이 수집기가 추가 된다면 대사 처리기도 추가 되어야 하고 대사 처리기는 동일한 동작을 수행하므로 중복코드가 발생하는 문제가 생기게 되었다.

![meme-2](/assets/images/posts/review-of-reconciliation-refactoring/meme-2.jpg)

그래서 클래스를 많이 생성하지 않고 앞으로 PG사가 추가 되더라도 손쉽게 기능을 확장할 수 있도록 리펙토링을 해야할 필요가 있다고 생각했다.

## 어떻게 바뀌었나?

초기 대사 설계는 아래와 같다.

![init-arch](/assets/images/posts/review-of-reconciliation-refactoring/init-arch.png)

처음에는 네이버 페이가 존재하지 않았으니 대사 처리기와 조정기가 2개씩 존재했다. 그래서 대사처리기, 조정기가 일별, 월별로 존재하면서 중복되는 코드가 있어도 문제가 되지 않는다고 생각했다.

중복은 2번까지는 허용하되 3번째 부터는 리펙토링을 한다 라는 [Rule of three](https://en.wikipedia.org/wiki/Rule_of_three_(computer_programming)){: target="\_blank" }를 생각했다.

하지만 네이버 페이가 생기면서 중복되는 코드는 4개가 되면서 리펙토링이 불가피해 졌다.

![meme-3](/assets/images/posts/review-of-reconciliation-refactoring/meme-3.jpeg)

먼저 변경된 설계를 보자.

![edit-arch](/assets/images/posts/review-of-reconciliation-refactoring/edit-arch.png)

위 설계외 크게 바뀐점은 대사 처리기와 조정기의 구현체가 하나로 바뀌고 수집기가 늘어난 것을 볼 수 있다.

수집기는 PG사 또는 DB에 따라, 일별이냐 월별이냐에 따라 얼마든지 추가될 수 있다. 하지만 대사 처리기와 조정기는 수집된 결과의 형식만 똑같다면 얼마든지 재사용 가능하다. 그래서 위와 같이 변경하였다.

여기서 스프링의 의존주입 매커니즘을 이해하고 있다면 궁금한 부분이 있을 것이다. 

어떻게 Collector를 주입할 것인가 이다. 

어노테이션 기반으로 빈을 정의하는 방식을 이용한다면 Conciliator와 Coordinator는 Collector마다 구현체를 생성하여 Bean의 이름에 맞춰 의존성을 주입해야 할 것이다.

```java
@Service
@RequiredArgsConstructor
public class DailyNaverPayConciliator implements Conciliator {

    private final PaymentBalanceCollector dailyOrderRepositoryBalanceCollector;
    private final PaymentBalanceCollector dailyNaverPayPaymentBalanceCollector;
    private final Coordinator dailyNaverPayCoordinator;

    @Override
    public ReconciliationResult reconcile(ReconciliationParameter param) {
        final var origin = reconcileOrigin(param);
        return dailyCoordinator.coordinate(origin);
    }

	  // ...
}
```

이와 같은 방식은 위에서 설계한 방식에서 사용되는 방식처럼 해야 하는데 말했다시피 로직에서 중복이 발생한다.

그래서 도입한 방법이 Configuration 클래스를 이용하여 빈을 정의하는 방식을 사용하는 것이다.

```java
@Configuration
public class ReconciliationServiceBeanConfig {
    @Bean
    public Conciliator dailyOrderConciliator(
            PaymentBalanceCollector retryableNicePayPaymentBalanceCollector,
            RepositoryBalanceCollector orderRepositoryBalanceCollector,
            RepositoryBalanceCollector paymentRepositoryBalanceCollector
    ) {
        final var coordinator = new ReconciliationCoordinator(paymentRepositoryBalanceCollector);
        return new PaymentHistoryConciliator(
                orderRepositoryBalanceCollector,
                retryableNicePayPaymentBalanceCollector,
                coordinator
        );
    }

		// ...

    @Bean
    public Conciliator monthlyNaverPayOrderConciliator(
            PaymentBalanceCollector monthlyNaverPayPaymentBalanceCollector,
            RepositoryBalanceCollector naverPayOrderRepositoryBalanceCollector,
            RepositoryBalanceCollector naverPayPaymentRepositoryBalanceCollector
    ) {
        final var coordinator = new ReconciliationCoordinator(naverPayPaymentRepositoryBalanceCollector);
        return new PaymentHistoryConciliator(
                naverPayOrderRepositoryBalanceCollector,
                monthlyNaverPayPaymentBalanceCollector,
                coordinator
        );
    }

		// ...
}
```

위와 같이 정의하면 `PaymentHistoryConfiliator` 를 그대로 사용하면서 다른 수집기들을 주입할 수 있게 된다.

하나의 인터페이스에 다수의 구현체가 존재하는 경우 위의 방식 말고도 `List` 나 `Map`으로도 정의할 수 있지만 위와 같이 정의한 이유는 `List`나 `Map`으로 정의하는 경우 대사 처리기 또는 조정기의 로직 내부에 조건문이 추가되어야 한다. 이는 만약 PG사가 추가되는 경우에 로직을 수정해야 하기 때문에 유연하지 못하다고 판단했다.

Configuration을 통해 빈을 정의한다면 별도로 로직을 수정하지 않고도 주입되는 객체를 변경해 줌으로써 얼마든지 기능을 확장할 수 있다. 그렇기 때문에 위와 같이 Configuration을 통해 빈을 정의하였다.

## 리펙토링 하면서 추가로 배운것

예전에도 얘기 했었는데 나는 유닛 테스트를 선호한다. 내가 느끼는 유닛 테스트의 기준이 참 모호한데 지금까지 이해하고 있는 유닛 테스트의 유닛은 하나의 클래스라고 생각했고, 해당 클래스가 가지고 있는 Public한 함수들은 모두 테스트하여야 한다고 생각했다. 그래서 대사 기능을 개발하면서 추가 되었던 모든 클래스에 대해 테스트 코드를 작성하면서 기능을 구현하였다. 

지난 기간 동안 TDD로 개발하면서 이렇게 테스트 코드를 작성하는데 도움이 된다고 생각했고 실제로 큰 문제가 없어서 내가 잘해 나간다고 생각하고 있었다.

하지만 지금과 같이 대사 기능을 대대적으로 리펙토링을 진행하면서 내가 만든 테스트 코드가 디자인 변경에 큰 걸림돌이 될 수도 있다는 것을 알게 되었다.

실제로 리펙토링을 진행하면서 디자인 변경 및 내부 코드 변경으로 인해 테스트 코드가 어마어마하게 변경되었다.

![git-commit-log](/assets/images/posts/review-of-reconciliation-refactoring/git-commit-log.png)

사실 처음에는 이렇게 테스트 코드가 변경되어야 하는게 맞다고 생각했다. 왜냐하면 코드의 변경이 일어났는데 테스트 코드가 변경되지 않는 다는 것은 코드의 검증력이 떨어진다고 생각했었기 때문이다.

이러한 고민을 하고 있는 와중에 다행히 이전에 함께 일했던 TDD 장인 개발자분과 술자리를 가지게 되었고 그분께 조언을 구한결과 다음과 같은 조언을 들을 수 있었다.

> 구현의 디테일을 테스트 하지마라

즉 모듈 단위에서 외부로 노출되는 기능만 테스트 하지 세세한 부분을 테스트 하다보면 디자인을 유연하게 바꾸지 못할 뿐더러 테스트를 위한 테스트가 될 수도 있다는 것이다.

알고보니 이 구문은 내가 알고 있는 것이었는데, 바로 private 함수를 테스트 해야 되나? 의 질문에 대한 답변에 자주 인용되는 문구였다.

사실 라이브러리와 같은 모듈을 개발할 때에는 사용자에게 노출되는 public 함수를 테스트 하는 것이 맞다. 하지만 우리와 같이 API를 개발할 때에는 외부에 노출되는 public 함수는 Controller에서 정의된 개별 API가 된다. Service, Repository, DTO 등등은 결국 사용자에게 노출되는 클래스들이 아니므로 private로 보면 되는 것이다. 그래서 Controller, Service, Repository 등등을 각각 테스트 할 것이 아니라 API를 테스트 하고 각각의 케이스별로 분기하여 커버리지를 높이는 방식으로 하여야 사용자에게 제공되는 기능은 변경되지 않으면서 내부 디자인을 자유자제로 변경하는 유연한 코드가 되는 것이다.

사실 위와 같은 깨달음은 내가 이때까지 가져왔던 유닛 테스트에 대한 생각을 바꿔야 하기 때문에 큰 충격으로 다가왔고 지난 TDD의 경험이 헛된 경험이 된 것 같아 허탈한 기분도 들었다.

![meme-4](/assets/images/posts/review-of-reconciliation-refactoring/meme-4.jpeg)

하지만 반대로 새롭게 더 나은 테스트를 작성할 수 있는 배움이 있어서 아주 기분이 좋기도 하다.

혹시나 이제껏 내가 주변의 친구들에게 이전의 TDD 방식을 전파했다면 이 이야기를 계기로 함께 바꿔 갔으면 좋겠다 라는 생각이 든다. (물론 이미 잘하고 있는 친구들이 있어서 타코를 100개 날려 주고 싶다.)

![meme-5](/assets/images/posts/review-of-reconciliation-refactoring/meme-5.jpg)

## 마무리

리펙토링 얘기에서 살짝 TDD 예기로 간거 같은데, 아무튼 이번 대사 작업을 하면서 항상 어노테이션 기반으로 빈을 정의하던 것에서 설정 기반으로 빈을 정의해 보면서 이런저런 고민들을 많이 할 수 있었고 그 과정에서 많이 배운거 같다. 그리고 큰 리펙토링 작업을 하면서 내가 가진 TDD의 시각을 전환할 수 있는 계기가 되어서 나에게 있어 큰 깨달음을 얻을 수 있어 뜻 깊은 작업이었다고 생각된다. 아직 갈길이 멀다ㅠ

![meme-6](/assets/images/posts/review-of-reconciliation-refactoring/meme-6.jpg)
