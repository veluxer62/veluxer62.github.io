---
layout: single
title: "2020년 12월 1주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

키트 재고 관리 기능이 1차적으로 완료되었다. 아직 목록에서 검색기능과 추가적인 요구사항이 조금 남아 있지만 운영서버에서 테스트를 할 수 있는 환경에 두고 QA를 완료했다. 앞으로 남은 작업은 기존 상품에 키트를 연결 시키는 것과 주문 시 재고 차감, 반품과 관련한 기능이 남아 있는 상태이다. 1월 안으로 작업을 완료해서 어서 빨리 운영환경에 배포할 수 있도록 집중해야겠다.

1차 작업이 끝나고 Azure Devops에서 사용할 파이프라인 구축을 하는데 한주를 다 쓴것 같다. 해야할 작업이 산더민데 갑자기 파이프라인 구축에 꽃혀서 많은 시간을 사용했는데, 많이 배울 수 있었으면서 한편으로는 우선순위가 높지 않은 작업에 너무 많은 시간을 쓴게 아닌가 싶기도 하다. 주말동안에 사내 공유할 문서도 완성했으니 그래도 뿌듯하긴 하다.

아래는 지난 주간 겪었던 이슈 또는 배운 내용을 정리한 것이다.

## 일급 컬랙션

재고 이력관리를 개발하면서 일급 컬랙션을 다루어 보아야 겠다고 생각하게 되었다. 일급 컬랙션은 쉽게 말해 컬랙션 속성만을 가진 클래스를 말하는데, 사용했을때 여러 장점을 누릴 수 있다. 일급 컬랙션 구현 내용은 블로그로 작성할 계획이다.

참고 사이트 : [https://velog.io/@lil_bear/first-class-collection](https://velog.io/@lil_bear/first-class-collection){: target="\_blank" }

## JPA Lock

JPA에서 사용하는 Lock 어노테이션은 Service 객체에서는 적용되지 않는다. 반드시 Repository의 함수에서 사용해야 한다.

## Lock Scope

재고관리 기능을 개발하면서 부모 Entity에서 Lock을 사용하는 경우 자식 Entity에까지 Lock이 전파되도록 구현하고 싶었다.
JPA에서 해당 기능을 지원하는지 찾아보니 아래와 같이 어노테이션을 적용할 수 있다는 글을 보게 되었다.

```kotlin
@QueryHints(QueryHint(name = "javax.persistence.lock.scope", value = "EXTENDED"))
```

하지만 실제로 사용해보니 아래와 같은 오류가 발생하는 것을 볼 수 있었다.

```
HHH000121: Ignoring unrecognized query hint [javax.persistence.lock.scope]
```

그래서 문서를 좀더 찾아보니 아직 해당 기능은 JPA에서 지원하지 않는다는 걸 알 수 있었고, 이슈로 등록되어 있는 것을 확인할 수 있었다.

이슈 링크 : [https://medium.com/@IEnoobong/locking-entity-foreign-key-in-jpa-dbda9c747d07
https://hibernate.atlassian.net/browse/HHH-9636](https://medium.com/@IEnoobong/locking-entity-foreign-key-in-jpa-dbda9c747d07
https://hibernate.atlassian.net/browse/HHH-9636){: target="\_blank" }

결국 근본적으로 해결할순 없었고 직접 구현해서 해결하였다. 다만 아래 코드와 같이 하면 자식 Entity를 2번 조회하는 단점이 있다 ㅠ

```kotlin
@Transactional(readOnly = true)
override fun getPack(id: Long): KitPack {
    val kitPackEntity = entityManager.find(
        KitPackEntity::class.java, id,
        PESSIMISTIC_WRITE,
        mapOf("javax.persistence.lock.timeout" to "5")
    )

    if (kitPackEntity == null || kitPackEntity.deletedAt != null) {
        throw NoSuchElementException("No value present")
    }

    /**
      * Hibernate에서 아직 PessimisticLockScope.EXTENDED를 제공해주지 않기 때문에 Kit Entity에게 Lock이 전파되지 않는다.
      * Lock을 전파시키기 위해서 NamedQuery를 이용해 Kit Entity를 Pessimistic Write 모드로 재조회 한 후 Pack Entity에 삽입해 준다.
      *
      * Pack Entity에서 자동으로 Kit Entity를 조회하기 때문에 조회 쿼리가 2번 실행되는 비효율이 있기는 하지만 Row 수가 많지 않을 것이고
      * 성능상 큰 차이가 나지 않을 것이므로 비효율적인 부분은 수용하도록 한다.
      *
      * PessimisticLockScope 이슈는 https://hibernate.atlassian.net/browse/HHH-9636 이슈카드에서 확인 가능하다.
      * 해당 이슈가 해결된다면 이 코드를 변경하면 좋을 것 같다.
      */
    val kitEntities =
        entityManager.createNamedQuery("findAllByPackIdAndDeletedAtIsNullForUpdate", KitEntity::class.java)
            .setParameter("kitPackId", id)
            .resultList

    kitPackEntity.kits.replaceAll { kitEntities.first { kitEntity -> kitEntity.id == it.id } }

    return kitPackEntity.toDomain()
}
```

## ifkakao

사내 개발자 회동 시간에 [ifkakao](https://if.kakao.com){: target="\_blank" }를 보는 시간을 가졌다.

오랜만에 Clickhouse에 대한 발표도 보게 되었고, Jooq를 이용한 정산시스템 개발, kotest 활용등등 볼만한 내용이 많았다.

약간 억지로 발표하는 듯한 느낌을 준 발표도 있긴했는데, 다음에 또 세션이 열리면 챙겨서 봐야겠다.

## 카프카 공부 사이트

카프카를 배우기 좋은 사이트가 있어서 기록해둔다.

[https://developer.confluent.io/learn-kafka/](https://developer.confluent.io/learn-kafka/){: target="\_blank" }

## Pessimistic Read vs Pessimistic Write

Pessimistic Read는 단순히 dirty read와 non-repeatable read를 방지하는 용도로 사용되는 Lock이다. 만약 데이터를 변경해야 한다면 Pessimistic Write를 사용해야 한다.

참고 링크 : [https://stackoverflow.com/questions/1657124/whats-the-difference-between-pessimistic-read-and-pessimistic-write-in-jpa](https://stackoverflow.com/questions/1657124/whats-the-difference-between-pessimistic-read-and-pessimistic-write-in-jpa){: target="\_blank" }

## http 3

매번 봐야지 봐야지 하면서 못보고 있던 http 3에 대해서 찾아보게 되었다. (사실 깊이있게 보지도 않았다.)

큰 특징은 QUIC라는 UDP 기반 프로토콜을 사용한다는 것이고 UDP 기반이기 때문에 TCP보다 좋은 성능을 가지면서 안정성으 함께 가져가는 프로토콜로 발전했다는 것이었다.

참고 링크 : [https://ykarma1996.tistory.com/86](https://ykarma1996.tistory.com/86){: target="\_blank" }