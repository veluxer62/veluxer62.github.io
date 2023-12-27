---
layout: single
title: "2020년 11월 4주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-fourth-week-of-november-weekly-retrospective/
---

기능이 어느정도 그림이 완성되고 개발속도도 어느정도 붙었다 싶었는데, 이번주만 요구사항이 두번이나 변경되었다. 어느정도 이해되는게 기능이 완성됨에 따라 이해관계자와 기획자의 이해도도 증가하게되고 관심도도 증가하게 되면서 처음에 본인이 생각했던 것과 다르다는 것을 느끼면서 요구사항 변경 요청을 하는 것이리라. 이번 프로젝트를 하면서 느낀점은 이해관계자와 요구사항정리를 처음에 빡세게 하지않으면 지금과 같이 추후 변경 내용이 많이 발생할 수 있다는 것이다. 앞으로 프로젝트를 시작할때 요구사항을 정리하는 시간을 꼭 넉넉하게 그리고 이해관계자들이 자신의 요구사항이 무엇인지 명확히 알 수 있도록 끊임없이 질문해야 겠다.

스크럼 방식으로 업무를 진행하면서 급한 업무라던지 계속해서 추가되는 신규 업무를 어느정도 받아들이는거는 이해가 된다. 스타트업에서 TF팀이 아닌 경우라면 스크럼방식으로 업무를 진행한다면 계획되지 않은 업무가 발생하는 것은 어쩔수 없는 부분이고 그러한 부분을 받아주기위해서 어느정도 스토리 포인트를 남겨두고 진행해야 되는것이 맞다고 생각한다. 하지만 그래도 기본적인 스크럼방식의 룰을 깨려고 하지 않으면 좋을 것 같다. 적어도 계획했던 일은 해야하고 그러한 일을 한 후에 회고를 통해 방향성을 바꿀 순 있지만 스프린트 중간에 요구사항이 변경되고 계획했던 작업 내용을 변경하는 것은 기본 룰을 깨는 것이라 조심해야 할것 같다.

아래는 이번주에 내가 겪었거나 고민했던 이슈들을 적은 것이다.

## Gradle 캐시

지난 주에 작업하던 프로젝트를 Spring Boot 2.4.0으로 업데이트 하였다. 작업하면서 큰 이슈는 없었는데 IDEA에서 Jackson Annotation이 동작하지 않는 현상이 있었다. 프로젝트를 다시 받아보기도 하고 IDEA 캐시도 지워보고 다 해보았는데 안되길래 Gradle 캐시를 지워보니 잘 동작하였다... 참 좋은 기능이지만 가끔 이렇게 애를 먹이는게 캐시인듯 하다.

그래이들에서 캐시를 지우는 방법은 아래 명령어를 사용하거나

```
./gradlew cleanBuildCache
```

캐시 디렉토리를 지워주면 된다.

```
rm -rf ~/.gradle/caches/
```

추가로 회사 동료가 그래서 추천해준 툴이 있는데 맥에 있는 캐시들을 초기화 해주는 툴이다. 자주 하면 우리의 맥이 힘들어 할테니 가끔 생각나면 한번씩 돌려보면 좋을 것 같다.

[https://macpaw.com/cleanmymac](https://macpaw.com/cleanmymac){: target="\_blank" }

## jackson 카멜 케이스를 스네이크 케이스로 변경해주는법

```
@Data
@Builder
@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
public class Person {
    private String firstName;
    private String lastName;
    private String byName;
    private String phoneNumber;
}
```

## kotlinfixture 1.0.0

kotlinfixture 애용하고 있는데 메이저 버전이 아직 없어서 아쉬웠는데 드디어 메이저 버전이 나왔다!

[https://github.com/appmattus/kotlinfixture/releases/tag/1.0.0](https://github.com/appmattus/kotlinfixture/releases/tag/1.0.0){: target="\_blank" }

## ElementCollection N + 1

JPA에서 연관관계 매핑 시 N + 1 문제가 많이 발생한다. 이는 ElementCollection에서도 동일하게 발생하는데, 이번에 이 문제를 겪게 되었고 다양한 방법이 있지만 현재 QueryDSL을 사용하고 있으므로 fetchJoin으로 해결하였다. 조심해야 하는 부분은 여러개의 테이블을 조인할 때 마지막에 fetchJoin을 적어주면 안되고 하나의 테이블 마다 모두 적어주어야 한다. (마지막에 한번만 써주면 마지막 테이블만 조인한다.)

```kotlin
val list = jpaQueryFactory
      .selectFrom(kitEntity)
      .innerJoin(kitEntity.options)
      .fetchJoin()
      .innerJoin(kitEntity.inventoryHistories)
      .fetchJoin()
      .innerJoin(kitEntity.settlements)
      .fetchJoin()
      .innerJoin(kitEntity.shippings)
      .fetchJoin()
      .where()
      .orderBy(kitEntity.id.desc())
      .offset(filter.paging.page.toLong())
      .limit(filter.paging.take.toLong())
      .fetch()
```

## 그레이들 버전 변경 방법

```
 ./gradlew wrapper --gradle-version 6.7.1 
```

근데 그레이들 버전을 변경하니까 QueryDSL에서 생성한 Q 클래스들이 인식이 되지 않는다. 동료들은 문제가 없는걸 보니 내 PC에서 문제인거 같은데 확인해봐야 겠다.

## sequence table cache

sequence table을 이용해서 PK를 사용할때 시퀀스 값이 1에서 값자기 1000으로 올라가 버리는 현상을 발견할 수 있다. 이는 sequnce table의 cache 때문인데, 아무 설정을 하지 않으면 기본값이 1000이다. cache를 하는 이유는 등록 성능 향상을 위한 것인데 insert가 많은 작업에서는 cache 설정을 한것과 안한것에 성능차이가 크기 때문에 반드시 설정을 해주면 좋다.

다만 지금 내가 만들고 있는 서비스의 테이블의 경우 insert보다는 조회나 업데이트가 자주 일어나는 테이블이라 insert의 성능이 중요하지 않고 시퀀스 값이 순차적으로 증가함을 보장하는게 중요하므로 no cache로 설정하였다. no cache로 설정하는 방법은 아래와 같다.

```
CREATE SEQUENCE sample_sequence START WITH 1 INCREMENT BY 1 NOCACHE;
```

참고자료 : [https://m.blog.naver.com/PostView.nhn?blogId=kwoncharlie&logNo=10113535370&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=kwoncharlie&logNo=10113535370&proxyReferer=https:%2F%2Fwww.google.com%2F){: target="\_blank" }

## 육각형 아키텍쳐(Hexagonal Architecture)

개발자 채용과제에 참가자분중에 한분이 육각형 아키텍쳐를 사용해서 과제를 제출하신분이 계셨다. 육각형 아키텍쳐로 개발되어 있는 코드를 본적이 없어서 처음에는 전략 패턴을 사용하셨나 했는데 육각형 아키텍쳐를 사용하신거였다. 면접 때 왜 사용하셨고 어떤 부분을 고민하셨는지 들을 수 있었는데 아주 재밌는 시간이었다.

솔직히 깊이 검토해보지 않았고 사용경험도 없어서 어떤 장점을 누릴 수 있을 지 알순 없지만 한번쯤 사용해 보는것도 식견을 늘리는데 좋아 보인다.

[https://blog.funhnc.com/entry/%EC%9C%A1%EA%B0%81%ED%98%95-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98Hexagonal-architecture](https://blog.funhnc.com/entry/%EC%9C%A1%EA%B0%81%ED%98%95-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98Hexagonal-architecture){: target="\_blank" }

[https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)](https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)){: target="\_blank" }
