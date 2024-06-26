---
layout: single
title: "2020년 7월 2주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-second-week-of-july-weekly-retrospective/
---

입사 후 한달 째 되는 7월 2주차 회고를 작성한다. 이번주는 주로 새롭게 구현하는 기능에 대한 기술 검토와 관련한 내용들과 이번주에 생긴 이슈에 대한 내용을 적어보고자 한다.

## 개발 챕터 리더들의 퇴사

프론트엔드와 백엔드의 리더들이 퇴사 한다는 소식을 접했다. 솔직히 프론트엔드의 리더분께서 퇴사하신다는 소식을 들었을 땐 입사하신지 오래되신 분이라 그러려니 했는데 백엔드 리더분께서 갑작스럽게 퇴사하신다는 소식을 들으니 마음이 싱숭생숭 하다. 채용 시 나를 뽑아 주신 분이기도 하고 내 포지션이 백엔드이다 보니 앞으로 업무 방향이 어떻게 잡힐지 걱정이 되긴하다. 비록 회사가 목적조직으로 이루어져 개발 리더의 부재가 어떻게 크게 작용할 지 감이 잡히질 않는다. 조금 걱정이긴 하지만 기존에 계시던 개발자 분들이 지금까지 잘 해오셨기에 앞으로도 잘해 주시리라 믿고 나도 내 몫을 잘 하기 위해 노력해야 겠다.

## 정산

7월에 상반기 결산 때문에 커버스 담당이다 보니 백엔드 동료가 바쁜시간을 보내고 있다. 잘은 모르겠지만 실제 데이터와 사용자 결제, 회계팀에서 정산한 내용이 다른점이 많아 이슈가 좀 있는 모양이다. 아직 온보딩 중이라 제대로 도와주지는 못하고 있지만 앞으로 현재 가지고 있는 문제점 들을 근본적으로 해결하고 불필요한 작업들을 최소화 하기 위한 작업들을 해야 할 것 같다 라는 생각을 가지게 되었다. 당장 다음 주 부터라도 동료랑 현재 가진 문제점이 무엇이고 어떻게 해결해 갈 수 있을 지 고민해 보아야 겠다.

## JaCoCo

테스트 코드 작성의 필요성은 개발자라면 누구나 알고 있다라고 생각한다. 하지만 바쁜 업무에, 그리고 익숙하지 않다는 이유로 테스트 코드를 작성하는 경험이 익숙하지 않은 개발자들이 테스트 코드를 선뜻 작성하지 않고 기능을 개발하고 있는 것도 현실이라 생각한다. 그럼 어떻게 테스트 코드를 작성하게 할 수 있을까? 일단 먼저 테스트코드의 필요성을 인지시키는 것이 중요할 것 같다. 테스트 코드를 통한 안정감을 한번이라도 느끼기 시작한다면 테스트코드를 점점 작성해 가는 횟수가 늘어날 것이라 생각한다. 그리고 필요한 것이 강제성이라 생각한다. 두번째 방법은 첫번째 개발자가 테스트의 필요성을 인지하지 않은 상태에서 도입하는 것은 위험한 결과를 낳을 수 있다고 생각한다. 괜히 반감만 생기고 생산성이 떻어진다고 생각할 수 있으니 말이다. 일단 개발자가 필요성을 인지하고 있다라고 가정하고, 테스트 코드를 작성하도록 강제하는 방법이 뭐가 있을 지 찾아보았다. 코드리뷰를 통해 테스트를 작성하는 지 감시할 수 있다라고 생각할 수 있지만 리뷰로 테스트의 커버리지를 하나하나 다 체크하는 것은 효율적이지 못하다고 생각했다. 그러다 우연히 [배달의 민족의 기술블로그](https://woowabros.github.io/experience/2020/02/02/jacoco-config-on-gradle-project.html){: target="\_blank" }에서 [JaCoCo](https://www.jacoco.org/jacoco/){: target="\_blank" }라는 코드 커버리지 라이브러리를 이용한 프로젝트 설정 방법을 찾게 되었다. 내용을 요약하면 JaCoCo를 이용하여 테스트 커버리지 하한선을 적고 커버리지 조건이 만족하지 못하면 배포를 할 수 없도록 하는 방법이다. 이 방법을 이용하면 테스트 코드가 없는 코드를 병합하는게 물리적으로 어렵도록 할 수 있다. (물론 커버리지 조건에 따라 다르겠지만) 기존 프로젝트에 도입하는 것은 조금 고민해 보아야 할 것 같지만 새로운 프로젝트 생성시에는 해당 라이브러리를 추가해보면 좋을 것 같다.

## Logger Testing

Slack WebHook 기능을 사용해야할 필요가 있어서 Logbak를 이용하여 메시지를 전달하는 기능을 개발하였다. Logbak을 이용하면 logger를 사용하면 내가 원하는 메시지를 어떻게 출력할지 손쉽고도 재사용 가능하도록 사용할 수 있다는 장점이 있다. 해당 코드를 작성할 때 테스트 코드를 어떻게 작성하면 좋을 지 고민을 했었는데 마침 나와 같은 고민을 한 [좋은 글](https://medium.com/@BillyKorando/how-to-test-logging-in-java-fc36c6965bd9){: target="\_blank" }이 있어서 참고해서 작성하였다.

## Timezone

회사에서 글로벌 서비스를 위한 대비책으로 DateTime 객체를 LocalDateTime에서 ZenedDateTime으로 변경하고 있다. 예전에 프론트 코드에서 로컬타임존을 UTC로 변경하는 테스트에서도 실수를 한적이 있는데 이번에도 실수를 할뻔해서 기록차원에서 적어본다. 실수내용은 Timezone 매커니즘을 제대로 이해하지 못하고 사용한 부분이었는데, 만약 내가 서버코드에 ZonedDateTime을 `2020년 10월 10일 11시 30분 0초`로 생성하고 DB에 저장하는 기능을 구현한다고 하였을 때, 서버는 타임존이 서울이고, DB는 UTC로 되어 있다면 DB에 저장되는 값은 서버에서 넘겨준 시간보다 9시간 뒤인 `2020년 10월 10일 2시 30분 0초`로 저장되어 있다. 이는 ZonedDateTime이 알아서 계산(심지어 섬머타임까지)해 주기 때문이다. 내 실수는 타임존을 서버에서 DB에 저장 시 강제로 UTC로 변환해 주는 코드를 넣으려고 했던 것이었다. 다행히(?) 코드 작성 후에 이상하다고 느껴서 고치긴 하였지만 부끄러운 경험이었다.

## Test Data

테스트를 작성할 때 언제나 테스트를 위한 데이터를 생성하는 코드를 작성한다. 테스트를 수행하는 코드보다 테스트를 위한 데이터를 생성하는 코드가 많은 경우가 종종 있다. 나는 이럴때마다 어떻게 하면 효율적으로 테스트코드를 만들지 고민하곤 하였다. 이전에는 [Fixture 도구](https://github.com/FlexTradeUKLtd/jfixture){: target="\_blank" }를 사용하기도 하였었다. 얼마전에는 [테스트 데이터도 테스트 목적에 따라서 나뉘어 진다는 글](https://www.guru99.com/software-testing-test-data.html){: target="\_blank" }도 보았었다. 최근에 느낀 것이지만 테스트 데이터를 생성할 때에 공통으로 잘 사용할 수 있도록 만드는 것도 중요하다고 생각하였다. 아무튼 테스트 데이터 생성은 귀찮고 번거로운 작업이 될 수 있지만 잘 작성해야 테스트가 의미있고 가독성있게 작성할 수 있고, 테스트 변경에 유연할 수 있기 때문에 중요한 작업이라 생각하고 항상 더 좋은 테스트 데이터 생성을 위해 고민해야 겠다.

## Active MQ의 Consumer가 메시지를 수신하는 방식?

Active MQ를 이용한 기능 개발 기술 검토 시 동료분과 Active MQ의 Consumer가 메시지를 수신할때 Trigger 방식이라고 설명하였었다. 하지만 동료분께서는 Trigger 방식이 아닌 Polling 방식일 것이라고 얘기를 하셨다. Kafka도 Broker를 구독하는 Consumer들은 Polling 방식으로 메시지를 수신한다고 한다. Active MQ에 대한 기술 검토 시 다이어그램의 화살표만 보고, 그리고 따로 Polling에 대한 언급이 없어서 당연히 Trigger 방식이라고 생각했었는데, 내가 잘못 알고 있는 것일 수도 있겠다라는 생각을 하여서 관련 내용을 찾아봐야 겠다.

## p6spy

김영한님의 QueryDSL 강의를 듣다가 연히 DB 쿼리 로깅 시 사용하면 좋은 라이브러리를 알게 되었다. 이름은 [p6spy](https://github.com/p6spy/p6spy){: target="\_blank" }인데 라이브러리를 사용하면 logger에서 쿼리문이 출력될 때 전달한 데이터가 `?`가 아니라 전달한 데이터 자체가 보이도록 할 수 있어 편리할 것 같다.

## Delayed Delivery

현재 진행중인 기능 구현 요구사항중에 특정 기간 이후에 메시지가 발송되고 특정 기능이 수행되도록 하는 기능을 추가해 달라는 내용이 있다. 해당 요구사항을 충족하기 위해서 고민했던 것이 스케줄 기능을 이용하는 방법과 지연 발송 기능을 제공하는 Queue를 사용하는 방법이었다. 스케줄 기능보다는 지연 발송 기능이 좋다고 판단해서 지연 발송 기능을 이용하려고 Active MQ 도입을 검토했었는데, 동료분께서 기존에 MQ서비스로 카프카를 이용하고 있는데 새로운 MQ를 도입하는 것은 앞으로 MQ 관리를 하나로 통일하는 것에 반하는 것 같다고 다른 방법 도입을 요청하셨었다. 찾아보니 [Redis를 이용한 지연 발송 기능](https://redislabs.com/ebook/part-2-core-concepts/chapter-6-application-components-in-redis/6-4-task-queues/6-4-2-delayed-tasks/){: target="\_blank" }과 [Agenda](https://github.com/agenda/agenda){: target="\_blank" }라는 라이브러리가 제공하는 기능 중 [MongoDB를 이용한 지연 발송기능](https://github.com/agenda/agenda#schedulewhen-name-data){: target="\_blank" }이 있었다. 최종적으로는 이미 현재 구축되어 있는 node 서버에 Agenda를 사용하고 있어 해당 라이브러리의 기능을 사용하기로 하였다. 이번 기회에 지연발송 기능을 제공하는 많은 서비스들이 있다는 것을 알 수 있게 되었고 실제 운영하면서 어떤 이슈가 생길 지 경험할 수 있을 것 같아서 좋은 기회가 될 것 같다. (현재 redis를 이용한 지연 발송을 사용하고 있는 팀이 있는데, 대량의 메시지를 큐에 담을 경우 발송시간이 예상했던 것보다 더 지연 되는 현상이 있다는 사례를 공유해 줬었다.)
