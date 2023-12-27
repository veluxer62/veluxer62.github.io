---
layout: single
title: "2020년 12월 3주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-third-week-of-december-weekly-retrospective/
---

연말이라 그런지 아무것도 하기 싫어증이 걸렸다.(~~핑계같겠지만 핑계다.~~) 도메인주도설계 책은 한달째 반도 않읽고 있다. 사실 내용이 머릿속에 쏙쏙 들어오질 않아서 내 이해력이 이정도 밖에 되지 않은가 하고 자괴감이 들기도 하다. 예전에 누군가 얘기한것 처럼 DDD에 대해서 제대로 알지도 못하면서 DDD를 한다고 얘기하는게 참 아이러니 하긴하다.

사내에 확진자가 발생하면서 지난주 내내 재택근무를 하였다. 결과론적이지만 확진자가 발생하기 전에 미리 재택근무를 시행했다면 어땠을까 하는 아쉬움이 많다. 확실히 재택을 하면 회사에서 업무를 하는 것보다 업무 집중도가 떨어지긴 한다. 체계와 문화가 잘 잡혀있는 경우라면 몰라도 그렇지 못하다면 더더욱 업무효율이 떨어질 수 밖에 없다. 회사에서는 다양한 방법으로 구성원들이 업무에 집중하고 효율을 발휘할 수 있도록 하기위해 노력하고 있다. 재택에 대한 부분도 적극적으로 수용해서 좀더 효과적으로 업무를 관리하고 구성원이 업무효율을 낼 수 있는 방법을 모색하면 어떨까 싶다.

아래는 지난주동안 내가 겪은 이슈나 새롭게 배운 내용을 정리한 것이다.

## Hikaricp

이전에서 써왔지만 이번에 설정하면서 정리도 할겸 다시한번 적어본다.

Hikaricp는 Spring Boot 2.0부터 기본으로 지정된 DBCP이다. 이전에는 Apache Common DBCP2 를 사용했던걸로 기억한다. 솔직히 성능차이는 직접 테스트를 해보지 않아서 차이를 체감하지 못했지만 [HikariCP GitHub 페이지](https://github.com/brettwooldridge/HikariCP){: target="\_blank" }에서는 성능상의 이점을 가진다고 한다.

Spring Boot에서 적용 예제는 아래와 같다.

```yaml
spring:
  datasource:
    hikari:
      jdbc-url: jdbc:mariadb://localhost:3306/test
      username: user
      password: password
      driver-class-name: org.mariadb.jdbc.Driver
      maximum-pool-size: 5
      minimum-idle: 5
      connection-timeout: 3000
```

다른 여러 설정도 있지만 Replica를 두지 않고 설정할 수 있는 방법 중에 가장 심플하게 설정할 수 있는 방법이라 생각된다.

커넥션 풀이 빠듯한 경우가 아니라면 `maximum-pool-size`와 `minimum-idle`을 동일하게 둠으로써 성능적으로 이점을 가져갈 수 있다. 왜냐하면 이렇게 설정하면 pool이 항상 최대값으로 idle 상태이기 때문에 pool이 새롭게 생성되는 비용을 줄일 수 있다. 덤으로 `idle-timeout`에 대한 설정도 고민하지 않아도 된다.

## DB Sharding

게임 서버와 같이 대량의 트래픽을 감당할 서버를 개발한 경험이 없어 DB Sharding이라는 개념을 이번에 처음 알게 되었다. (~~이전에 들었을 수도 있는데 기억이 나질 않는다 ㅎㅎ~)

Sharding은 파티셔닝과 비슷하게 데이터가 많은 하나의 테이블을 사용할 때 발생할 수 있는 가용성, 성능 등의 이슈를 해결하기 위한 방법이다. 샤딩에 대한 내용은 잘 정리된 [블로그](https://nesoy.github.io/articles/2018-05/Database-Shard){: target="\_blank" }를 통해서 대체한다.

이글에서 가장 인상적인 말은 아래와 같다.

> 샤딩을 피하는 방법을 우선적으로 고려해 보자

> 샤딩의 방법들 중 하나를 선택한다면 반드시 Trade-Off가 있다.

## Immutable Database

AWS에 Immutable Database로 [QLDB](https://aws.amazon.com/ko/qldb/){: target="\_blank" }라는 Database를 제공한다. 설명에서는 금융권에서 사용하면 좋다라고 적혀있는데 다른 도메인에서도 이력관리나 감사로그 등 변경하지 않아야할 데이터들을 관리하는 용도로 사용하면 아주 좋을 것 같다.

RDS랑 비교해봐도 RDS는 t3.medium / 30GB 기준 온디멘드 요금이 월별 78.92 USD가 나오는데 QLDB는 100만거의 쓰기/읽기 IO에 30GB 용량을 사용하는 경우 73.06 USD로 비슷하거나 더 낮은 가격대를 형성한다.

## Kotlin Graphql

얼마전에 구독설정해 둔 알림을 통해 Kotlin을 이용한 Graphql 사용방법을 알려주는 영상을 보게 되었다. 현재 회사에서 Node환경에서 Graphql을 사용하고 있는데 Kotlin을 통해서 Graphql을 사용하는것도 좋아보이긴 하다. [영상 링크](https://blog.jetbrains.com/kotlin/2020/12/expedia-group-bootiful-apis-with-graphql-and-kotlin/?mkt_tok=eyJpIjoiTVRWa01qRmpPV05sT0dVMSIsInQiOiJzT0NkSWpZanhjYU1mVEY5dmNkRFwvNzQ5ZDNZWFZCQ0Y5cnljSTEzcmh4NzB3YlFaYjZ3a3U0dHRtK1I0ZEVVbFUxTkY1UXJmOUpUS0wxb3BtNEVpVloxZmhDOWRJS09DdGNGeEcxc00wWTczWE9WMm9MdFZnZm1QdFJtVWpKdEEifQ%3D%3D){: target="\_blank" }

영상에서는 Graphql 뿐만 아니라 Coroutine도 함께 소개하는데 Graphql서버가 Coroutine을 사용하면 논블락킹을 이용해 효과적으로 여러 데이터를 한번에 조회할 수 있기때문에 Gateway 역할을 할때 장점을 누릴 수 있다.

기회되면 튜토리얼 한번 해봐야 겠다.

## ActiveMQ

인벤토리 서비스를 만들면서 Delayed Queue를 사용할 일이 있어서 ActiveMQ를 사용해 보기로 했다. 이전에 [Active MQ Tutorial](https://veluxer62.github.io/tutorials/active-mq-tutorial-by-scenario/){: target="\_blank" }을 해보기도 했어서 설정은 손쉽게 할 수 있었다.

### Docker compose

로컬 환경에서 직접 설치해서 하는건 크게 어렵지 않다. 하지만 우리는 Docker라는 아주 좋은 컨테이너 환경을 이용할 수 있기 때문에 Active MQ도 Docker compose를 통해서 Docker 배포를 할 수 있다.

아래 샘플 파일을 보자.

```yaml
version: "3.1"

services:
  activemq:
    image: ivonet/activemq
    ports:
      - 8161:8161
      - 61616:61616
      - 61613:61613
    volumes:
      - .log/activemq:/var/log/activemq
```

Docker 이미지에 대한 정보는 아래 링크를 참고하자.

[https://hub.docker.com/r/ivonet/activemq/](https://hub.docker.com/r/ivonet/activemq/){: target="\_blank" }

### Test Error Log

activemq 를 사용할때 테스트환경에서 in-memory로 실행하면 마지막에 오류가 반환됨을 볼 수 있다.
실행해는 문제 없지만 해당 로그가 보이는게 싫으면 아래와 같이 설정하면 된다.

```yaml
## application.yml
spring:
  activemq:
    in-memory: true
    broker-url: vm://localhost?broker.persistent=false&broker.useShutdownHook=false
```

참고 : [https://stackoverflow.com/questions/9591203/unable-to-shutdown-embedded-activemq-service-using-the-built-in-brokerservice-st](https://stackoverflow.com/questions/9591203/unable-to-shutdown-embedded-activemq-service-using-the-built-in-brokerservice-st){: target="\_blank" }

## GIT 브런치 이름 변경

Git에서 로컬 브런치의 이름을 변경하려면 아래와 같이 하면된다.

```bash
git branch -m 변경전_branch_name 새로운_branch_name
```

## k8s HPA

deployment의 replica 수와 hpa의 minReplicas 수가 다른 경우 어떻게 동작할까?

예를 들어 deployment의 replica가 2이고 hpa의 minReplicas가 1인 경우 배포시 최초 2개가 생성되고 hpa의 에이전트에 의해 1개로 줄어든다.

## Bundle install 에러

Mac OS를 Big Sur로 업데이트하고나서 bundle 명령어를 실행하니 아래와 같이 오류가 발생하였다.

```
an error occurred while installing commonmarker (0.17.13), and Bundler cannot continue
... 이하 생략
```

해결방법은 `rbenv`와 `ruby-build`를 아래와 같이 설치해주고 배시 실행 파일을 조금 수정하면 되었다.

```
brew update
brew install rbenv ruby-build
```

```
vim ~/.zshrc
```

아래 내용 추가

```
[[ -d ~/.rbenv  ]] && \
  export PATH=${HOME}/.rbenv/bin:${PATH} && \
  eval "$(rbenv init -)"
```
