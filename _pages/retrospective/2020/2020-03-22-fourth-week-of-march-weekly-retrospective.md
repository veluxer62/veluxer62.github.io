---
layout: single
title: "2020년 3월 4주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-fourth-week-of-march-weekly-retrospective/
---

## KEEP

최근 [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" } 문서를 다시 보게 되면서 Kotlin의 [Coroutine](https://docs.spring.io/spring/docs/current/spring-framework-reference/languages.html#coroutines){: target="\_blank" }을 공식적으로 지원한다는 것을 알게 되었다. Java에서 Reactor를 사용한다면 Kotlin은 Coroutine을 사용하는데, 현재 내가 사용하고 있는 언어가 Kotlin이다 보니 Coroutine의 지원은 반가웠다. 그래서 현재 진행하고 있는 [개인 프로젝트](https://github.com/veluxer62/occupying){: target="\_blank" }에 적극적으로 사용하기 위해 연습도 해 볼겸, R2DBC도 습득해 볼겸 toy 프로젝트로 간단한 CRUD 앱을 만들어 보고 있다. 구현 방법은 **Spring WebFlux**에서 **Functional Endpoint**방식을 이용한 API, **Coroutine**, **Spring Data R2DBC**를 이용하기로 하였다. 익숙하지 않아 현재 작성하고 있는 코드가 이상적인 방법인지는 알수가 없어 공식 가이드 문서 및 여러 사례들을 참고하면서 작성해보려고 노력하고 있다보니 Spring MVC를 이용하면 하루도 안되서 만들 간단한 CRUD 앱도 몇일째 잡고 헤매고 있다.아쉬운 부분은 아직 Spring Data R2DBC는 정식 릴리스 버전이 1개 밖에 존재하지 않고, 가장 중요한 R2DBC 모듈이 정식 릴리스 버전이 존재하지 않아 실무에 적용은 아직 시기상조인 것 같다. 그래도 새로운 방식으로 개발을 하다보니 설레기도 하고 성능이 어떨지 궁금하기도 하다. 기회가 되면 나중에 부하 테스트도 함께 해보면 재밌을것 같다.

지난주에 이어 이번주도 영어공부는 [Spring WebFlux 레퍼런스 문서](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }를 읽어보는 것으로 하고 있다. 30분 읽고 30분 번역한 내용을 정리하는 방식으로 진행하고 있는데, 독해 서적을 이용해 공부하는 것 보다 지문의 양이 절대적으로는 적지만 독해 + 기술습득의 콤포는 만족스럽다. 주말에는 독해할 때 정리해둔 것들을 읽기 쉽게 번역하는 작업을 병행 하였는데, 이때 내가 해석했던 구문이랑 실제로 번역된 내용이랑 다른 부분들을 많이 찾아볼 수 있어서 앞으로도 이러한 활동을 계속한다면 내가 지문을 읽어가는데 많은 도움이 될 수 있을 것 같다.

## WATCH

지난주부터 재개되어 이번주는 헬스장을 가게 되었다. 화요일과 목요일은 튜터일정으로 가지 못하고 나머지 날은 갈 수 있었다.(~~이번주에 2번만 간건 안비밀~~) 하지만 오늘 코로나로 인해 4월 5일까지 다시 또 휴관을 한다는 문자메시지를 받았다. 다음주부터 스피닝을 할 수 있다는 생각에 좋았지만 다시 또 운동을 한동한 못하게 되었다. 하다못해 집에서 씻기전에 스쿼트라도 해야겠다.

저번주에 이어 이번주도 책읽기와 휴대폰을 보는 것을 줄이고자 마음먹고 Action Item으로 등록하여 실천의지를 다졌다. 이번주도 저번주와 같이 만족스러운 행동을 보이지 못했고, 다음주를 위한 Action Item으로 다시 등록해야 겠다. (이러다 Action Item만 매일 등록하는건 아닌지 모르겠다.) 계속해서 지켜지지 않으면 실천 가능성을 높일 수 있는 다른 방법을 강구해보아야 겠다.

## CHANGE

이번주는 다사다난한 주였던 것 같다. 특히 앞으로 내가 회사생활을 하면서 좀더 조심하고 경솔하지 말아야 한다는 교훈을 얻을 수 있었던 한주였다.
나의 의도는 좀더 잘 해보고자 했던 행동이 누군가에게는 기분을 상하게 할 수도, 누군가에게는 가쉽거리가 될 수도 있다는 것을 배우게 되면서 말조심, 사람조심이라는 말이 사회생활에서 왜 중요한가를 알게 해 주는 계기가 되었다. 돌이켜 생각해보면 다 나의 자만심 때문이고 가벼움 때문이었다. 좀더 성숙해지고 겸손한 사람이 되자.

## LAST ACTION ITEMS

- [x] Occupying 프로젝트 리펙토링
- [ ] 하루에 30분 이상 책 읽기
- [ ] 휴대폰 보는 시간 줄이기
- [ ] 운동 열심히 하기
- [ ] 다이어트! (저녁 간단히 먹기)

## ACTION ITEMS

- Occupying 프로젝트 리펙토링
- R2DBC 튜토리얼 작성
- 하루에 30분 이상 책 읽기
- 휴대폰 보는 시간 줄이기
- 다이어트! (저녁 간단히 먹기)
- 씻기전에 스쿼트 하기!

## ACHIEVEMENT RATE

영어 공부
<progress value="7" max="7"></progress>
7/7 (<b>100%</b>)

운동
<progress value="2" max="7"></progress>
2/7 (<b>29%</b>)

- [x] 블로그 작성

1. [How to update forked repository master](/tutorials/how-to-update-forked-repository-master/){: target="\_blank" }
