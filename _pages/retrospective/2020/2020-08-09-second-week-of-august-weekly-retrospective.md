---
layout: single
title: "2020년 8월 2주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-second-week-of-august-weekly-retrospective/
---

이번주는 대사기능 추가 작업과 신규시스템 개편작업을 계속해서 진행했다. 매주하는 개발자 세미나에서 ClickUp과 Github 연동, Kotest와 Junit을 비교한 내용을 발표하기도 하였다. 신규 시스템에서 재고 관리 기능이 필요한데, 동시성 처리에 대한 고민 많이 하였다. 구현이 끝나고 나면 좋은 경험이 될거 같다. 또 가격과 관련한 정보를 다룰 때 BigDecimal 객체를 사용하기로 하였는데 이부분도 좋은 경험이 될거 같다.

## Test Data
대사기능을 구현하면서 Unit 테스트 코드를 많이 만들어 놓았는데, 이때 테스트를 위한 데이터 생성 코드이 중복을 줄이기 위해서 공통 테스트 생성 함수를 만들어서 사용하고 있다. 가독성을 위해 데이터 생성 시 암묵적으로 설정된 값들을 테스트 데이터 생성 클래스에 많이 넣어 두었는데, 사용하기엔 편리한데 과연 이렇게 작성하는게 다른 동료가 테스트 코드를 이해하는데 도움이 될지는 의문이다. 좀더 나은 방법이 있는지 고민해 봐야겠다.

## 비관적 잠금

개편하는 시스템에서 재고 관리 시 [낙관적 잠금](https://en.wikipedia.org/wiki/Optimistic_concurrency_control){: target="\_blank" }을 사용하기보다는 [비관적 잠금](https://en.wikipedia.org/wiki/Record_locking){: target="\_blank" }이 더 좋다고 판단해서 비관적 잠금을 사용하기로 결정하였다. 아래는 잠금 전략 결정 시 고민했던 내용들이다.

- 동시적 경합 발생 가능성이 높다. 상품 정보는 여러 시스템에서 사용할 가능성이 높고 재고 변경 요청은 여러시스템에서 동시에 발생할 가능성이 높으므로 경합발생 가능성이 높다고 판단했다. 경합 발생이 높은 경우에 낙관적 잠금을 사용하면 예외의 잦은 발생으로 사용자 경험이 좋지 못하기 때문에 비관적 잠금을 사용하는게 낫다는 판단을 했다.
- 성능저하는 단순함으로 해결한다. 재고 변경 요청에 대해서만 잠금을 하므로 Locking으로 인한 시스템 성능저하를 최소화 하였다. 다른 메타정보에 대한 변경은 다른 API를 제공하였다.
- timeout으로 Dead Lock이 발생하지 않도록 조치한다. 비관적 잠금은 Dead Lock 발생 가능성이 존재한다. 이는 Timeout으로 해결하였다.
- 재고 변경 요청은 `변경할 최종 값`이 아니라 `변경할 수`이다. Maria DB의 경우 기본적으로 Transaction 격리 수준을 `Repeatable Read`로 한다. 그래서 재고 변경 요청을 `변경할 최종 값`으로 하는 경우 원치 않는 재고 값으로 변경 될 수 있다. 예를 들어 `사용자1`이 2개의 재고를 1개로 변경하고자 하고 동시에 `사용자2`가 2개의 재고를 1개로 변경하고자 하는 경우 `변경할 최종 값`으로 수정 요청을 하는 경우 즉, `A의 재고의 1로 변경`으로 요청하는 경우 2개의 Transaction 결과는 0이 아니라 1이 될 것이다. 이러한 문제를 해결하는 방법은 `변경할 수`으로 요청 즉, `A의 재고를 1개 줄임`으로 요청하도록 변경해야 한다.
- `select for update`을 사용한다. 처음 비관적 잠금을 위해 repository에 navtive 쿼리로 `update item set stock = stock - 1 where id = 1`와 같이 구현하려고 하였다. 하지만 이런 경우 `stock`이 0개 이하인 경우 예외가 발생 로직 처리가 쉽지 않다고 판단했다. 그래서 JPA의 `@Lock(LockModeType.PESSIMISTIC_WRITE)`을 사용하기로 하였다.

## Flyway

[Flyway](https://flywaydb.org/){: target="\_blank" }는 Database의 상태를 코드를 통한 버전관리가 가능하도록 도와주는 도구로 DDL, DML의 이력 추척 및 관리를 코드를 통해 할 수 있기 때문에 JPA와 함께 사용하면 아주 좋은 도구 이다. Spring Boot환경에서 적용방법도 아주 쉽기 때문에 이번에 사용하는 어플리케이션에 도입해서 사용하기로 하였다.

## AOP args

AOP를 적용할 때 매개변수를 활용해야 하는 경우가 종종 있다. 이번에 AOP를 적용하면서 사용한 방법이 괜찮아서 기록차 적어둔다.

```java
@Pointcut("execution(* com.example.demo.Conciliator.reconcile(..))")
public void reconcile() {
}

@Around("reconcile() && args(queryDate)")
public ReconciliationResult handle(ProceedingJoinPoint pjp, LocalDate queryDate) {
    // Some code ...
}
```

위 코드처럼 args 표현식을 사용하면 실제 Bean 객체에서 다루는 매개변수를 그대로 사용할 수 있다.
참고 링크: [https://blog.outsider.ne.kr/844](https://blog.outsider.ne.kr/844){: target="\_blank" }

## Git author 변경

현재 회사 PC에 2개의 Github 계정을 사용하고 있다. 개인 계정과 회사 계정을 분리하고 싶은 마음에 이런 결정을 하였다. 그러면서 repository 별로 auth 설정을 분리하여 코드 push 시 별도로 로그인을 하지 않아도 되도록 하였다. 그러면서 git config를 기본으로 회사 계정으로 잡아두고 개인 계정을 repository별로 config 설정해 주어야 했는데 이부분을 잊어버리면서 commit이 회사 계정으로 들어가 버렸다. 하나하나 변경해 주려고 하니 이미 변경된 코드를 rebase 한 후 강제로 다시 push해야 했다. 하지만 변경해서 push 해도 커밋로그가 깔끔하게 남겨지지 않아 그냥 이전 내용은 남겨 두고 새로운 커밋은 사용자를 잘 변경해서 하기로 하였다. 앞으로 좀더 꼼꼼하게 커밋로그를 확인해야겠다...

## IntelliJ Javadoc Display hot key

Intellj에서 Javadoc을 표시하는 단축키는 `Ctrl` + `J` 이다. 까먹지말자.

## Git Hook

Git Hook을 이용하여 코드 commit 전에 style fommating과 push전에 Test가 실행되도록 하면 좋지 않을까 라는 생각이 들었다. 다음주에 해당 기능을 적용하는 방법을 알아보고 적용해 보아야 겠다. java에는 node진영에서 사용하는 [husky](https://www.huskyhoochu.com/npm-husky-the-git-hook-manager/){: target="\_blank" }와 같은 hook을 공유하는 방법이 별도로 없는거 같다. husky를 위해서 어플리케이션에 사용하지 않는 node를 사용하는건 별로인것 같고... 좋은 방법이 없을까?

## Docker

현재 회사에서는 Docker를 아주 능숙하게 다루시는 분들이 많다. 항상 사용하면 좋을거 같다라고 느끼고 있었지만 막상 실무에 사용하지 않으니 익숙해 지지 않는 것 같다. 이번에 새롭게 구축하는 어플리케이션에서 배포시 Docker image로 배포되도록 해볼까 한다. 관련 내용은 추후에 계속 정리해 나가야 겠다.
