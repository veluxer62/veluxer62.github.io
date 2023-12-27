---
layout: single
title: "2020년 3월 5주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-fifth-week-of-march-weekly-retrospective/
---

## KEEP

얼마전에 [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc){: target="\_blank" }의 릴리스 버전이 1.0.0 버전이 출시되어 있다는것을 알게 되면서 Spring Data R2DBC를 이용하여 [Spring WebFlux 샘플 앱](https://github.com/veluxer62/toy/tree/master/spring-data-r2bc-example){: target="\_blank" }을 만들어 보았다. 아직 공식적으로 제공하는 R2DBC 라이브러리가 H2, PostgreSql, MsSql 밖에 없고 0.8 버전이라서 현업에서 아직 사용하기에 무리가 있어 보인다. 그리고 공식 문서에도 나와 있지만 ORM이 아니기 때문에 JPA에 익숙한 나로써는 당연히 되어야 한다는 기능이 (e.g 자동 DDL 기능) 되지 않아서 직접 구현해줘야 할 부분들이 많다는 점도 개선점이 많이 보였다.

Spring WebFlux로 구현된 기능에 대해 Spring MVC와 비교해서 부하테스팅도 진행해 보았다. 기대했던것과 같이 Spring WebFlux에서는 안정적인 응답속도와 Thread 개수를 유지해주는 모습을 볼 수 있었다. 하지만 Spring MVC는 모든 응답에 대한 성공 응답을 반환하지만 Spring WebFlux는 일부 응답이 실패하는 현상을 발견할 수 있어 안정성 측면에서는 아직 보완해야할 부분이 있어 보였다. 로그를 보니 R2DBC의 DB Connection에서 오류가 발생하는 듯 하였다. 관계형 데이터베이스가 가진 Connection Pool의 문제인듯 한데 no-sql 데이터베이스인 MongoDB를 이용한 테스트를 다시한번 해보아야 겠다.

부하테스트 도구로 [nGrinder](http://naver.github.io/ngrinder/){: target="\_blank" }를 이전에 사용해 보았는데 이번에 새롭게 설치하려고 하다보니 Java 7을 설치하라고 해서(~~Java 8로 업그레이드 해주시면 안되나요 ㅠㅠ~~) 다른 유명한 도구인 [JMeter](https://jmeter.apache.org/){: target="\_blank" }를 사용해보기로 했다. 사용법은 튜토리얼을 따라서 해보니 크게 어렵지는 않았다. 하지만 로컬용 테스트로는 나쁘지 않은데 별도 부하테스트용 서버로 이용하기에는 nGrinder가 나아보였다. 또 UI가 고해상도를 지원하지 않는 점도 조금 아쉬웠다.

## WATCH

현재 영어 공부를 [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }를 번역하는 것으로 공부하고 있다. 30분 읽고 30분 번역한 내용을 적는 방법으로 공부를 하고 있는데 이전에 책으로 공부하는 방식에 비해 절대적인 양이 부족해 보인다. 하지만 이전에는 책에서 번역된 내용을 읽기만 하였다면 이번에는 번역된 내용이 없기 때문에 직접 읽고 번역해보아야 한다. 번역기의 힘을 빌리기도 하지만 좀더 문맥을 이해하는데 노력을 기울일 수 있어서 도움이 되는 것 같기도 하다. 아직까지도 어느 방식이 더 낫다고 판단하기가 쉽지가 않다. 계속 지켜봐야겠다.

## CHANGE

운동을 제대로 안한지 너무 오래 됬다. 코로나 때문에 헬스장이 휴관기간이 오래되면서 처음 한주 두주는 기분좋게 휴식을 가지자는 마음으로 운동을 쉬었는데 휴관 기간이 길어지면서 이제는 집에서 운동을 안하면 안되겠다는 생각을 가지게 되었다. 집에서 씻기전에 운동을 해야겠다.

## LAST ACTION ITEMS

- [ ] Occupying 프로젝트 리펙토링
- [x] R2DBC 튜토리얼 작성
- [x] 하루에 30분 이상 책 읽기
- [ ] 휴대폰 보는 시간 줄이기
- [x] 다이어트! (저녁 간단히 먹기)
- [ ] 씻기전에 스쿼트 하기!

## ACTION ITEMS

- Occupying 프로젝트 리펙토링
- 하루에 30분 이상 책 읽기
- 씻기전에 스쿼트 하기!
- MongoDB를 이용한 부하 테스트 하기

## ACHIEVEMENT RATE

영어 공부
<progress value="7" max="7"></progress>
7/7 (<b>100%</b>)

운동
<progress value="0" max="7"></progress>
0/7 (<b>0%</b>)

- [x] 블로그 작성

1. [Spring WebFlux - Spring Data R2DBC tutorial](/tutorials/spring-data-r2dbc-tutorial/){: target="\_blank" }
