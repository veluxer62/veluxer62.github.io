---
layout: single
title: "2021년 1월 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

지난 1월은 정말 다사다난했다. 현재 회사에 입사하면서 정해놨던 목표가 있었고 길지 않은 기간동안 현재 회사 생활을 하면서 목표했던 것을 이루기 위해서 한걸음씩 나아가려고 노력했다. 기술적으로 새로운 시도를 해보기 위해 없는 시간 짜내어가며 기존에 익숙한 개발 방법이 아닌 새로운 개발 방법을 사용하여 개발을 하여 어플리케이션을 만들어보기도 하였고, 내가 가진 지식을 동료 개발자들에게 전파해 주기위해 따로 시간을 내어 세션 자료를 만들고 발표를 하였다. 나름대로 뿌듯함을 느꼈었고 함께 개발지식을 늘려가며 회사에서 더 나은 시스템을 만들어보고자 꿈을 키웠었다. <br/>
하지만 이런 노력들은 회사에서 가지는 문화와 가치에 따라서 불필요해 질 수 있음을 느끼게 되었다. 물론 모두가 같은 마음을 가질 순 없을테지만 자신이 가지고 있는 목표가 합리적이지 않은 이유로 변경되거나 중단될뻔한 위기를 여러번 겪고나서 애써 외면하던 현재의 문제에 대해 다시한번 생각하게 되었다. <br/>
결국 [인디언 옥수수](http://blog.naver.com/PostView.nhn?blogId=sokki77&logNo=221697657626&categoryNo=7&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView){: target="\_blank" } 중에 지금 내가 가진 옥수수가 누가 봐도 작은 옥수수임을 인정하고 더 큰 옥수수를 찾기 위해 결정을 내렸다. 이곳에서 목표했던 성과는 모두다 이루지 못했지만 나름대로 새로운 시도를 해보았고, 새로운 것을 많이 배웠으며 무엇보다 너무 좋은 사람들을 만나고 소통할 수 있어서 7개월 전의 나의 선택에 후회는 하지 않는다.

아래는 1월 중 내가 배우거나 겪었던 이슈에 대해서 정리한 내용이다.

## SCTP

스트림 제어 전송 프로토콜이라는게 있다는 걸 알게 되었다. 무려 20년 전에 정의한 프로토콜이라는데 이제서야 알았다니 부끄럽지만 회사에서 동료가 SCTP에 대한 소개를 해주었고 알게되었다. 스트림 활용을 많이 하지 않고 있어서 아직 개념이 잘 와닿지 않지만 한번씩 눈에 띄면 어떻게 활용되는지 봐둬야 겠다.

참고 자료 : [SCTP 표준기술 동향](https://protocol.knu.ac.kr/tech/CPL-TR-04-02.pdf){: target="\_blank" }

## bull

Node.js 서버에서 메시지 지연발송 기능을 구현하기 위해 자바스크립트 라이브러리 중 Redis를 이용한 Delayed Queue 기능을 사용할 수 잇는 [bull](https://github.com/OptimalBits/bull){: target="\_blank" }이라는 라이브러리를 사용해 보았다. Active MQ를 이용해서 구현해 보려고 하였는데 javascipt 라이브러리에 익숙하지 않다보니 생각보다 구현코드가 깔끔하지 않았고 그나마 괜찮았던 라이브러리가 bull이라서 사용해 보았는데 나쁘지 않았다.

## GraalVM

[GraalVM](https://www.graalvm.org/){: target="\_blank" }은 OpenJDK처럼 컴파일러이다. 어플리케이션의 성능증가, 컴파일 시간 향상 다양한 언어 지원등을 지원하는데 Spring, 트위터, 오라클 등에서도 지원하는거 같으니 한번 써봐야겠다.

## 몽고 세션 정리

- MongoDB의 Document는 16M의 크기 제한이 있다.
- 인덱스를 잘못 걸면 안건 것보다 쿼리 성능이 더 안좋아 질 수 있다.
- 인덱스 적용 룰
  - and인경우 카디너리티가 높은 인덱스를 수행 후 나머지는 filter를 하는 방식
    ```
    {$and: [{a: xxx}, {b: xxx}, {c: xxx}]}

    createIndex(a) <- 중요
    createIndex(b)
    createIndex(c)
    ```

  - or인 경우 걸려있는 모든 인덱스를 태운다.
    ```
    {$and: [{a: xxx}, {b: xxx}, {c: xxx}]}
    createIndex(a) <- 중요
    createIndex(b) <- 중요
    createIndex(c) <- 중요
    ```

  - 순정렬 인덱스와 역정렬 인덱스를 2개 만들 필요 없다. 하나만 만들어도 동일한 성능을 보이기 때문이다.
  
  - E-S-R rule
    ```
    a. Equality
    b. Sort
    c. Range
    ```
    위와 같은 순서로 인덱스를 주면 성능적으로 좋다.

- 몽고 노드에는 세컨더리 분석용 노드가 있다.
- Aggregation 제한조건
  - 메모리 제약이 있다.
    `aggregate({$match}, {$addFields}, {$project})` 이런 식으로 하면 문제 없음
  - blocking sort
    - `aggregate({$match}, {$addFields}, {$group}, {$sort})`
    - 100MB 제한이 있다.
    - allowDiskUse()를 쓰면 회피 할 수 있으나 disk io가 증가할 수 있음

- 인덱스 생성 방법
  - createIndex : 전 노드가 동시에 인덱스 생성
  - cxreateIndex-Rolling : Secondary First -> Role Change -> 기존 Primary 인덱스 생성
    - compass에서는 없고 Atlias에만 있는 기능
    - Cold Cache: Role Change 할때 메모리 캐시가 사라짐. 순간적으로 I/O가 칠수 있음
      - 사전에 컬렉션 Select 하는 방식으로 성능 문제를 회피 할 수 있음.

- write conflict 케이스
  ```
  transaction session(A) vs transaction이 아닌 session(B) -> A에서 conflict

  transaction session(A) vs transaction session(B) -> 둘다 성공

  transaction 타임아웃은 기본적으로 60초 (Atlias는 변경 불가)
  ```

- ChangeStream
  ```
  coll.watch([{$match:{isSuccess: 'YES'}}])
  ```
  변경 이벤트를 받고 싶은경우 watch를 사용하면 된다.

- Audit(1 ~ 5분) - 추가비용
  - 해당 시간 내 이벤트를 데이터나 api로 받을 수 있다.
  - 100ms 이상 소요 쿼리는 로그에 남음. -> 쿼리, 플랜
  - update 쿼리만 하고싶은경우 $unset, $set

- Aggregation 시 주의사항
  ```
  aggregate(
    {$match}, <- 첫번째 match절만 index를 탄다
    {$addFields},
    {$project},
    {$match}
  )
  ```

  - 4.4버전부터는 모든 스테이지에서 plan을 볼 수 있다. 하지만 현재는 안됨(소요시간, 다큐먼트리턴갯수)

- 쓸모없는 인덱스도 성능에 영향을 미친다.
  ```
  {partialIndex: {$exist: d}}
  ```
  - 존재하지 않거나 거의 없는 필드에 대한 인덱스를 주고싶은경우 좋음

- 메모리
  - 64GB 메모리를 쓴다면
    - 32GB : Atlas WT Cache (80% read only cache, 5% dirty cache) : 비압축
    - 32GB : OS FS Cache + Session cache(세션 1MB) + .... : 압축 (성능차이는 10%내외)

- getmore를 사용하는 케이스
  - 커서에서 fetch하는 애
  - 루신엔진을 사용하는 경우
  - aggregation 파이프라인

## Anti Corruption Layer

[부패방지계층](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/anti-corruption-layer){: target="\_blank" }이란 예를 들어 모놀리식을 MSA로 전환 시 부득이하게 모놀리식 API와 MSA API로 모두 이전이 불가능한 경우에 중간에 레이어를 둠으로써 MSA의 설계를 더럽히지 않으면서 모놀리식과 공존할 수 있도록 설계하는 패턴을 말한다.

## Kotlin에서 JsonCreator 사용 예시

```kotlin
enum class MessageResultCode(private val code: String) {
    KAKAO_SUCCESS("1000"),
    KAKAO_UNKNOWN_NUMBER("4001"),
    KAKAO_NOT_SUPPORTED_NUMBER("4002"),
    KAKAO_ERROR("9999"),
    LMS_SUCCESS("R000"),
    LMS_UNKNOWN_NUMBER("R500"),
    LMS_BLOCKED_NUMBER("R501"),
    LMS_ERROR("R502");

    companion object {
        @JsonCreator
        @JvmStatic
        fun fromCode(code: String): MessageResultCode {
            return values().first { it.code == code }
        }
    }
}
```


## Kotlin JPA Entity 클래스 사용

Kotlin JPA 사용시 Entity의 setter를 외부에서 사용하지 못하도록 하는 방법에 대한 고민이 있었다. Entity는 기본적으로 mutable 변수를 써야하므로 val를 쓰는것은 추천하지 않기 때문에 var를 써야한다.

처음엔 아래와 같이 코드를 작성해 봤는데 hibernate에서 오류가 발생함을 볼 수 있었다.

```kotlin
@Entity
class MessageHistory(
    @Id
    val id: UUID = UUID.randomUUID(),

    phoneNumber: String,
) {
    @Column
    final var phoneNumber: String = phoneNumber
        private set

    fun test() {
        phoneNumber = "ddd"
    }
}
```

```
org.hibernate.HibernateException: Getter methods of lazy classes cannot be final: com.service.demo.MessageHistory#getPhoneNumber
	at org.hibernate.proxy.pojo.ProxyFactoryHelper.validateGetterSetterMethodProxyability(ProxyFactoryHelper.java:96) ~[hibernate-core-5.4.23.Final.jar:5.4.23.Final]
	at org.hibernate.proxy.pojo.ProxyFactoryHelper.validateProxyability(ProxyFactoryHelper.java:88) ~[hibernate-core-5.4.23.Final.jar:5.4.23.Final]
	at org.hibernate.tuple.entity.PojoEntityTuplizer.buildProxyFactory(PojoEntityTuplizer.java:100) ~[hibernate-core-5.4.23.Final.jar:5.4.23.Final]
	at org.hibernate.tuple.entity.AbstractEntityTuplizer.<init>(AbstractEntityTuplizer.java:160) ~[hibernate-core-5.4.23.Final.jar:5.4.23.Final]
	at org.hibernate.tuple.entity.PojoEntityTuplizer.<init>(PojoEntityTuplizer.java:53) ~[hibernate-core-5.4.23.Final.jar:5.4.23.Final]
	at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:na]
  ... 이하생략
```

JPA가 내부적으로 Entity의 proxy를 생성하여 Entity를 상속하여 사용하는 것처럼 보이는데 이때 final 변수는 상속을 할 수 없고, private setter도 상속으로 사용할 수 없는 문제가 있다.

그래서 protected setter를 사용하도록 변경하였고 final 키워드도 제거할 수 있게 되었다.

고민 결과 최선의 코드는 아래와 같다.

```kotlin
@Entity
class MessageHistory(
    promotionId: UUID,
    phoneNumber: String,
    content: String,
) {
    @Id
    var id: UUID = UUID.randomUUID()
        protected set

    @Column(nullable = false)
    var promotionId = promotionId
        protected set

    @Column(nullable = false)
    var createdAt: ZonedDateTime = ZonedDateTime.now()
        protected set

    @Column
    @Enumerated(STRING)
    var provider: Provider? = null
        protected set

    @Column(nullable = false)
    @Enumerated(STRING)
    var status = REQUESTED
        protected set

    @Column(nullable = false)
    var phoneNumber = phoneNumber
        protected set

    @Column(length = 4000)
    var content = content
        protected set

    @Column
    var sentAt: ZonedDateTime? = null
        protected set

    @Column
    var responseAt: ZonedDateTime? = null
        protected set

    @Column
    @Enumerated(STRING)
    var resultCode: MessageResultCode? = null
        protected set

    fun changeStatusToSend(provider: Provider) {
        if (status != REQUESTED) {
            throw DomainException("이미 전송된 메시지 이력은 전송상태로 변경할 수 없습니다.")
        }

        this.provider = provider
        this.status = SENT
        this.sentAt = ZonedDateTime.now()
    }

    fun putResponseResult(code: MessageResultCode) {
        if (status != SENT) {
            throw DomainException("전송되지 않은 메시지는 전송결과를 저장할 수 없습니다.")
        }

        status = code.getHistoryStatus()
        responseAt = ZonedDateTime.now()
        resultCode = code
    }
}
```

추가로 kotlin에서 지연 로딩을 사용하려면 gradle에 아래와 같이 설정해주어야 한다.

```kotlin
allOpen {
    annotation("javax.persistence.Entity")
    annotation("javax.persistence.MappedSuperclass")
    annotation("javax.persistence.Embeddable")
}
noArg {
    annotation("javax.persistence.Entity")
    annotation("javax.persistence.MappedSuperclass")
    annotation("javax.persistence.Embeddable")
}
```

## JPA open in view

JPA에서 open in view 설정을 할 수 있는데 만약 true로 설정한다면 영속성 컨텍스트가 트랜잭션 범위를 넘어선 레이어까지 살아 있게끔 할 수 있다. false인 경우에는 트랜잭션 범위까지만 살아 있다. 기본값은 true이다. 해당 설정의 장점은 지연로딩을 트랜잭션의 범위 밖 (예를들면 서비스에 트랜잭션이 걸려 있다면 컨트롤러까지 지연로딩을 미룰 수 있다.)까지 미룰 수 있기 때문에 성능적으로 이점을 가져갈 수 있다.

## JMS 리스너에서 동시성 처리 설정

```kotlin
@JmsListener(destination = MESSAGE_SEND_REQUEST, concurrency = "3-10")
```

## Kafka topic convention

Topic 설정 시 컨벤션을 어떻게 할지 고민될때 참고하면 좋을 사이트.

[https://devshawn.com/blog/apache-kafka-topic-naming-conventions/](https://devshawn.com/blog/apache-kafka-topic-naming-conventions/){: target="\_blank" }

## 대학 수학 지식 관련 PDF

수학지식 관련해서 보면 좋을만한 PDF.

[https://venhance.github.io/napkin/Napkin.pdf](https://venhance.github.io/napkin/Napkin.pdf){: target="\_blank" }

## ENUM을 json으로 serialize할때 소문자로 표시 방법

```kotlin
enum class Provider {
    KAKAO,
    LMS;

    @JsonValue
    fun getKey() = this.name.toLowerCase()
}
```

## JPA Entity Graph 사용시 주의할점

```kotlin
interface DocumentRepository : JpaRepository<Document, Long> {
    // 1.
    @EntityGraph(attributePaths = ["approvalLines"])
    fun findByDrafter(drafter: String): List<Document>

    // 2.
    fun findByApprovalLinesAssignment(assignment: String): List<Document>

    // 3.
    @EntityGraph(attributePaths = ["approvalLines"])
    fun findByDrafterOrApprovalLinesAssignment(drafter: String, assignment: String): List<Document>
}
```

1. Document Entity에 drafter정보가 있으므로 EntityGraph를 사용해도 문제없이 쿼리가 된다.
2. Document와 ApprovalLine이 1:N 관계인데 ApprovalLine의 assignment가 입력한 값과 동일한 Document를 조회하는 함수이다. 이때 EntityGraph를 사용하는 경우 assignment가 일치하는 모든 Document를 조회하지만 ApprovalLine도 assignment값이 일치하는 값만 가져오므로 EntityGraph를 사용하지 말아야 한다.
3. Document의 drafter값과 ApproveLine의 assignment값을 모두 포함하는 Document를 모두 조회하는 쿼리인데 이때 EntityGraph를 사용하지 않으면 중복된 값이 조회되는 것을 확인할 수 있다. EntityGraph를 사용하면 중복을 제거하여 조회결과를 반환하므로 해당 쿼리는 EntityGraph를 사용해주어야 한다.

## Sonaqube

프로젝트의 코드에 대한 정적 분석 도구로 [소나큐브](https://www.sonarqube.org/){: target="\_blank" }라는게 있다.

소나큐브 대시보드

```bash
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
```

에저 데브옵스의 파이프라인에도 적용할 수 있다.

[https://www.sonarqube.org/microsoft-azure-devops-integration/](https://www.sonarqube.org/microsoft-azure-devops-integration/){: target="\_blank" }

인텔리제이에서 연동해서도 사용할 수 있다.

[https://shinsunyoung.tistory.com/64](https://shinsunyoung.tistory.com/64){: target="\_blank" }


소나큐브 적용 방법

```kotlin
plugins {
  id("org.sonarqube") version "3.1.1"
}
```

```kotlin
buildscript {
  repositories {
    maven {
      url = uri("https://plugins.gradle.org/m2/")
    }
  }
  dependencies {
    classpath("org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:3.1.1")
  }
}

apply(plugin = "org.sonarqube")
```

## wasp zap

[보안 감사 도구](https://www.zaproxy.org/){: target="\_blank" }

## EnableAspectJAutoProxy

인터페이스 구현체로 빈을 주입하는 경우 java dynamic proxy가 생긴다. 클래스를 직접 주입하면 CGLIB이 동작한다.
CGLIB이 성능적으로 뛰어나다 그이유는 컴파일 단계에서 프록시 파일을 생성하기 때문이다.
만약 인터페이스 구현체도 CGLIB을 사용하도록 하려면 `@EnableAspectJAutoProxy(proxyTargetClass = true)` 을 사용하자.
근데 설정없이 인터페이스로 주입해보니 CGLIB으로 프록시가 생성되네???...