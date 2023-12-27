---
layout: single
title: "2020년 7월 3주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-third-week-of-july-weekly-retrospective/
---

뭐했다고 벌써 7월 19일이다. 7월도 이제 하반기에 접어들고 있다. 이제 좀 적응 했다고 현재 재직중인 회사에서 일아나고 있는 일들이 조금씩 보이기 시작했다. 이전에는 심각하게 받아들였었을 것 같은 일들이 이제 모든 회사에 있을법한 일이라 생각해서 인지 그러려니 하고 느껴진다. 내 마음가짐을 다잡고 앞으로 내가 무얼 할 수 있고 무얼 하고 싶은지 고민하면서 업무를 해나가야 겠다.

아래는 이번주 동안의 이슈내용을 정리한 내용이다.

## Docker

도커의 장점을 얘기하자면 내가 굳이 적지 않아도 많은 사람들의 글들을 보면 충분히 알 수 있다. 첫 1~2년차 때에 Docker에 대해서 알게 되었고 그 당시 조금 공부하다가 다른 관심사에 밀려서 공부를 더 깊이 하지 않았었다. 그때 공부를 좀 깊이 하였다면 내가 지금 Docker에 대해서 공부를 다시 시작하려 하면서 느끼고 있는 막막함을 느끼지 않았지 않았을까!ㅠㅠ 아무튼 얼마전에 Docker를 좀더 코드화해서 관리할 수 있는 기능인 [docker-compose](https://docs.docker.com/compose/){: target="\_blank" }에 대해서 알게 되었고 내가 자주 사용하는 서비스들을 코드화해 두면 PC나 여러 VM 환경에서도 손쉽게 원하는 서비스를 설치할 수 있을 것 같아서 조금씩 연습해 보고 있다. 지금은 Redis와 MongoDB만 코드화 하였지만 앞으로 점점더 필요한 서비스를 위한 compose 설정 파일들을 만들어 가야 겠다. 해당 설정 파일들의 repository는 [여기](https://github.com/veluxer62/docker-compose){: target="\_blank" }에서 보면 된다.

## Airflow를 이용한 Cron Job

[Airflow](https://airflow.apache.org/){: target="\_blank" }는 이전 직장 동료분께서 여행사 크롤링을 위한 배치잡을 위해서 사용하는 것을 본적이 있었다. 당시 편리한 UI와 손쉬운 설정으로 사용하면 좋겠다라는 생각만 했었는데, 여기에서도 PA팀에서 Cron Job을 모두 Airflow로 전환하고 앞으로는 Airflow 사용을 권장하게 되었다. 최근에 내가 개발한 대사 기능을 일일 Cron Job으로 실행할 필요가 있어서 PA팀의 권장사항에 따라 Airflow를 이용하려고 하였는데, 아직 Docker의 개념이 잘 정리되지 않은 상황에서 Airflow를 사용해보려 하니 가이드 없이는 사용하기가 쉽지 않았다. 동료의 도움을 받아 좀 많이 배워봐야 겠다.

## Promise All

Javascript는 async, await를 이용하여 non-blocking 함수를 작성할 수 있다. 이번에 어떤 기능을 구현할 때 여러 non-blocking 함수를 호출하여 반환한 값을 조합해서 사용해야 하는 상황이 있었는데, 처음에는 아래와 같이 구현하였었다.

```javascript
async foo() {
  const data1 = await foo1();
  const data2 = await foo2();

  // ...
}
```

해당코드는 `foo1`함수가 실행되고 난 후 `foo2`함수가 실행된다. 둘다 non-blocking 함수이기 때문에 thread는 효율적으로 사용할 수 있겠지만 만약 `foo1`함수가 2초 `foo2`함수가 3초가 걸린다면 `foo`함수는 최소 5초 이상이 소요된다.

해당 코드를 보고 동료분께서 코드의 효율을 위해 [`Promise.all`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/all){: target="\_blank" } 사용을 추천해 주셨고 해당 코드를 사용하므로써 내가 구현한 코드가 좀더 효율적인 기능을 수행할 수 있게 되었다.

```javascript
async foo() {
  const [data1, data2] = await Promise.all([
    foo1(),
    foo2()
  ]);

  // ...
}
```

위 코드와 같이 작성하면 `foo1`과 `foo2`는 병렬로 실행되므로 `foo`함수는 최소 3이상이 소요된다.

자바에서 blocking 코드 작성에 익숙한 내가 Node.js 코드를 작성하면서 많은 부분을 배울 수 있을 것 같다. 배운 부분들을 나중에 Webflux를 사용할 때 많이 활용해보면 좋을 것 같다.

## Notion Github Integration

회사에서는 협업관리툴로 Notion과 Clickup을 사용한다. ClickUp은 Jira나 Azure 보다는 유명하진 않지만 나름 괜찮은 UI를 제공하고 현재 회사에서 원하는 기능을 충분히 제공하고 있어 나쁘지 않다고 생각하고 있다. 하지만 실제 운영부서분들은 Notion을 적극적으로 사용하고 있고 ClickUp의 사용권한이 없는 경우가 많아 Notion과 GitHub의 연동으로 업무관리를 하면 좋지 않을까 라는 생각에 찾아보게 되었다.

아쉽게도 Notion에서 아직 Github 연동 기능은 제공하지 않고 있다. [https://github.com/n8n-io/n8n/issues/48])(https://github.com/n8n-io/n8n/issues/48){: target="\_blank" } 빠른 시간내에 해당 기능이 나오면 좋을 것 같다!

## Random User API

GraphQL 공부를 하면서 책에 임시로 사용자 계정을 사용하기 위한 [Random User API](https://randomuser.me/){: target="\_blank" } 기능을 소개해 주었다. 사용자 계정을 이용한 여러 어플리케이션을 개발하면서 사용하면 아주 좋을 것 같은 API라서 꼭 기억해두고 두고두고 활용해야 겠다.

## Constructor binding

Kotlin을 이용한 Spring 사용 시 `@ConfigurationProperties`를 이용해서 properties를 Data Class로 사용하려고 하면 binding이 제대로 되지 않는 이슈가 있다. 이는 binding 시 기본 생성자를 이용하여 객체를 생성한 후 setter를 이용하여 데이터를 주입하는 방식을 이용하는 Java Beans 정책 때문인데, 이런 문제를 해결하기 위해서 Data Class 생성 시 default 값을 주거나 Data Class가 아닌 일반 Class를 이용하는 방법등을 고려할 수 있다. 하지만 나는 Properties의 정보를 제공하는 역할을 수행하는 클래스이므로 Data Class로 선언됨이 맞다고 생각하고 있고 기본값을 주는 것은 의도하지 않은 값이 제공될 수 있다고 생각해서 좋지 않은 방법이라 생각했었다. 그러던 중 [레진 코믹스의 기술 블로그](https://tech.lezhin.com/2020/07/15/kotlin-webflux){: target="\_blank" }에서 `@ConstructorBinding`을 활용한 불변한 설정 객체를 만드는 방법에 대해 알게 되었다. 앞으로 Properties 클래스를 사용할 때에는 해당 어노테이션을 활용해야겠다. 해당 설정을 사용하는 방법은 별도로 튜토리얼을 진행해 봐야겠다.
