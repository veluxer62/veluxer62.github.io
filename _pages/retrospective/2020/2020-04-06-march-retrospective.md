---
layout: single
title: "2020년 3월 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-march-retrospective/
---

## KEEP

이번달도 2월과 마찬가지로 하루도 빠지지 않고 영어공부를 하였다. [English for Developers](http://www.yes24.com/Product/Goods/19992192){: target="\_blank" }를 이용하여 하던 공부에 비해서 효율이 많이 떨어지긴 하였지만 그래도 꾸준히 영어 공부를 하고 있다는 것에 만족을 하고 있다. [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }페이지를 이용하여 영어 공부를 할 때 처음에는 단순히 독해 후 단락별로 해석한 내용을 적었었는데 현재는 완전히 우리말로 번역한 내용들을 적어가며 진행중이다. 하루에 번역할 수 있는 내용이 많지 않아 언제 해당 문서를 다 번역할 수 있을 진 모르겠지만 매끄럽게 영문을 우리말로 번역을 하면서 독해력을 좀더 키울 수 있을 것 같고 번역을 완료하고 나면 뿌듯함과 Spring WebFlux에 대한 이해도가 좀더 높아 질 수 있다는 추가적인 이익도 가져갈 수 있을 것 같다.

이번달은 새로운 기술습득과 이전해 사용해 보지 않았던 툴들을 많이 경험해보았다. 3월에 새롭게 배운 기술 및 툴들은 아래와 같다.

- 먼저 개인 프로잭트인 [Occupying](https://github.com/veluxer62/occupying){: target="\_blank" }의 구조 개선 및 신규 기능 추가를 위해 시퀀스 다이어그램과 클래스 다이어그램을 작성해 보기로 하였다. 작성할 때 사용한 툴은 오픈소스인 [PlantUML](https://plantuml.com/){: target="\_blank" }로 Web 상에서 시퀀스 다이어그램이나 클래스 다이어그램을 그릴 수 있는 도구를 사용하였다.

- [Occupying](https://github.com/veluxer62/occupying){: target="\_blank" }에서 기존에 [Reactor](https://projectreactor.io/)를 이용하는 방식에서 [Coroutine](https://docs.spring.io/spring/docs/current/spring-framework-reference/languages.html#coroutines){: target="\_blank" }을 사용하는 방식으로 변경하면서 작업 중이다. 작년까지는 Spring WebFlux에서 공식적으로 Coroutine를 지원해 주지 않았지만 최근에 프로젝트를 개선하려고 WebFlux 문서를 다시보니 Coroutine를 공식적으로 지원해주고 있어 프로젝트에서 Kotlin을 사용하고 있으니 Coroutine을 사용하는 것으로 변경하는게 나아 보여 변경하면서 진행 중이다. Coroutine의 사용 형태 자체는 javascript의 async와 비슷해서 어렵지 않게 사용하고 있지만 정확히 개념을 이해하고 사용하기 위해서는 좀더 시간을 들여 공부를 해야할 필요가 있어 보인다.

- [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc){: target="\_blank" }를 이용한 간단한 튜토리얼을 위한 앱을 만들어 보앗다. Spring Data R2DBC가 1.0 버전이 나왔다는 것을 얼마전에 알게 되었다. 아직 현업에서 사용하기에는 무리가 있어 보이긴 하지만(R2DBC 버전이 아직 0.8 버전이다.) 1.0 버전이 나왔으니 한번쯤 사용해 보는것이 좋을 것 같아보여서 튜토리얼 블로그도 작성해볼 겸 간단한 앱을 만들어 보았다. 이후에 No-SQL을 이용한 앱도 만들어볼 계획이다.

- Spring WebFlux와 Spring MVC의 가용성 테스트를 위해서 [JMeter](https://jmeter.apache.org/){: target="\_blank" }를 사용해 보았다. JMeter는 Apache에서 만든 웹 어플리케이션의 성능을 테스트하기 위한 아주 유명한 오픈 소스 도구이다. 이전에는 [nGrinder](http://naver.github.io/ngrinder/){: target="\_blank" }를 이용한 테스트를 했었는데, 단일 로컬 환경에서 테스트 하기에는 JMeter가 좀더 나아 보여 사용해 보았다. (~~nGrinder는 JDK7을 설치해야 해서 설치하기 귀찮아...~~) UI 적인 측면과 테스팅을 위한 서버를 구축하고 사용하기에는 nGrinder가 나아보이지만 간단한 로컬 환경 테스트는 JMeter가 나아 보인다. 하지만 고해상도 지원이 되지 않아서 좀 불편하긴 하였다.

매주 책상에 둘 꽃을 산다. 아내가 꽃을 좋아하기도 하지만 책상에 이쁜 꽃이 있으니 내가 작업을 할때도 분위가 확 살아서 이제는 꽃이 시들때 쯤이면 내가 사고 싶어서 꽃을 사다 둔다. 앞으로도 자주 사다 놓아야 겠다.

![flower](/assets/images/retrospective/flower.jpg)

## WATCH

영어 공부방법을 이번달부터 변경하면서 방식에 대한 고민을 아직도 하고 있다. 일단 선택은 WebFlux에 대한 문서 번역과 번역한 내용을 기록하는 방식으로 하였지만 이전에 비해 읽는 영어 지문의 양이 많이 줄어있어서 이 부분에 대해서는 아직까지도 잘 하고 있는지 아닌지 고민이다. 그래도 일단 칼은 뽑았으니 무라도 썰어봐야겠다는 심정으로 WebFlux 문서를 일단 모두 번역하고 나서 다음에 어떻게 공부를 할지 결정해봐야 겠다. (아마 WebFlux 번역은 2~3개월 걸리지 않을까 생각된다 ㅠㅠ)

의도적으로 책을 읽기 위한 노력을 계속해서 이어가고 있다. 이런 저런 이유로 가장 먼저 미루는 활동이 책읽기가 되곤 하는데 책을 읽고자 하는 내 의지가 강하다면 짧은 10분이라도 책을 읽는데 시간을 할애할 수 있을 것이란 생각에 지난 내 과거를 반성해 본다. 단시간에 습관을 만드는 것이 가능하지 않을 거라 생각하고 있지만, 책을 더 자주 읽기 위해서 잘보이는 곳에 두고 항상 들고 다닐 수 있도록 한번 전략을 바꿔봐야 겠다. ~~그래도 이번달은 읽은 책이 두권이나 되네 :)~~

## CHANGE

확실히 1, 2월에 비해 3월이 되니 전에 비해 공부에 대한 의지가 많이 줄어든게 느껴진다 (~~1, 2월이 너무 과했을 수도 있다~~)
물론 이부분을 예상하지 않았던 것은 아니지만 다시한번 마음을 다잡아야 할 필요성을 느낀다. 우선 규칙적으로 잠자는 시간을 지키고 휴대폰을 보는 시간을 줄이는 것이 필요해 보인다. 이부분은 1월부터 계속해서 고쳐야 한다고 말하고 있지만 아직까지도 잘 고쳐지지 않고 있다. 이정도면 나란 녀석이 의지가 없는게 아닐까 생각된다. 고칠 수 있는 다른 방법을 모색해봐야 겠다.

## LAST ACTION ITEMS

- [x] 다음 영어공부 방법 고민하기
- [ ] 디자인 패턴 코드로 공부하기
- [x] 집에서 할 수 있는 간단한 운동법 찾기
- [ ] 잠자리에 누으면 휴대폰 보지 않기
- [x] Occupying 프로젝트 설계 완성

## ACTION ITEMS

- 항상 책을 휴대하기
- 금요일에 할일 정하기
- 디지털 노마드 책읽기
- WebFlux 튜토리얼 및 성능 비교 포스팅 하기

## ACHIEVEMENT

### 월간목표

- [x] 책읽기

1. [Neo4j로 시작하는 그래프 데이터베이스 2/e](/book-review/graph-database-starting-with-neo4j/){: target="\_blank" }
2. [넛지](/book-review/nudge/){: target="\_blank" }

### 주간목표

- 블로그 1개 이상 작성하기

  <progress value="4" max="5"></progress> 4/5 (<b>80%</b>)

- 주간 회고 작성하기

  <progress value="5" max="5"></progress> 5/5 (<b>100%</b>)

### 일일목표

- 영어공부 30분 이상 하기

  <progress value="31" max="31"></progress> 31/31 (<b>100%</b>)

- 운동 하기

  <progress value="3" max="31"></progress> 3/31 (<b>10%</b>)

![January Calendar](/assets/images/retrospective/march-calendar.png)
