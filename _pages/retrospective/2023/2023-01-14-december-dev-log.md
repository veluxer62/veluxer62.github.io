---
layout: single
title: "2022년 12월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2022-december-dev-log/
---

회사에서 올해 설정한 KPI를 달성했다. 여러가지 지표가 있었지만 가장 중요한 목표는 월별 주문수를 달성하는 것이었다. 솔직히 주문서비스를 오픈하고 대표님께서 KPI목표를 발표하실때까지만해도 별 기대를 하지 않았고 보상에 대해서도 시큰둥하게 받아들였다. 이전 회사에서도 OKR를 설정하고 달성율에 따라 성과금을 준다고 하였지만 목표치가 터무니 없이 높았고 이번에도 그럴 것이라 예상했기 때문이다.

하지만 7월과 8월이 지나가면서 점점 주문수가 올라가기 시작하면서 어쩌면 목표했던 수치를 달성할 수 있을 것 같다라는 기대감이 생기기 시작했고 9월 10월 들어서는 아래 그래프처럼 주문수가 기하급수적으로 늘어가기 시작하면서 어쩌면 KPI를 달성할 수 있지 않을까라는 기대감이 점점더 커지기 시작했다.

![2022-kpi-graph](/assets/images/retrospective/2022-kpi-graph.png)

그래서 10월부터는 KPI를 달성하기 위해 제품 개발 및 운영 전략을 단기적으로 변경하고 KPI 목표 달성을 위한 단기적인 개발에 집중하였다. 물론 장기적으로 좋아보이진 않는다고 생각한다. 하지만 해당 전략덕분인지 11월쯤에는 가시적인 예상치가 나오기 시작했고 결국 목표를 달성할 수 있게 되었다.

요즘 회사에서는 KPI보다는 OKR을 더 선호하는 추세인것 같다. OKR이나 KPI에 대한 나의 지식이 부족해서인지는 몰라도 OKR보다 KPI가 좀더 정량적으로 다가갈 수 있고 단기적으로 달성할 수 있는 목표가 제시되어서 동기부여가 더 잘되었던 것 같다. 물론 지속적으로 단기목표만 쫓다보면 제품을 크게 보았을 때 좋지 않은 방향으로 갈 수 있다고 생각한다. 하지만 이전에 OKR을 두고 회사의 장기적인 목표를 위해 제품을 개발할때에는 관리자 입장에서는 어땠을지는 모르겠지만 팀원으로써는 목표에 대한 동기부여가 크게 와닿지는 않았던 것 같다.

이야기가 조금 다른곳으로 샜지만 아무튼 KPI를 달성함으로써 좋지 않은 외부환경에도 불구하고 제품에 대한 시장성을 확인하고 회사 구성원들의 노력에 대한 보상을 받을 수 있어 좋은 한해의 마무리가 되었던 것 같다.

<br/>

KPI 목표달성에 대한 예상치가 어느정도 나오는 11월까지만 주문서비스에 대한 기능 개발을 집중하기로 하고 12월부터는 내년을 위한 다음 과제를 위한 프로젝트를 시작하게 되었다. 청구/결제 프로젝트인데 주문서비스가 유통사와 매장의 식자재 주문에 대한 편의성을 제공해주었다면 청구/결제 서비스는 유통사와 매장의 식자재 청구 및 수납에 대한 편의성을 제공하기 위한 서비스이다.

그동안 PMO 및 PD분들께서 시장조사 및 사전 인터뷰를 통해서 요구사항들을 정리해오셨고 이를 기반으로 팀 전체가 정구/결제 서비스를 위한 요구사항 구체화 논의를 시작하였다. 요구사항을 정리하면서 우리가 생각하는 청구/결제에 대한 일반적인 사용자 스토리와는 조금 상이한 요구사항들이 있었고 이는 유통사와 현재 매장의 상황에 의해 어쩔 수 없이 도출된 것이라는 이야기가 나오면서 해당 요구사항을 조율하는데 많은 논의를 진행하였다. (사실 명명백백한 요구사항에 대해서는 특별하게 논의를 할 내용이 없긴하다)

논의과정에서 재미있는 현상이 발견되었는데, 이상적이진 않지만 유통사와 매장의 현재 상황을 크게 해치지 않는 선에서 기능을 먼저 구현하는 쪽으로 합의가 되었지만 논의가 길어지면 길어질수록 처음에 이의가 제기되었던 이상적인 사용자 스토리로 기능 요구사항으로 회기하는 현상이었다. 모두가 어쩔 수 없다고 받아들이기는 하지만 마음한켠으로는 자기가 생각하는 이상적인 방법으로 제품을 만들어보려는 노력이 엿보이는 대목이었다. 하지만 여기서 나는 요구사항과 실제 구현에서 괴리가 발생한다고 생각되었다. 이렇게 사전에 긴 시간을 내어 협의하고 조율하는데도 계속해서 괴리가 생기는데 만약 이런 조율없이 바로 제품을 만들기 시작하면 그 괴리는 얼마나 커졌을까?

논의가 끝난 후 설계를 할 때에도 간혹 요구사항에 대한 불만들이 나오곤 하였다. 현재 시장 상황이 우리가 겪은 경험상 이상적이지 않다고 생각하기 때문이라 생각한다. 하지만 고객 입장에서는 우리가 생각한 불편함이 오히려 익숙하고 편한것일 수도 있다는 이야기들을 계속 들으면서 고객의 관성을 깨트리는 것이 얼마나 어려운지 다시한번 느끼게 되었다.

<br/>

아래는 12월동안 정리한 이슈 내용들이다.

## 육각형 모델에 대한 좋은 글

육각형 모델에 대한 좋은 글이 있어 적어본다.

[https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/){: target="\_blank" }

DDD(Domain Driven Design), CQRS(Command and Query Responsibility Segregation)등을 이야기할 때 빠지지 않고 이야기 되는 아키텍쳐가 바로 육각형 모델(Hexagonal Architecture)이다. 사실 개인적으로 육각형 모델이 이상적이긴 하지만 그 복잡함으로 인해 프로젝트에 도입하기가 쉽지 않다라고 생각된다. 만약 내가 잘 이해하고 사용한다고 하더라도 팀원 모두가 그 개념을 잘 이해하고 사용하기가 쉽지 않다고 생각되기 때문에 차라리 단순하고 명백한 구조를 가진 아키텍쳐를 먼저 도입하고 조금씩 개선하는 방법으로 가는게 어떤가 생각된다.

## ChatGPT

[ChatGPT](https://openai.com/api/)에 대한 이야기가 많이 보여서 나도 테스트 해보았다. 확실히 이전까지 봐왔던 AI봇들과는 퀄리티가 다르다고 생각된다. 앞으로 데이터가 더 많이 쌓일수록 더 높은 퀄리티를 제공하지 않을까 생각된다.

ChatGPT를 이용해서 개발에 활용할 수도 있다는 이야기들도 있는데 개인적으로 개발은 단순한 기능구현 뿐만아니라 아키텍팅도 중요하다고 생각하기 때문에 ChatGPT를 대체하기에는 시간이 더 필요하지 않을까 생각된다.

## jjwt

Java의 JWT 라이브러리인 [jjwt](https://github.com/jwtk/jjwt){: target="\_blank" }가 버전이 올라가면서 `io.jsonwebtoken:jjwt`에서 `io.jsonwebtoken:jjwt-api`로 패키지가 변경되었다. 그러면서 API도 여러가지 바꼈는데 버전을 업그레이드 하려면 이것저것 바꿔야할게 많아 보인다.

## DGS 버전 이슈

[DGS](https://github.com/Netflix/dgs-framework){: target="\_blank" }의 버전이 `5.4.x`로 업그레이드하면서 빌드시 오류가 발생하는 현상이 있었다.

`dependency-management`의 이슈로 보이는데 아래와 같이 `gradle.properties`를 설정하면 해결할 수 있다.

```
graphql-java.version=19.2
```

해당 이슈는 [https://github.com/Netflix/dgs-framework/issues/1281#issuecomment-1284694300](https://github.com/Netflix/dgs-framework/issues/1281#issuecomment-1284694300){: target="\_blank" }에서 참고하자.

## Firebase Admin Java SDK 버전 이슈

[Firebase Admin Java SDK](https://github.com/firebase/firebase-admin-java){: target="\_blank" }의 버전을 업그레이드 하면서 `google-client-api`에서 빌드 오류가 발생하였다. 해당이슈는 Spring Boot에서 GCP 관련 의존성이 함께 사용되는 경우 발생한다. 해결방법은 아래와 같이 `google-client-api`를 제외처리 해주면 해결할 수 있다.

```kotlin
implementation("com.google.firebase:firebase-admin:9.1.1") {
    exclude(group = "com.google.api-client", module = "google-api-client")
}
```

해당 이슈는 [https://github.com/firebase/firebase-admin-java/issues/533](https://github.com/firebase/firebase-admin-java/issues/533){: target="\_blank" }에서 참고하자.

## mockk verify 이슈

mockk을 이용하여 verify를 수행할 때 여러번 실행되는 경우가 있다.
이때 byteArrayStream과 같이 stream 객체를 이용하면 readByte를 호출하면 한번만 값을 반환하기 때문에 원치하는 거짓양성이 발생할 수 있다.
이를 해결할 방법은 없을까? 방법을 찾아봐야겠다.

## name lock

Mysql에서 제공해주는 lock 기능 중에 `Name Lock`이 있다는 것을 알게되었다. 다른 데이터베이스에서는 사용하는지 모르겠지만 Mysql에서는 테이블의 이름을 변경하거나 삭제될때 테이블 레벨에서 묵시적으로 네임 락을 사용한다고 한다.

[saltfactory님의 MySQL에서 사용하는 Lock 이해](http://web.bluecomtech.com/saltfactory/blog.saltfactory.net/database/introduce-mysql-lock.html){: target="\_blank" }글을 참고하자.

## JPA Cascade 이슈

아래와 같이 두 Entity에 서로 OneToOne으로 관계가 맺어져 있는 경우 `Bar` Entity가 삭제되지 않는 이슈가 있었다.

```kotlin
class Foo {
    @OneToOne(mappedBy = "foo2", orphanRemoval = true)
    var bar: Bar? = null
        protected set

    @OneToOne(mappedBy = "foo", cascade = [CascadeType.ALL], orphanRemoval = true)
    var bar2: Bar? = null
        protected set
}

class Bar {
    @OneToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "id", nullable = false)
    val foo: Foo = foo

    @OneToOne(fetch = FetchType.LAZY, optional = false, cascade = [CascadeType.ALL], orphanRemoval = true)
    @JoinColumn(name = "foo_id", nullable = false)
    val foo2: Foo = foo2
}
```

해당 문제는 Cascade 문제로 아래와 같이 `Foo` Entity의 `bar`연관관계에 cascade를 추가해주면 해결된다.

```kotlin
class Foo {
    @OneToOne(mappedBy = "foo2", cascade = [CascadeType.REMOVE], orphanRemoval = true)
    var bar: Bar? = null
        protected set

    ...생략
}
```

문제는 해결되었는데....왜 Cascade가 삭제 추제인 Entity에 영향을 미치는지는 이해하지 못했다....어렵다...JPA

## vucuum관련해서 좋은 글

우아한형제들 블로그에서 vuccum관련해서 좋은 글이 게시되었다. 수박 겉핥기식으로 알고 있었는데 자세한 설명이 있어서 좋았다.

[PostgreSQL Vacuum에 대한 거의 모든 것](https://techblog.woowahan.com/9478/){: target="\_blank" }

## gradle 업그레이드 방법

Gradle 버전을 업그레이드 하려면 어떻게 할 수 있을까? 단순히 `gradle-wrapper.properties`에 적혀있는 버전을 바꾸면 된다고 생각할 수도 있지만 Gradle 홈페이지에서 제공하는 좀더 나은 방법이 있다. 바로 아래 명령어를 실행하는 것이다.

```
./gradlew wrapper --gradle-version 7.6
```

좀 더 자세한 내용은 [https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:upgrading_wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:upgrading_wrapper){: target="\_blank" }에서 참고하자.

## 함수명에 대한 고민

코드리뷰를 하면서 아래와 같이 재미있는 논의가 있어서 적어본다.

validation에 대한 내용을 함수명에 표현하면 좋겠다는 의견이 있었다. 예를 들면 아래와 같은 것이다.

```kotlin
fun getFoo(id: UUID): Foo {
    val config = repository.getConfig()
    if (!config.useable) {
        throw IllegalStateException()
    }

    return repository.findById(id).orElseThrow()
}
```

위 함수명을 적을 때 `getFoo`가 아닌 `getFooIfUseableAndExists`와 같이 적어줘야할까? 나의 의견은 함수의 추상화 수준에 따라 다르겠지만 `getFoo`가 낫다라는 생각이다.

만약 예외를 해당 함수를 사용하는 코드에 인지시켜 주고 싶다면 지금은 선호되지 않지만 [Checked Exception](https://en.wikipedia.org/wiki/Exception_handling#Checked_exceptions){: target="\_blank" }이나 함수형에서 주로 사용하는 [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html){: target="\_blank" }을 활용하면 어떨까 생각된다.

함수명에 모든 제약조건에 대한 내용이 담긴다면 지나치게 함수명이 길어져 가독성이 오히려 떨여지지 않을까라는 생각도 든다. 다만 앞에서 말했다시피 추상화 수준에 따라서 선택할 여지는 있어보인다.

## Throttling Consumer

샌드버드의 메시지 전송 제약조건때문에 다수의 주문서를 취소하는 경우 주문서 취소 메시지가 실패하는 이슈가 있었다. 그에 대한 해결방법으로 `Throttling Consumer`를 구현했는데 구현에 대한 샘플 코드는 아래 저장소에서 참고하자. 해당 방법은 추후에 블로그로도 적어봐야겠다.

[https://github.com/veluxer62/throttling-consumer-tutorial](https://github.com/veluxer62/throttling-consumer-tutorial){: target="\_blank" }

## JMS message converter 에러 해결방법

JMS를 이용하여 메시지를 발송하는데 아래와 같이 에러가 발생하였다.

```
2022-12-28 12:44:36.096  INFO 1 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 35 migrations (execution time 00:00.245s)
2022-12-28 12:44:36.181  INFO 1 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema "public": 35
2022-12-28 12:44:36.187  INFO 1 --- [           main] o.f.core.internal.command.DbMigrate      : Schema "public" is up to date. No migration necessary.
2022-12-28 12:44:36.805  INFO 1 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2022-12-28 12:44:36.954  INFO 1 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.6.14.Final
2022-12-28 12:44:37.676  INFO 1 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
2022-12-28 12:44:37.983  INFO 1 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.PostgreSQL10Dialect
2022-12-28 12:44:44.098  INFO 1 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2022-12-28 12:44:44.155  INFO 1 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
{"@timestamp":"2022-12-28T12:45:08.344Z","log.level": "WARN","message":"spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.springframework.boot.autoconfigure.orm.jpa.JpaBaseConfiguration$JpaWebConfiguration"}
{"@timestamp":"2022-12-28T12:45:16.855Z","log.level": "INFO","message":"The default project ID is null","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"com.google.cloud.spring.autoconfigure.core.GcpContextAutoConfiguration"}
{"@timestamp":"2022-12-28T12:45:17.155Z","log.level": "INFO","message":"Default credentials provider for service account firebase-adminsdk-o5ts8@dodo-cart-dev.iam.gserviceaccount.com","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"com.google.cloud.spring.core.DefaultCredentialsProvider"}
{"@timestamp":"2022-12-28T12:45:17.156Z","log.level": "INFO","message":"Scopes in use by default credentials: [https://www.googleapis.com/auth/pubsub, https://www.googleapis.com/auth/spanner.admin, https://www.googleapis.com/auth/spanner.data, https://www.googleapis.com/auth/datastore, https://www.googleapis.com/auth/sqlservice.admin, https://www.googleapis.com/auth/devstorage.read_only, https://www.googleapis.com/auth/devstorage.read_write, https://www.googleapis.com/auth/cloudruntimeconfig, https://www.googleapis.com/auth/trace.append, https://www.googleapis.com/auth/cloud-platform, https://www.googleapis.com/auth/cloud-vision, https://www.googleapis.com/auth/bigquery, https://www.googleapis.com/auth/monitoring.write]","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"com.google.cloud.spring.core.DefaultCredentialsProvider"}
{"@timestamp":"2022-12-28T12:45:41.375Z","log.level": "INFO","message":"Will secure any request with [org.springframework.security.web.session.DisableEncodeUrlFilter@dd4fac6, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@1832db43, org.springframework.security.web.context.SecurityContextPersistenceFilter@1aba0267, org.springframework.security.web.header.HeaderWriterFilter@78df2a0e, org.springframework.web.filter.CorsFilter@4f8c8136, org.springframework.security.web.authentication.logout.LogoutFilter@3b591a07, com.spoqa.cart.application.configuration.security.JwtAuthenticationFilter@39954f8a, com.spoqa.cart.application.configuration.security.JwtAuthenticationFilter@13bd90b4, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@3203d5, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@2ddbc030, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@1d5ba943, org.springframework.security.web.session.SessionManagementFilter@36fedb2b, org.springframework.security.web.access.ExceptionTranslationFilter@24257f51, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@74ec23fc]","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.springframework.security.web.DefaultSecurityFilterChain"}
{"@timestamp":"2022-12-28T12:45:42.950Z","log.level": "INFO","message":"Configuring GraphiQL to use GraphQL endpoint at '/graphql'","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"com.netflix.graphql.dgs.webmvc.autoconfigure.GraphiQLConfigurer"}
{"@timestamp":"2022-12-28T12:45:51.276Z","log.level": "INFO","message":"Exposing 1 endpoint(s) beneath base path '/actuator'","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.springframework.boot.actuate.endpoint.web.EndpointLinksResolver"}
{"@timestamp":"2022-12-28T12:45:51.980Z","log.level": "INFO","message":"Adding {logging-channel-adapter:_org.springframework.integration.errorLogger} as a subscriber to the 'errorChannel' channel","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.springframework.integration.endpoint.EventDrivenConsumer"}
{"@timestamp":"2022-12-28T12:45:51.981Z","log.level": "INFO","message":"Channel 'application.errorChannel' has 1 subscriber(s).","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.springframework.integration.channel.PublishSubscribeChannel"}
{"@timestamp":"2022-12-28T12:45:51.981Z","log.level": "INFO","message":"started bean '_org.springframework.integration.errorLogger'","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.springframework.integration.endpoint.EventDrivenConsumer"}
{"@timestamp":"2022-12-28T12:45:52.285Z","log.level": "INFO","message":"starting server: Undertow - 2.2.20.Final","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"io.undertow"}
{"@timestamp":"2022-12-28T12:45:52.366Z","log.level": "INFO","message":"XNIO version 3.8.7.Final","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.xnio"}
{"@timestamp":"2022-12-28T12:45:52.453Z","log.level": "INFO","message":"XNIO NIO Implementation Version 3.8.7.Final","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.xnio.nio"}
{"@timestamp":"2022-12-28T12:45:52.785Z","log.level": "INFO","message":"JBoss Threads version 3.1.0.Final","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.jboss.threads"}
{"@timestamp":"2022-12-28T12:45:53.094Z","log.level": "INFO","message":"Undertow started on port(s) 8080 (http)","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"org.springframework.boot.web.embedded.undertow.UndertowWebServer"}
{"@timestamp":"2022-12-28T12:45:53.782Z","log.level": "INFO","message":"Initializing Spring DispatcherServlet 'dispatcherServlet'","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"XNIO-1 task-1","log.logger":"io.undertow.servlet"}
{"@timestamp":"2022-12-28T12:45:53.786Z","log.level": "INFO","message":"Initializing Servlet 'dispatcherServlet'","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"XNIO-1 task-1","log.logger":"org.springframework.web.servlet.DispatcherServlet"}
{"@timestamp":"2022-12-28T12:45:53.843Z","log.level": "INFO","message":"Completed initialization in 57 ms","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"XNIO-1 task-1","log.logger":"org.springframework.web.servlet.DispatcherServlet"}
{"@timestamp":"2022-12-28T12:45:54.456Z","log.level": "INFO","message":"Started ApplicationKt in 97.285 seconds (JVM running for 98.786)","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"main","log.logger":"com.spoqa.cart.ApplicationKt"}
{"@timestamp":"2022-12-28T12:47:54.277Z","log.level":"ERROR","message":"rest internal error: org.springframework.jms.support.converter.MessageConversionException: Could not map JSON object [SampleMessage(date=2022-12-28T21:47:53.855124335+09:00[Asia/Seoul], text=TEST)]; nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Java 8 date/time type `java.time.ZonedDateTime` not supported by default: add Module \"com.fasterxml.jackson.datatype:jackson-datatype-jsr310\" to enable handling (through reference chain: com.spoqa.cart.application.presentation.rest.SampleMessage[\"date\"])\n\tat org.springframework.jms.support.converter.MappingJackson2MessageConverter.toMessage(MappingJackson2MessageConverter.java:195)\n\tat org.springframework.jms.core.JmsTemplate.lambda$convertAndSend$5(JmsTemplate.java:661)\n\tat org.springframework.jms.core.JmsTemplate.doSend(JmsTemplate.java:604)\n\tat org.springframework.jms.core.JmsTemplate.lambda$send$3(JmsTemplate.java:586)\n\tat org.springframework.jms.core.JmsTemplate.execute(JmsTemplate.java:504)\n\tat org.springframework.jms.core.JmsTemplate.send(JmsTemplate.java:584)\n\tat org.springframework.jms.core.JmsTemplate.convertAndSend(JmsTemplate.java:661)\n\tat com.spoqa.cart.application.presentation.rest.HomeController.test(HomeController.kt:31)\n\tat java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\n\tat java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:78)\n\tat java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\n\tat java.base/java.lang.reflect.Method.invoke(Method.java:568)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:150)\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:117)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1071)\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:964)\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)\n\tat org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:497)\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:584)\n\tat io.undertow.servlet.handlers.ServletHandler.handleRequest(ServletHandler.java:74)\n\tat io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:129)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:337)\n\tat org.springframework.security.web.access.intercept.FilterSecurityInterceptor.invoke(FilterSecurityInterceptor.java:115)\n\tat org.springframework.security.web.access.intercept.FilterSecurityInterceptor.doFilter(FilterSecurityInterceptor.java:81)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:122)\n\tat org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:116)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:126)\n\tat org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:81)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter(AnonymousAuthenticationFilter.java:109)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter(SecurityContextHolderAwareRequestFilter.java:149)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter(RequestCacheAwareFilter.java:63)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:223)\n\tat org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:217)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:223)\n\tat org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:217)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:103)\n\tat org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:89)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.web.filter.CorsFilter.doFilterInternal(CorsFilter.java:91)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.header.HeaderWriterFilter.doHeadersAfter(HeaderWriterFilter.java:90)\n\tat org.springframework.security.web.header.HeaderWriterFilter.doFilterInternal(HeaderWriterFilter.java:75)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:112)\n\tat org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:82)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal(WebAsyncManagerIntegrationFilter.java:55)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.session.DisableEncodeUrlFilter.doFilterInternal(DisableEncodeUrlFilter.java:42)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:346)\n\tat org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:221)\n\tat org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:186)\n\tat org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:354)\n\tat org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:267)\n\tat io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61)\n\tat io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131)\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61)\n\tat io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131)\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61)\n\tat io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131)\n\tat org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:96)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61)\n\tat io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131)\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61)\n\tat io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131)\n\tat io.undertow.servlet.handlers.FilterHandler.handleRequest(FilterHandler.java:84)\n\tat io.undertow.servlet.handlers.security.ServletSecurityRoleHandler.handleRequest(ServletSecurityRoleHandler.java:62)\n\tat io.undertow.servlet.handlers.ServletChain$1.handleRequest(ServletChain.java:68)\n\tat io.undertow.servlet.handlers.ServletDispatchingHandler.handleRequest(ServletDispatchingHandler.java:36)\n\tat io.undertow.servlet.handlers.RedirectDirHandler.handleRequest(RedirectDirHandler.java:68)\n\tat io.undertow.servlet.handlers.security.SSLInformationAssociationHandler.handleRequest(SSLInformationAssociationHandler.java:117)\n\tat io.undertow.servlet.handlers.security.ServletAuthenticationCallHandler.handleRequest(ServletAuthenticationCallHandler.java:57)\n\tat io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43)\n\tat io.undertow.security.handlers.AbstractConfidentialityHandler.handleRequest(AbstractConfidentialityHandler.java:46)\n\tat io.undertow.servlet.handlers.security.ServletConfidentialityConstraintHandler.handleRequest(ServletConfidentialityConstraintHandler.java:64)\n\tat io.undertow.security.handlers.AuthenticationMechanismsHandler.handleRequest(AuthenticationMechanismsHandler.java:60)\n\tat io.undertow.servlet.handlers.security.CachedAuthenticatedSessionHandler.handleRequest(CachedAuthenticatedSessionHandler.java:77)\n\tat io.undertow.security.handlers.AbstractSecurityContextAssociationHandler.handleRequest(AbstractSecurityContextAssociationHandler.java:43)\n\tat io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43)\n\tat io.undertow.servlet.handlers.SendErrorPageHandler.handleRequest(SendErrorPageHandler.java:52)\n\tat io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43)\n\tat io.undertow.servlet.handlers.ServletInitialHandler.handleFirstRequest(ServletInitialHandler.java:275)\n\tat io.undertow.servlet.handlers.ServletInitialHandler.access$100(ServletInitialHandler.java:79)\n\tat io.undertow.servlet.handlers.ServletInitialHandler$2.call(ServletInitialHandler.java:134)\n\tat io.undertow.servlet.handlers.ServletInitialHandler$2.call(ServletInitialHandler.java:131)\n\tat io.undertow.servlet.core.ServletRequestContextThreadSetupAction$1.call(ServletRequestContextThreadSetupAction.java:48)\n\tat io.undertow.servlet.core.ContextClassLoaderSetupAction$1.call(ContextClassLoaderSetupAction.java:43)\n\tat io.undertow.servlet.handlers.ServletInitialHandler.dispatchRequest(ServletInitialHandler.java:255)\n\tat io.undertow.servlet.handlers.ServletInitialHandler.access$000(ServletInitialHandler.java:79)\n\tat io.undertow.servlet.handlers.ServletInitialHandler$1.handleRequest(ServletInitialHandler.java:100)\n\tat io.undertow.server.Connectors.executeRootHandler(Connectors.java:387)\n\tat io.undertow.server.HttpServerExchange$1.run(HttpServerExchange.java:852)\n\tat org.jboss.threads.ContextClassLoaderSavingRunnable.run(ContextClassLoaderSavingRunnable.java:35)\n\tat org.jboss.threads.EnhancedQueueExecutor.safeRun(EnhancedQueueExecutor.java:2019)\n\tat org.jboss.threads.EnhancedQueueExecutor$ThreadBody.doRunTask(EnhancedQueueExecutor.java:1558)\n\tat org.jboss.threads.EnhancedQueueExecutor$ThreadBody.run(EnhancedQueueExecutor.java:1449)\n\tat org.xnio.XnioWorker$WorkerThreadFactory$1$1.run(XnioWorker.java:1282)\n\tat java.base/java.lang.Thread.run(Thread.java:831)\nCaused by: com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Java 8 date/time type `java.time.ZonedDateTime` not supported by default: add Module \"com.fasterxml.jackson.datatype:jackson-datatype-jsr310\" to enable handling (through reference chain: com.spoqa.cart.application.presentation.rest.SampleMessage[\"date\"])\n\tat com.fasterxml.jackson.databind.exc.InvalidDefinitionException.from(InvalidDefinitionException.java:77)\n\tat com.fasterxml.jackson.databind.SerializerProvider.reportBadDefinition(SerializerProvider.java:1300)\n\tat com.fasterxml.jackson.databind.ser.impl.UnsupportedTypeSerializer.serialize(UnsupportedTypeSerializer.java:35)\n\tat com.fasterxml.jackson.databind.ser.BeanPropertyWriter.serializeAsField(BeanPropertyWriter.java:728)\n\tat com.fasterxml.jackson.databind.ser.std.BeanSerializerBase.serializeFields(BeanSerializerBase.java:774)\n\tat com.fasterxml.jackson.databind.ser.BeanSerializer.serialize(BeanSerializer.java:178)\n\tat com.fasterxml.jackson.databind.ser.DefaultSerializerProvider._serialize(DefaultSerializerProvider.java:480)\n\tat com.fasterxml.jackson.databind.ser.DefaultSerializerProvider.serializeValue(DefaultSerializerProvider.java:319)\n\tat com.fasterxml.jackson.databind.ObjectWriter$Prefetch.serialize(ObjectWriter.java:1518)\n\tat com.fasterxml.jackson.databind.ObjectWriter._writeValueAndClose(ObjectWriter.java:1219)\n\tat com.fasterxml.jackson.databind.ObjectWriter.writeValue(ObjectWriter.java:1060)\n\tat org.springframework.jms.support.converter.MappingJackson2MessageConverter.mapToTextMessage(MappingJackson2MessageConverter.java:280)\n\tat org.springframework.jms.support.converter.MappingJackson2MessageConverter.toMessage(MappingJackson2MessageConverter.java:185)\n\t... 122 more\n","ecs.version": "1.2.0","service.name":"dodo-cart-service-kotlin","event.dataset":"dodo-cart-service-kotlin.CONSOLE","process.thread.name":"XNIO-1 task-1","log.logger":"com.spoqa.cart.application.presentation.rest.ControllerExceptionHandler","transaction.id":"857c901bb252b1d6","trace.id":"71ca340ca0ec7a87342456a43a8815a4
```

원인은 `LocalDateTime`객체를 변환할 수 없다는 것이었는데 아래와 같이 설정해주면 해당 문제를 해결할 수 있다.

```kotlin
@Configuration
class MessageConfig {
    @Bean
    fun jacksonJmsMessageConverter(): MessageConverter {
        return MappingJackson2MessageConverter().apply {
            val objectMapper = ObjectMapper().apply { registerModule(JavaTimeModule()) }
            setObjectMapper(objectMapper)
            setTargetType(MessageType.TEXT)
            setTypeIdPropertyName("_type")
        }
    }
}
```

예전에도 해당 이슈를 겪었던 것 같은데 잊어버렸나보다 ^^;;

## ssm port forwarding

VPC 설정으로인해 외부에서 AWS의 특정 인스턴스로 접근할 수 없는 경우 SSM을 이용하여 접근할 수 있다. 하지만 어떤 상황에서는 포트포워딩을 사용해야하는 경우가 있는데(브라우저 실행이라던지) 이런 경우 아래와 같이 ssm port forwarding 명령어를 사용하면 좋다.

```
# find the instance ID based on Tag Name
INSTANCE_ID=$(aws ec2 describe-instances \
               --filter "Name=tag:Name,Values=CodeStack/NewsBlogInstance" \
               --query "Reservations[].Instances[?State.Name == 'running'].InstanceId[]" \
               --output text)
# create the port forwarding tunnel
aws ssm start-session --target $INSTANCE_ID \
                       --document-name AWS-StartPortForwardingSession \
                       --parameters '{"portNumber":["80"],"localPortNumber":["9999"]}'
```

만약 특정 리모트 서버의 포트를 포워딩하려면 아래와 같이 명령어를 실행하면된다.

```
aws ssm start-session --target <INSTANCE-ID> \
--document-name AWS-StartPortForwardingSessionToRemoteHost \
--parameters '{"host":[<HOST>],"portNumber":["80"],"localPortNumber":["9999"]}'
```
