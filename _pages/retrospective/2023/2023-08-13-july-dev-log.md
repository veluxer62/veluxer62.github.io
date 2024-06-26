---
layout: single
title: "2023년 7월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2023-july-dev-log/
---

아무래도 역마살이 낀게 분명하다. 회사가 이사를 계획중이라는 소식을 들었기 때문이다. 지금까지 회사를 다니면서 이직때문에 옮기는 경우를 제외하고도 인원이 늘어나 사무실을 옮기거나 합병 등등 여러가지 이유로 1년에 한번꼴로 사무실을 옮기는 중이다. 지금 회사에서도 현재 2년이 넘는 기간동안 다니고 있는데 입사하고 1년이 넘어가는 시점에 사무실을 옮겼었다. 당시에는 도도 포인트 서비스를 가지고 있던 상황이라 큰 사무실로 이사를 했었는데, 도도 포인트 인원들이 빠진 현재는 사무실이 불필요하게 커서 좀 더 작은 사무실로 이사를 계획중이라고 한다. 일단 건물주랑 조율중이라고는 하는데 너무 좁은 사무실로만 안갔으면 좋겠다.

<br/>

올해는 다른 해에 비해 봄이 길어서 6월까지도 자전거를 타고 출근할 수 있을 정도로 날이 선선했는데 7월이 되니 갑자기 엄청나게 더워져 어느해보다 더운 여름을 나고 있다. 샤워시설이 없다보니 이제는 땀냄새가 날까봐 자전거를 타지도 못한다. 올해 그랜드 슬램을 달성하려면 가을에 부지런히 자전거를 타야겠다. 그전에 코스를 미리 계획해둬야겠다.

<br/>

운영중인 서버의 Spring Boot 버전을 2.x.x에서 3.x.x로 업그레이드 하였다. 사실 Spring Boot 3.0.0 버전이 출시된지 얼마되지 않았던 시점에 업그레이드를 시도했지만 Spring Cloud쪽에서 아직 대응이 되어있지 않아 업그레이드를 포기했어야 했다. 명세표 서비스와 관련된 코드를 제거함으로써 사용하지 않는 의존성들을 제거할 수 있었는데, 이번 기회에 다시 Spring Boot의 버전을 업그레이드할 수 있는 기회다 싶어서 업그레이드를 결심하게 되었다.

3.1.1버전으로 버전이 올라가면서 3.0.0에 비해서 상당히 안정되었고 관련된 많은 라이브러리들도 대응이되어있는 상황이라 다소 이슈는 있었지만 그래도 무사히 업그레이드를 마칠 수 있었다. 그래서 이번달 이슈들은 스프링 버전을 업그레이드를 하면서 겪었던 이슈들이 다수 포함되어 있다.

<br/>

아래는 7월동안 정리한 이슈 내용들이다.

## Spring Boot 3 버전 업그레이드 관련 이슈 - `PostgreSQLEnumType`

Spring Boot 3으로 버전을 업그레이드하면서 Hibernate 버전이 6으로 변경되었다. 이로인해 많은 이슈들이 있었는데 그 첫번째 이슈는 바로 `PostgreSQLEnumType`이다.

Hibernate5에서는 PosgreSQL의 Enum 타입에 대해 Entity로 매핑할 때 아래와 같이 정의하면 되었다.

```kotlin
@Column
@Type(type = "com.vladmihalcea.hibernate.type.basic.PostgreSQLEnumType")
@Enumerated(EnumType.STRING)
var gender: Gender? = gender
    protected set
```

하지만 Hibernate6으로 업그레이드 되면서 아래와 같이 정의방식이 변경되었다.

```kotlin
@Column
@Type(PostgreSQLEnumType::class)
@Enumerated(EnumType.STRING)
var gender: Gender? = gender
    protected set
```

다만 이렇게만 변경하면 아래와 같이 오류가 발생하게 되는데

```
Caused by: java.lang.NullPointerException: Cannot invoke "org.hibernate.boot.spi.MetadataBuildingContext.getMetadataCollector()" because "this.metadataBuildingContext" is null
	at org.hibernate.type.spi.TypeConfiguration$Scope.getDialect(TypeConfiguration.java:459)
	at org.hibernate.type.EnumType$LocalJdbcTypeIndicators.getDialect(EnumType.java:516)
	at org.hibernate.type.descriptor.jdbc.VarcharJdbcType.shouldUseMaterializedLob(VarcharJdbcType.java:90)
	at org.hibernate.type.descriptor.jdbc.VarcharJdbcType.resolveIndicatedType(VarcharJdbcType.java:79)
	at org.hibernate.type.descriptor.java.BasicJavaType.getRecommendedJdbcType(BasicJavaType.java:33)
	at org.hibernate.type.descriptor.java.StringJavaType.getRecommendedJdbcType(StringJavaType.java:56)
	at org.hibernate.type.EnumType.configureUsingReader(EnumType.java:180)
	at org.hibernate.type.EnumType.setParameterValues(EnumType.java:142)
	at com.vladmihalcea.hibernate.type.basic.PostgreSQLEnumType.setParameterValues(PostgreSQLEnumType.java:46)
	at org.hibernate.mapping.MappingHelper.injectParameters(MappingHelper.java:109)
	at org.hibernate.mapping.BasicValue.setExplicitCustomType(BasicValue.java:836)
	at org.hibernate.boot.model.internal.BasicValueBinder.fillSimpleValue(BasicValueBinder.java:1363)
	at org.hibernate.boot.model.internal.SetBasicValueTypeSecondPass.doSecondPass(SetBasicValueTypeSecondPass.java:27)
	at org.hibernate.boot.internal.InFlightMetadataCollectorImpl.processSecondPasses(InFlightMetadataCollectorImpl.java:1857)
	at org.hibernate.boot.internal.InFlightMetadataCollectorImpl.processSecondPasses(InFlightMetadataCollectorImpl.java:1803)
	at org.hibernate.boot.model.process.spi.MetadataBuildingProcess.complete(MetadataBuildingProcess.java:328)
```

[hypersistence-utils](https://github.com/vladmihalcea/hypersistence-utils){: target="\_blank" } 이슈([https://github.com/vladmihalcea/hypersistence-utils/issues/519](https://github.com/vladmihalcea/hypersistence-utils/issues/519){: target="\_blank" })를 보면 `com.vladmihalcea:hibernate-types-XX`가 `io.hypersistence:hypersistence-utils-hibernate-XX`로 바뀌었다는 것을 알 수 있다.

> The project name was changed from Hibernate Types to Hypersistence Utils because the scope of the project is much broader now, offering Spring utilities as well.

의존성을 `io.hypersistence:hypersistence-utils-hibernate-62`로 바꾸니 해결되는듯 했으나 아래와 같이 다른에러가 발생하는 것을 볼 수 있었다.

```
Caused by: org.hibernate.HibernateException: No type mapping for org.hibernate.type.SqlTypes code: 1111 (OTHER)
	at org.hibernate.type.descriptor.sql.spi.DdlTypeRegistry.getTypeName(DdlTypeRegistry.java:205)
	at org.hibernate.type.descriptor.sql.spi.DdlTypeRegistry.getTypeName(DdlTypeRegistry.java:184)
	at org.hibernate.mapping.Column.getSqlTypeName(Column.java:277)
```

원인을 찾아보니 이슈[https://github.com/vladmihalcea/hypersistence-utils/issues/625](https://github.com/vladmihalcea/hypersistence-utils/issues/625){: target="\_blank" }에서 `columnDefinition`를 정의해주면 된다고 가이드해주고 있어서 아래와 같이 `columnDefinition`를 추가해주니 해결되었다. `columnDefinition`의 값은 PostgreSQL에 정의한 ENUM의 이름을 기입하면 된다.

```kotlin
@Column(columnDefinition = "gender")
@Type(PostgreSQLEnumType::class)
@Enumerated(EnumType.STRING)
var gender: Gender? = gender
    protected set
```

ps. DB의 ENUM은 꼭 필요한 경우가 아니라면 사용하지 말자...

## Spring Boot 3 버전 업그레이드 관련 이슈 - `ActiveMQ` VM 모드

테스트코드에서 `ActiveMQ`를 VM모드로 사용중이다. 그런데 버전을 업그레이드하니 URL 문자열 파싱 에러가 발생한다.

```yml
spring:
  activemq:
    in-memory: true
    broker-url: vm://localhost?broker.persistent=false&broker.useShutdownHook=false
```

```
Caused by: org.springframework.context.ApplicationContextException: Failed to start bean 'org.springframework.jms.listener.SimpleMessageListenerContainer#0'
Caused by: org.springframework.jms.UncategorizedJmsException: Uncategorized exception occurred during JMS processing
Caused by: jakarta.jms.JMSException: Could not create Transport. Reason: java.io.IOException: Transport scheme NOT recognized: [vm]
Caused by: java.io.IOException: Transport scheme NOT recognized: [vm]
Caused by: java.io.IOException: Could not find factory class for resource: META-INF/services/org/apache/activemq/transport/vm
```

원인을 찾아보니 Spring Boot 3에서 사용하는 ActiveMQ 모듈에 VM 모드를 지원 종료한 것으로 보인다. ActiveMQ 5.19.0 버전에 다시 나온다는 얘기도 있다.

- 출처: [https://github.com/spring-projects/spring-boot/issues/34011](https://github.com/spring-projects/spring-boot/issues/34011){: target="\_blank" }

## Spring Boot 3 버전 업그레이드 관련 이슈 - `QueryDSL` Like 이슈

QueryDSL에서 like를 쓰면 아래와 같이 syntax 에러가 발생한다.

```kotlin
select(store.id)
    .from(store)
    .where(store.name.like("%$storeName%")),
```

```
Hibernate: select count(v1_0.id) from vendor v1_0 where v1_0.reg_num like ? escape !
2023-07-03T16:50:52.132+09:00  WARN 32056 --- [pool-1-thread-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : SQL Error: 0, SQLState: 42601
2023-07-03T16:50:52.132+09:00 ERROR 32056 --- [pool-1-thread-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : ERROR: syntax error at end of input
  Position: 75

JDBC exception executing SQL [select count(v1_0.id) from vendor v1_0 where v1_0.reg_num like ? escape !] [ERROR: syntax error at end of input
org.springframework.dao.InvalidDataAccessResourceUsageException: JDBC exception executing SQL [select count(v1_0.id) from vendor v1_0 where v1_0.reg_num like ? escape !] [ERROR: syntax error at end of input
  Position: 75] [n/a]; SQL [n/a]
```

Hibernate6으로 변경되면서 PostgreSQL의 방언도 바뀐듯한데 아마 QueryDSL에서 대응이 되지 않은것 같다라는 예상을 해본다. (이슈를 검색해보아도 오래전 글만 발견되지 알맞는 이슈가 발견되지 않았다.)

`contain`함수를 사용해보았지만 동일한 현상이 재현되어서 아래와 같이 `likeIgnoreCase`로 임시로 조치하였다. 동일한 이슈를 겪으시는 분들이 있는지 좀더 살펴봐야겠다.

```kotlin
select(store.id)
    .from(store)
    .where(store.name.likeIgnoreCase("%$storeName%")),
```

## Spring Boot 3 버전 업그레이드 관련 이슈 - PostgreSQL 함수

hibernate 6으로 업그레이드하고 나서 아래 함수들이 정상 동작하지 않았다.

```kotlin
fun <A, E> ArrayPath<A, E>.contains(element: E): BooleanExpression {
    return Expressions.booleanTemplate(
        "{0} = function('ANY', {1})",
        element,
        this,
    )
}

val <A, E> ArrayPath<A, E>.isEmpty: BooleanExpression
    get() = Expressions.booleanTemplate("{0} = '{}'", this)
```

이것또한 QueryDSL의 문제인지 PostgreSQL의 방언 문제인지 정확한 원인을 찾진 못하였다. 일단 임시로 아래와 같이 문제를 해결하였다.

```kotlin
fun <A, E> ArrayPath<A, E>.contains(element: E): BooleanExpression {
    return Expressions.booleanTemplate(
        "ARRAY_POSITION({0}, {1}) IS NOT NULL",
        this,
        element,
    )
}

val <A, E> ArrayPath<A, E>.isEmpty: BooleanExpression
    get() = Expressions.booleanTemplate("COALESCE(array_length({0}, 1), 0) = 0", this)
```

## Spring Boot 3 버전 업그레이드 관련 이슈 - URL trailing slash

Spring boot 3으로 업그레이드 하면서 이제 URL에 대한 trailing slash를 명시적으로 구분하게되었다. 언어전환을 하면서 파이선 프로젝트에서 행해지던 관행인 trailing slash를 더이상 사용하지 않고 있었는데 `https://host/graphql/`를 프론트에서 사용하고 있었나 보다. DGS Framework의 코드를 보면 `/graphql` 경로를 사용하고 있는 것을 볼 수 있다.

```kotlin
@ConfigurationProperties(prefix = "dgs.graphql")
@Suppress("ConfigurationProperties")
data class DgsWebMvcConfigurationProperties(
    /** Path to the GraphQL endpoint without trailing slash. */
    @DefaultValue("/graphql") var path: String = "/graphql",
)


@RequestMapping(
    "#{@'dgs.graphql-com.netflix.graphql.dgs.webmvc.autoconfigure.DgsWebMvcConfigurationProperties'.path}",
    produces = [MediaType.APPLICATION_JSON_VALUE]
)
fun graphql(
    @RequestBody body: ByteArray?,
    @RequestParam fileParams: Map<String, MultipartFile>?,
    @RequestParam(name = "operations") operation: String?,
    @RequestParam(name = "map") mapParam: String?,
    @RequestHeader headers: HttpHeaders,
    webRequest: WebRequest
): ResponseEntity<Any> {
}
```

그래서 Graphql API가 호출되지 않는 이슈가 있었는데, 어차피 맞춰주었어야했던 경로라 클라이언트에서 맞춰주기로 하였다.

- 출처: [https://www.baeldung.com/spring-boot-3-url-matching](https://www.baeldung.com/spring-boot-3-url-matching){: target="\_blank" }

## Spring Boot 3 버전 업그레이드 관련 이슈 - 잘못된 OneToOne

hibernate 6으로 업그레이드 되면서 QueryDSL에서 잘못된 `@OneToOne` 사용에 대한 연관관계 로딩을 못하는 이슈가 발생했다.

여기서 잘못된 `@OneToOne`이란 `@OneToMany` - `@ManyToOne`으로 선언해야 할 테이블 관계를 억지로 `@OneToOne`으로 선언해 사용하는 경우인데, One to One 관계를 정의할 때 부모 Entity에 자식 Entity ID를 두는 형태로 해야함에도 부모 Entity에 불필요한 컬럼을 두고싶지 않아 자식 Entity에서 부모 Entity ID를 가지도록한 것이었다.

일단 잘못된 사용방식이기 때문에 고쳐서 해결했지만....QueryDSL이 더이상 유지보수를 안하고 있다는 점에서 리스크가 점점 커지는것 같다.

## Spring Boot 3 버전 업그레이드 관련 이슈 - Password Encoder

Spring Boot 3으로 업그레이드하면서 Spring Security의 버전이 6으로 변경되었다. 그러면서 기본 Password Encoder의 알고리즘이 변경되었다.
그래서 사용자가 로그인 시 비밀번호가 일치하지 않는다는 상황이 펼쳐질것이 예상되었다. 다행히 소셜 로그인의 경우에는 대응할 필요가 없었지만 비밀번호를 사용하는 계정의 경우에는 비밀번호를 새롭게 입력할 수 있도록 유도하였다.

## Spring Boot 3 버전 업그레이드 관련 이슈 - Timezone

hibernate6으로 올라가면서 JPA에서 타임존을 기본적으로 UTC로 강제 설정한다. 서버에서 아래와 같이 기본 타임존을 설정해주어도 DB에서 조회한 값을 매핑한 Entity의 날짜 데이터는 UTC로 설정됨을 볼 수 있었다.

```kotlin
@SpringBootApplication
class Application {
    @PostConstruct
    fun startUp() {
        TimeZone.setDefault(TimeZone.getTimeZone(SEOUL_ZONE_ID))
    }
}
```

보통의 경우에는 큰 문제가 되지 않는데 format을 통한 문자열로 시간을 표현해줄 때 의도치 않게 9시간이 빠진 시간을 볼 수 있다. 왜냐하면 foramt은 타임존을 무시하기 때문이다. 그래서 format을 사용할 때 KST로 변경해서 표현해주어야 한다.

- 출처: [https://github.com/hibernate/hibernate-orm/blob/6.0/migration-guide.adoc#instant-mapping-changes](https://github.com/hibernate/hibernate-orm/blob/6.0/migration-guide.adoc#instant-mapping-changes){: target="\_blank" }

## Spring Boot 3 버전 업그레이드 관련 이슈 - hibernate 6.2.5.final 버그

Spring Boot 3.1.1 버전에서 사용하는 Hibernate 6.2.5.final 버전에 몇가지 버그가 있다. 그 중 우리가 겪었던 주목할만한 이슈는 아래 두가지였다.

- 하이버네이트 6.2.5.Final 기준 lazy 로드와 배치로드에 버그가 있음
- 조회 조건에 any 연산자가 계속 추가되어 실행됨

그래서 hibernate 6.2.6.final로 강제로 바꿔주었다.

```
# gradle.properties
hibernate.version=6.2.6.final
```

- 출처: [https://in.relation.to/2023/06/30/hibernate-orm-626-final/](https://in.relation.to/2023/06/30/hibernate-orm-626-final/){: target="\_blank" }

## Spring Boot 3 버전 업그레이드 관련 이슈 - Composite Primary Key

Hibernate 6.2.6에서 Composit Primary Key를 사용할때 `EmbeddedId`나 `IdClass`를 정의하지 않고 사용하면 `default_batch_fetch_size`를 설정해도 N+1 문제가 발생한다.

그래서 만약 Composit Primary Key를 사용하려면 `IdClass`를 정의해서 사용하는게 좋다. 특히 IdClass는 Entity가 아니라 실제 매핑할 ID를 적어주어야 한다.

```kotlin
@Entity
@IdClass(OrderableProductTemplateProductAssocId::class)
class OrderableProductTemplateProductAssoc(
    orderableVendorProduct: OrderableVendorProduct,
    orderableProductTemplate: OrderableProductTemplate,
) : Serializable {
    init {
        orderableVendorProduct.addOrderableProductTemplateAssoc(this)
    }

    @Id
    private val orderableVendorProductId: UUID = orderableVendorProduct.id

    @Id
    private val orderableProductTemplateId: UUID = orderableProductTemplate.id

}

private class OrderableProductTemplateProductAssocId : Serializable {
    private lateinit var orderableVendorProductId: UUID
    private lateinit var orderableProductTemplateId: UUID
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false

        other as OrderableProductTemplateProductAssocId

        if (orderableVendorProductId != other.orderableVendorProductId) return false
        if (orderableProductTemplateId != other.orderableProductTemplateId) return false

        return true
    }

    override fun hashCode(): Int {
        var result = orderableVendorProductId.hashCode()
        result = 31 * result + orderableProductTemplateId.hashCode()
        return result
    }
}
```


## JPA Soft Delete

도메인의 요구사항에 따라서는 DB의 Row를 완전히 지우지 않고 Flag 처리하는 Soft Delete 처리를 해야하는 경우가 있다. 이번에도 해당 로직을 구현해야할 필요가 있어서 JPA에서 제공해주는 `@Where`를 사용해서 Soft Delete 처리를 해주어 보았는데 나름 괜찮게 동작했다. 특히 연관관계 조회시에도 wehre절이 잘 동작한다는 점이 마음에 들었는데, 수동으로 where절을 넣어줄때에는 단순 조회쿼리에는 잊어버리지 않고 잘 넣어주는데 다른 Entity와 join하는 경우 자주 잊어버리거나 쿼리가 복잡해지는 경우를 많이 겪었기 때문이다.

```kotlin
@Entity
@Where(clause = "deleted_at is null")
class User {
    @Column
    var deletedAt: OffsetDateTime? = null
        protected set
}
```

## 동적 LDAP 클라이언트 사용 시 주의사항

LDAP을 사용할때 동적으로 클라이언트를 설정해서 사용하려면 아래와 같이 `afterPropertiesSet`를 명시적으로 호출해주어야 한다. 그렇지 않으면 Client가 정상적으로 동작하지 않는다.

```kotlin
LdapContextSource ctxSrc = new LdapContextSource();
    ctxSrc.setUrl("ldap://<url>");
    ctxSrc.setBase("dc=example,dc=local");
    ctxSrc.setUserDn("<user>@example.local");
    ctxSrc.setPassword("<pass>");

ctxSrc.afterPropertiesSet(); // this method should be called.

LdapTemplate tmpl = new LdapTemplate(ctxSrc);
setLdapTemplate(tmpl);
```

## JIB 빌드시간 변경

JIB 이미지 빌드 시간을 현재로 바꾸려면 아래와 같이 하면된다.

```kotlin
jib {
    val ecrRegistry = System.getenv("REGISTRY")

    from {
        image = "amazoncorretto:17.0.7"
    }
    to {
        image = ecrRegistry
        tags = setOf(getBuildVersion())
    }
    container {
        jvmFlags = listOf(
            "-XX:InitialRAMPercentage=65.0",
            "-XX:MaxRAMPercentage=65.0",
        )
        creationTime.set("USE_CURRENT_TIMESTAMP") // <--- 여기 설정을 추가해주면 된다.
    }
}
```

- 참고: [https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin#container-closure](https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin#container-closure){: target="\_blank" }

## Kotlin으로 XML Marshalling시 이슈

kotlin 으로 마샬링 할때 `xmlElement` 속성이 매핑되지 않으면 `@field:XmlElement`로 정의해주자

- 출처: [https://stackoverflow.com/questions/47049867/xmlelement-does-not-work-when-used-in-kotlin](https://stackoverflow.com/questions/47049867/xmlelement-does-not-work-when-used-in-kotlin){: target="\_blank" }

## @TransactionalEventListener 이슈

주문서 일괄 점수 시 100개정도 접수를 하면 60개만 접수완료 메시지를 발생하는 이슈가 발생했다.
원인을 찾아보니 @TransactionalEventListener가 동시에 100개를 요청했을 때 60개만 전송하고 10개는 무시처리되는것이었다.

[블로그](https://velog.io/@byeongju/TransactionalEventListener%EB%A5%BC-%EB%8F%99%EA%B8%B0%EB%A1%9C-%EC%B2%98%EB%A6%AC%ED%95%A0-%EB%95%8C%EC%9D%98-%EB%B0%9C%EC%83%9D%EA%B0%80%EB%8A%A5%ED%95%9C-%EB%AC%B8%EC%A0%9C){: target="\_blank" }에서 힌트를 좀 얻었는데, 아무래도 DB Connection Pool이 모자라서 이벤스 수신이 되지 않았나 예상이 되기는 하다.

> ApplicationEventListener로 발행한 event를 @TransactionEventListener로 동기 처리를 할 때, 이벤트를 발행하는 쪽에서 Transaction이 commit 되었음에도 Connection은 놓지 않고 있는다.

일단 @EventListener로 변경하니 문제가 해결되어서 이슈는 해결하였지만 왜 그런일이 발생했는지 정확한 원인은 파악해보아야겠다.
