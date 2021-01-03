---
layout: single
title: "2020년 4월 2주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

## KEEP

현재 내가 속한 개발실 파티의 파티장님의 제안으로 분산시스템을 위한 [CQRS](https://martinfowler.com/bliki/CQRS.html){: target="\_blank" } 스터디를 진행하기로 하였다. 스터디를 위해 읽기로한 자료는 [Exploring CQRS and Event Sourcing](https://www.microsoft.com/en-us/download/details.aspx?id=34774){: target="\_blank" }이다. 우리는 여기에서 **Event Sourcing**에 대한 내용은 일단 접어두고 **CQRS**에 대한 내용만 우선 스터디 하기로 하였다. 읽어야할 내용이 적지 않고 영문으로 되어있어 일단 자료를 제대로 다 읽어 보는 것 부터 해야겠다. (~~한글로 된 문서를 이해하는 것도 쉽지 않는데, 영어 문서라니...~~) 마침 영문으로 된 글을 독해를 해야하니 기존에 [WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" } 문서를 읽는 것으로 하던 영어공부를 **Exploring CQRS and Event Sourcing** 문서를 읽는 것으로 바꿔야 겠다. 대신 많은 글을 빨리 읽기 위해서 글을 읽고 이해하는 수준까지만 해석을 하고 한글로 번역하려고 하지는 않을 예정이다. 이번 스터디는 분산 시스템에서 더 나은 개발을 하기 위한 나의 시야를 좀더 넓혀 줄 수 있을 것이라 기대가 되어 좋은 기회라 생각한다. 내것으로 만들기 위해 최선을 다해보자.

최근 WebFlux를 이용한 웹 어플리케이션을 만드는 방법에 대한 튜토리얼 블로그를 시리즈 형태로 작성하고 있다. 현재까지 내가 개발해왔던 Spring MVC를 이용한 개발 방법을 WebFlux로 어떻게 구현할 수 있는지를 사례별로 소개하고 튜토리얼을 통해 개발 방법을 가이드 하면 좋을 것 같다. 이 글들은 읽는 사람들에게 아주 사소한 도움을 줄 수도 있지만 나에게는 이 글을 적으면서 명확하게 개념을 정리할 수 있어서 나를 위한 글이기도 하다. 앞으로 당분간은 WebFlux를 이용하여 여러 구현을 시도해 볼 것이고 이러한 시도들을 튜토리얼로 적어내는데 집중할 예정이다.

지난주 회고에서 계획했던 것처럼 이번주부터 책을 읽던 읽지 않던 휴대하고 다니고 있다. 책이 가방속에 있다보니 지하철에서 쓸대없이 휴대폰을 보기보다 책을 읽는 시간을 가지게 되어서 앞으로도 조금 무겁더라도 책은 항상 가방속에 두고 다녀야겠다고 생각이 든다.

## WATCH

이번 주 부터 금요일은 회사에 출근하지 않는다. 첫주에 무얼 해야하나 고민하고 있는데 친한 동료분께서 몇몇 동료들과 등산을 가자고 초대를 해주셨다. 최근에 헬스장도 계속 미뤄지고 있어서 집에서 간단한 스쿼트로만 운동을 이어가고 있는데, 등산으로 스트레스도 풀고 운동도 하고 좋을 것 같아서 가기로 결정하였다. 산은 [관악산](https://ko.wikipedia.org/wiki/%EA%B4%80%EC%95%85%EC%82%B0){: target="\_blank" }으로 가기로 하였는데 정상까지 올라가는데 4시간 정도, 내려오는데 1시간 반 정도 소요되었다. 올라갈때 시간이 많이 소요된 이유는 직선 코스가 아닌 능선을 타는 코스를 선택하여서인데 코스의 난이도가 높은 편이었다. 산을 못타지는 않는 편인데 오랜만에 등산을 하고 나니 집에 돌아와서 잠만자고 다른 활동은 아무것도 하지 못했다. 토요일에도 컨디션이 회복되지 않아서 집중을 하지 못하고 영어공부와 블로그 작성을 위한 튜토리얼 준비만 하였다. 등산을 하는 것은 좋은데 이렇게 다음날까지 컨디션에 영향을 미친다면 다음 등산을 좀더 생각해 봐야 겠다 ^^;;

## CHANGE

최근들어 나아졌다고 생각했던 부정적인 생각들이 다시 새록새록 피어나는 것 같다. 근거없이 부정적인 생각들이 늘어나는 것 같아 내가 점점 고장나고 있지 않은가 생각이 든다. 그런 생각들의 원인이 나라고 생각하고 좀 더 나은 방향으로 문제를 해결하기 위해 내 생각을 고쳐먹어야 겠다.

## LAST ACTION ITEMS

- [ ] Occupying 프로젝트 리펙토링
- [x] 하루에 30분 이상 책 읽기
- [x] 씻기전에 스쿼트 하기!
- [ ] MongoDB를 이용한 WebFlux 부하 테스트 하기

## ACTION ITEMS

- 하루에 30분 이상 책 읽기
- 씻기전에 스쿼트 하기!
- MongoDB를 이용한 WebFlux 부하 테스트 하기
- WebFlux + MongoDB 튜토리얼 작성
- CQRS 스터디 준비
- 금요일 계획

## ACHIEVEMENT RATE

영어 공부
<progress value="7" max="7"></progress>
7/7 (<b>100%</b>)

운동
<progress value="5" max="7"></progress>
5/7 (<b>71%</b>)

- [x] 블로그 작성

1. [Spring WebFlux - Functional Endpoint With Coroutine Tutorial](/tutorials/spring-web-flux-functional-endpoint-api/){: target="\_blank" }
2. [Spring WebFlux - Annotated Controller with reactor Tutorial](/tutorials/spring-web-flux-annotated-controller-api/){: target="\_blank" }
