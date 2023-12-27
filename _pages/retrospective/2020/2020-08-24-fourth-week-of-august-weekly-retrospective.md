---
layout: single
title: "2020년 8월 4주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-fourth-week-of-august-weekly-retrospective/
---

교회발 코로나 전염이 다시 확산되면서 회사에서 재택근무를 권장함에 따라 난생처음 재택근무를 하게 되었다. 집에서 업무를 한적은 있지만 재택으로 각을 잡고 일은 한적은 처음이라 업무 퍼포먼스가 잘나올지 반신반의 하였다. 이틀정도 재택을 해본 결과 아직 재택을 할 준비가 덜된거 같다라는 결론을 내렸다. 다행히 집에는 업무를 하기에 불편하지 않는 탁자와 의자가 있었지만 집이라는 공간에서 일한다는 것 자체가 집중력을 발휘하기 힘든 공간이었고, 집중이 안되니 업무 효율성이 많이 떨어지는 것을 느낄 수 있었다. 아무래도 다음주에는 조심스럽게 사무실로 출근을 해야 할것 같다.

아래는 이번주에 내가 새롭게 알게 되었거나 고민했었던 내용들이다.

## Kotlin 1.4

코들린 1.4가 출시 되었다. 

[https://blog.jetbrains.com/ko/kotlin/2020/08/kotlin-1-4-released-with-a-focus-on-quality-and-performance-ko/](https://blog.jetbrains.com/ko/kotlin/2020/08/kotlin-1-4-released-with-a-focus-on-quality-and-performance-ko/){: target="\_blank" }

주된 피처는 IDE 성능 개선과 컴파일러 추가 언어 기능 추가 등이 있다.

## Event-Driven Architecture

[MongoDB를 사용한 이벤트 중심 아키텍처](https://www.mongodb.com/presentations/-mongodb-event-driven-webinar-recording) 웨비나를 들었다. MongoDB 웨비나이다 보니 DB 중심으로 아키텍쳐를 구성하는 것은 조금 아쉽긴 했지만 아키텍쳐 구성 방법에 대한 시야를 넓힐 수 있어서 좋은 경험이었다.

## Insomnia

API 테스트나 디버깅용으로 주로 [POSTMAN](https://www.postman.com/){: target="\_blank" }을 사용했었다. [Insomnia](https://insomnia.rest){: target="\_blank" }는 회사 동료가 알려준 도구로 POSTMAN과 거의 흡사한 기능을 제공해준다. 차이점은 [여기](https://gist.github.com/samoshkin/c0a2c0dd85b1d5b02d893a0f6ac0e93c){: target="\_blank" }를 참고하자. (프리미엄 버전 말고는 큰 차이는 없어보인다.)

## jOOQ

[jOOQ](https://www.jooq.org){: target="\_blank" }는 자바 기반 SQL 쿼리 API이다. [QueryDSL](https://github.com/querydsl/querydsl){: target="\_blank" }과 아주 흡사해 보이는데 두 라이브러리의 차이가 있는지는 한번 튜토리얼을 해봐야 겠다.

## kotlinfixture factory

현재 진행중인 프로젝트에 테스트 데이터 생성 도구로 [kotlinfixture](https://github.com/appmattus/kotlinfixture){: target="\_blank" }를 사용중이다. kotlinfixture를 사용하면 테스트 시 데이터 생성을 위한 코드가 많이 줄어들어 테스트 코드의 가독성을 많이 줄일 수 있다. 다만 테스트할 객체 생성 시 생성 조건이 존재한다면 fixture 시 의도하지 않게 테스트가 실패하는 경우가 생길 수 있다. 그래서 kotlinfixture에서는 이러한 [제약조건에 맞게 객체를 생성하도록 도와주는 설정](https://github.com/appmattus/kotlinfixture/blob/main/fixture/configuration-options.adoc){: target="\_blank" }이 존재한다. factory를 설정하면 각 fixture 코드마다 데이터 생성 코드를 적지 않아도 되기 때문에 유용하게 사용할 수 있다.
