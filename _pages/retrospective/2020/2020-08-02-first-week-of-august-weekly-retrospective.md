---
layout: single
title: "2020년 8월 1주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-first-week-of-august-weekly-retrospective/
---

조직이 개편되면서 자리이동이 있었다. 이전에 비해 자리가 많이 넓어져서 좀 쾌적해 져서 좋다!

조직개편과 함께 다시 또 정산시즌이 다가왔다. 거기다가 CS 당번까지 겹치면서 동료 개발자가 너무나 바쁘다. 작업을 도와주면 좋겠지만 히스토리를 잘 알지 못하는 상황에서 함수로 데이터를 손을 댓다가는 오히려 일만 더 가중 시킬 수 있어 조심스럽다. 동료가 이렇게 바쁘다 보니 진행중이던 시스템 개편 작업도 지지부진 한 상태이다.

이번주는 테스트 라이브러리를 비교하는데 많은 시간을 할애하였다. 기존에 익숙하게 사용하던 [Junit5](https://junit.org/junit5/docs/current/user-guide/){: target="\_blank" }와 이번에 새롭게 알게된 [Kotlin Test](https://www.kotlinresources.com/library/kotlintest/){: target="\_blank" }, 그리고 Groovy를 이용하는 [Spock](http://spockframework.org/){: target="\_blank" }를 사용해보고 장단점을 비교해 보는 것이었다. 이번주에 비교글을 작성하는 것이 목표였으나 아마 다음주에나 작성이 가능해 보인다.

## JPA Entity 이름 정의 시 주의 사항

아무생각 없이 주문과 관련한 Entity명으로 `Order`를 사용하였었는데 오류가 발생해서 원인을 한참동안 찾았었다. 이유는 `Order`라는 이름이 SQL에서 예약어로 사용되고 있어서 인데, 테이블 명을 사용할 때 예약어와 동일한지 체크해서 잘 사용하자 ㅠ

## spock

[Spock](http://spockframework.org/spock/docs/1.3/index.html){: target="\_blank" }은 Groovy를 이용해 사용할 수 있는 테스팅 프레임워크 이다. 동료가 Groovy를 이용한 테스트 작성 시 Junit 보다 훨씬 가독성이 좋고 코드량을 줄일 수 있다고 추천을 해주셔서 튜토리얼을 진행해 보았다. 명시적으로 Given, When, Then을 작성한다는 부분과 Data Driven Testing 시 가독성이 너무나 뛰어나서 Java로 개발된 어플리케이션에서는 Spock를 도입해서 사용하면 아주 좋을 것 같다. 다만 코들린을 사용하는 경우에는 코틀린에서 제공하는 함수등을 Groovy에서 사용하기가 쉽지 않아서 Kotlin으로 개발된 어플리케이션에서는 Kotlin Test나 Junit을 그냥 사용하는 것이 나아 보인다.

## Icarus

[Icarus](https://github.com/ppoffice/hexo-theme-icarus){: target="\_blank" } GitHub 블로그 테마다. 동료분께서 해당 테마로 블로그를 운영중이신데 괜찮아보여서 마이그레이션을 쉽게 할 수 있다면 옮겨볼까 생각중이다.

## Aurora DML throughput

현재 회사에서는 RDS로 Maria DB를 사용하고 있다. 동료분 중 한분이 Aurora DB를 사용하면 DML시 높은 성능을 보여주면서 잠금으로 인한 쿼리 실패 확율을 대폭 줄일 수 있다는 얘기를 들었다. 그 외에도 Aurora DB를 사용하면 여러 장점을 누릴 수 있다고 하는데 자세한건 아래 링크를 참고하자.

[https://www.percona.com/live/e18/sites/default/files/slides/Deep%20Dive%20on%20Amazon%20Aurora%20-%20FileId%20-%20160328.pdf](https://www.percona.com/live/e18/sites/default/files/slides/Deep%20Dive%20on%20Amazon%20Aurora%20-%20FileId%20-%20160328.pdf){: target="\_blank" }

## Range 클래스

Spring Data에서 Query Method의 Between을 사용할 때 [Range 클래스](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Range.html){: target="\_blank" }를 사용할 수 있다는 것을 알게 되었다. Range 클래스는 Spring Data 패키지에서 제공하는 클래스로 인자로 받은 두 값의 범위를 나타내는 함수이다. 제네릭 타입으로 아무 클래스를 매개변수로 사용할 수 있지만 내가 사용한다면 주로 숫자형 타입이나 날짜형 타입을 주로 사용할 것 같다. 이 클래스의 매력은 포함여부를 생성시점에 결정할 수 있고 `contains`함수를 이용하여 손쉽게 데이터 범위내에 원하는 값이 포함되어 있는지 알 수 있다는 점이다. 해당 클래스에 대한 간단한 튜토리얼을 작성해 봐야겠다.
