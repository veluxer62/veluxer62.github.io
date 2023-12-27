---
layout: single
title: "2022년 3월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2022-march-dev-log/
---

3월까지 언어전환 작업을 끝내고 QA에 입고하기로 하면서 바쁜나날을 보냈던것 같다. 다행히 약속했던 기한은 지켰지만 막바지에 일정에 치여서 코드 퀄리티에 많이 신경쓰지 못한것 같아서 조금 아쉬움이 있다. QA기간 중에는 배치 작업을 진행해야하고 배치 작업 이후에는 바로 다음 프로젝트가 있다. 쌓여있는 업무들이 많이 있어서 기술 부채를 계속 쌓아갈것 같은 불안감이 든다. 물론 코드리뷰를 통해서 최소한의 기술 부채를 줄이긴 위한 노력들을 이어갈테지만 과거의 기술 부채 또는 개선사항들을 지속해서 고치기 위해서 어떤 노력을 할지 고민해봐야 겠다.

4월초에 지인과 함께 자전거 여행을 계획하고 있다. 처음 목표는 일주일 정도 휴가를 사용해서 서울-부산 코스로 가보려고 했지만 휴가를 길게 내기 좀 어려움이 있어서 서울-속초로 타협을 하였다. 여행을 가기 위한 준비물을 산다고 오랜만에 쇼핑을 많이 했다. 주머니는 가벼워졌지만 오랜만에 느끼는 설렘에 하루 빨리 마무리 짓고 떠나겠다는 마음으로 열심히 일할 수 있었던 것 같다.

일이 바빠지면서 매일 새벽에 자고 늦게 일어나면서 생활리듬이 많이 깨졌다. 그나마 운동은 계속해서 하고 있어서 컨디션이 최악은 아닌데 계속해서 이런 패턴을 이어가면 몸이 많이 상할것 같아 주의해야겠다. 그리고 일이 쌓여있다보니 공부보다 회사일을 하는 비율이 많이 높아졌다. 이번 회사에 오면서 공부보다 실무를 하면서 배운것들을 잘 정리하자는 마음가짐을 가졌지만 그래도 공부를 완전 하지 않는 것은 좋지 않은 것 같다. 의도적으로 공부하는 시간을 조금씩 할애해야겠다. 일단 공부해보고 싶은 것부터 정해봐야겠다.

아래는 3월동안 정리한 이슈 내용들이다.

## JPA의 Entity Property가 DB Column명이 예약어인 경우 처리

JPA를 이용하여 Entity를 정의할때 Column명은 class의 property명을 기본적으로 따라간다. 만약 property와 column명을 다르게 하고 싶다면 `@Column(name="foo") val foo: String`와 같이 처리하면 된다.

그런데 만약 DB Column명이 예약어인 경우에 위에서 말한 방식대로 정의한다면 영속화 시 오류가 발생한다. 왜냐하면 DB에서 예약어를 사용하는 경우 `"`를 붙여주어 예외처리를 해줘야하는데 전송되는 쿼리에서 예외처리를 해주지 않고 전송하기 때문이다. 그래서 아래와 같이 예외처리가 되도록 해주어야 한다.

```kotlin
@Column(name="\"foo\"") val foo: String
```

`@Column`을 이용하지 않고 아래와 같이 property명을 바꾸어 줄 수도 있다.

```kotlin
@Column val `foo`: String
```

개인적으로는 DB의 예약어 문제이기 때문에 class의 property를 바꾸어 주는거보다 `@Column`의 이름을 변경해주는게 좀 더 나아보인다.

## ktlint trailing comma

ktlint에서 trailing comma 룰을 적용하려면 [experimental-rules](https://github.com/pinterest/ktlint/blob/master/README.md#experimental-rules){: target="\_blank" }을 적용해야 lint가 정상적으로 동작한다. (이전에 `experimental-rules`을 켜지 않고 `trailing comma`가 적용된 줄 알고 사용했었다 ㅠㅠ)

자세한 설정은 아래 코드를 참고하자.

```kotlin
// build.gradle.kts

plugins {
    id("org.jlleitschuh.gradle.ktlint") version "10.2.1"
}

ktlint {
    version.set("0.44.0")
    enableExperimentalRules.set(true)
}
```

## CircleCI에서 job별로 workspace 공유하는 방법

CircleCI에서 파이프라인을 만들다보면 job별로 파일을 공유해야할 필요가 있을 때가 있다. 특히 checkout한 파일을 가져와서 해당 파일을 이용하여 여러 job들을 나누어서 작업을 하다보면 job별로 context가 공유되지 않기 때문에 매번 checkout해야 하는 불편함이 있다. 매번 checkout해서 재사용해도 되지만 `persist_to_workspace`와 `attach_workspace`를 사용하면 checkout한 파일을 workflow가 진행되는 동안 공유할 수 있기 때문에 job별로 매번 checkout해야하는 불편함을 해소할 수 있다. (설정 방법은 [공식문서](https://circleci.com/docs/2.0/language-java-maven/#persisting-build-artifacts-to-workspace){: target="\_blank" }를 참고하자.)

## Github Pull Request File Tree

Github에 Pull Request 파일을 볼때 Tree로 보여주는 플러그인이 추가되었다. 예전에 [Octotree](https://www.octotree.io/){: target="\_blank" }를 통해 파일들을 트리형태로 보기도 하였는데 공식적인 플러그인을 제공해주니 편하다.

![Pull Request File Tre](/assets/images/retrospective/github-pull-request-file-tree.png)

## RecordApplicationEvents

Spring을 이용하여 이벤트를 이용한 기능을 개발할때 Application Event를 활용하곤 한다. 개인적으로 아래와 같이 [ApplicationEventPublisher](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationEventPublisher.html){: target="\_blank" }를 이용하여 이벤트를 발행하고 Listener를 이용하여 이벤트를 처리하는 형태로 개발하는데 이때 테스트는 주로 `ApplicationEventPublisher` stub을 만들어서 호출여부를 검증하는 형태로 개발하곤 하였다.

```kotlin
@SpringBootTest
class FooServiceTest(
    private val fooService: FooService,
) {
    @SpykBean
    private lateinit var applicationEventPublisher: ApplicationEventPublisher

    @Test
    fun test() {
        // Some Code...

        verify {
            val event = FooEvent()
            applicationEventPublisher.publishEvent(event)
        }
    }
}
```

위와 같은 방법 말고 [RecordApplicationEvents](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/event/RecordApplicationEvents.html){: target="\_blank" }를 이용하면 다른 방법으로 이벤트 발생여부를 검증할 수 있다.

```kotlin
@SpringBootTest
@RecordApplicationEvents
class FooServiceTest(
    private val fooService: FooService,
) {
    @Autowired
    private lateinit var applicationEvents: ApplicationEvents

    @Test
    fun test() {
        // Some Code...

        val actual = applicationEvents.stream(FooEvent::class.java).findFirst()
        val expected = FooEvent()
        actual shoulBe expected
    }
}
```

참고 블로그: [https://rieckpil.de/record-spring-events-when-testing-spring-boot-applications/](https://rieckpil.de/record-spring-events-when-testing-spring-boot-applications/){: target="\_blank" }

## TransactionalEventListener 에 대해서 배운것들

- 트랜잭션이 커밋되어야지만 이벤트가 실행된다. (after commit 옵션에서)
- 롤백 되면 이벤트가 발생하지 않는다. 즉 커밋이 끝나고 실행되기 때문에 순서에 대한 걱정을 할필요가 없다.
- `@Transactional`이 선언된 함수에서 호출했을 시에만 동작한다. 없는 함수가 실행되면 예외도 발생하지 않고 이벤트가 발생하지 않으므로 주의해야한다.
- `@Async`와 조합할 경우에도 동일하게 롤백 시 이벤트 발행하지 않는다. 다만 Transaction이 끊어지기 때문에 lazy 로딩은 동작하지 않는다.

## Logger 테스트 방법

테스트 시 Logger에 로그가 원하는대로 남았는지 확인하고 싶은 경우가 있다. 아래와 같이 코드를 작성하면 로그를 테스트할 수 있다.

```kotlin
import ch.qos.logback.classic.Level
import ch.qos.logback.classic.spi.ILoggingEvent
import ch.qos.logback.core.AppenderBase
import io.kotest.core.spec.style.FunSpec
import io.kotest.core.test.TestCase
import io.kotest.matchers.shouldBe
import mu.KLogging
import org.slf4j.Logger
import org.slf4j.LoggerFactory

class FooTest : FunSpec() {
    private val sut = Foo()
    private val logger = LoggerFactory.getLogger(Logger.ROOT_LOGGER_NAME) as ch.qos.logback.classic.Logger
    private val listAppender = TestAppender()

    override fun beforeEach(testCase: TestCase) {
        logger.addAppender(listAppender)
        listAppender.start()
    }

    init {
        context("a") {
            test("b") {
                sut.test()
                listAppender.lastEvent.level shouldBe Level.INFO
                listAppender.lastEvent.message shouldBe "test!!"
            }
        }
    }
}

class Foo {
    companion object : KLogging()

    fun test() {
        logger.info { "test!!" }
    }
}

class TestAppender : AppenderBase<ILoggingEvent>() {
    private val events: MutableList<ILoggingEvent> = mutableListOf()

    override fun append(e: ILoggingEvent) {
        events.add(e)
    }

    val lastEvent: ILoggingEvent get() = events.last()
}
```

## Spring Security CORS 설정 시 주의할점

### preflight

CORS preflight 요청은 인증처리를 하지 않겠다는 것인데, CORS semantic 상으로 CORS prefight에는 Authorization 헤더를 줄 이유가 없으므로 CORS preflight 요청에 대해서는 401 응답을 하면 안된다.

이 부분의 설정이 없으면 spring-security 설정에 의해서 CORS preflight 요청이 정상적으로 처리되지 못해서 실제 CORS 요청이 정상적으로 이루어지지 않는다.

그래서 .requestMatchers(CorsUtils::isPreFlightRequest).permitAll() 코드가 반드시 필요하다.

```kotlin
@Configuration
@EnableGlobalMethodSecurity(securedEnabled = true)
class WebSecurityConfig : WebSecurityConfigurerAdapter() {
    override fun configure(http: HttpSecurity) {
        http.csrf().disable()
            .cors().configurationSource(corsConfigurationSource())
            .and()
            .authorizeRequests()
            .requestMatchers(CorsUtils::isPreFlightRequest).permitAll()
            .anyRequest().authenticated()
    }

    @Bean
    fun corsConfigurationSource(): CorsConfigurationSource {
        return UrlBasedCorsConfigurationSource().apply {
            registerCorsConfiguration(
                "/**",
                CorsConfiguration().apply {
                    allowedOrigins = listOf("*")
                    allowedHeaders = listOf("*")
                    allowedMethods = listOf("*")
                    allowCredentials = true
                },
            )
        }
    }
}
```

### alloworigins

spring 5.3부터 cors 에서 `alloworigins`가 `*`이고 `allowCredentials`가 `true`이면 예외를 반환하도록 변경되었다. 그래서 만약 `alloworigins`를 전체로 하고 싶으면 `allowCredentials`를 `false`로 줘야한다.

아래 코드는 Spring Sercurity에 적용되어있는 CORS 설정 코드중 일부이다.

```java
/**
* Validate that when {@link #setAllowCredentials allowCredentials} is true,
* {@link #setAllowedOrigins allowedOrigins} does not contain the special
* value {@code "*"} since in that case the "Access-Control-Allow-Origin"
* cannot be set to {@code "*"}.
* @throws IllegalArgumentException if the validation fails
* @since 5.3
*/
public void validateAllowCredentials() {
    if (this.allowCredentials == Boolean.TRUE &&
            this.allowedOrigins != null && this.allowedOrigins.contains(ALL)) {

        throw new IllegalArgumentException(
                "When allowCredentials is true, allowedOrigins cannot contain the special value \"*\" " +
                        "since that cannot be set on the \"Access-Control-Allow-Origin\" response header. " +
                        "To allow credentials to a set of origins, list them explicitly " +
                        "or consider using \"allowedOriginPatterns\" instead.");
    }
}
```

참고

- [https://oddpoet.net/blog/2017/04/27/cors-with-spring-security/](https://oddpoet.net/blog/2017/04/27/cors-with-spring-security/){: target="\_blank" }
- [https://developer.mozilla.org/ko/docs/Web/HTTP/CORS#%ED%94%84%EB%A6%AC%ED%94%8C%EB%9D%BC%EC%9D%B4%ED%8A%B8\_%EC%9A%94%EC%B2%AD](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS#%ED%94%84%EB%A6%AC%ED%94%8C%EB%9D%BC%EC%9D%B4%ED%8A%B8_%EC%9A%94%EC%B2%AD){: target="\_blank" }
- [https://github.com/spring-projects/spring-framework/issues/24763](https://github.com/spring-projects/spring-framework/issues/24763){: target="\_blank" }

## JPA Embedded 클래스 null 관련 이슈

JPA의 Embedded 클래스는 flat하게 정의된 클래스들을 context별로 묶어줄때 유용하다. 만약 묶어준 column들이 모두 nullable한 값일때 모든 값이 `null`이라면 Embedded 클래스 자체도 `null`로 반환된다. Kotlin을 이용하여 변수를 정의할 때 `?`를 붙이지 않으면 `null`이 아닌 값이 할당되어 있을것이라 기대하게 된다. 하지만 Embedded 클래스의 이러한 특징으로 인해서 실제로 not-null로 변수를 정의하였더라도 null값을 반환하여 런타임 시 오류가 발생할 수 있다. 그러므로 Embedded 클래스의 모든 property가 nullable하다면 해당 클래스를 사용하는 Entity의 property도 nullable하게 정의해야한다.

```kotlin
class Foo(
    @Embedded
    var embeddedFoo: EmbeddedFoo?,
)

@Embeddable
class EmbeddedFoo(
    @Column
    var a: String?,

    @Column
    var b: String?,
)
```

## Kubernetes 환경에서 Spring Boot 어플리케이션에 AWS 권한 주입 시 이슈

k8s 환경에서 Spring Boot 어플리케이션 서버를 서빙할때 service account를 이용한 AWS 권한 주입시 spring cloud aws 의 기본설정으로는 권한을 제대로 주입받지 못하는 이슈가 있었다.

`AWS credential provider`는 별도의 설정이 없으면 `DefaultAWSCredentialsProviderChain`에 의해서 OS환경에 맞는 provider를 선택한다. service account를 주입받은 pod에는 `AWS_WEB_IDENTITY_TOKEN_FILE`, `AWS_ROLE_ARN` 환경변수가 존재하므로 `DefaultAWSCredentialsProviderChain`이 `WebIdentityTokenCredentialsProvider`를 반환할거라 예상했지만 `EC2ContainerCredentialsProviderWrapper`를 반환했다.

그래서 명시적으로 아래와 같이 `credentialProvidor` 빈을 주입시켰다.

```kotlin
@Configuration
class AwsCredentialProviderConfig {
    @Bean
    @Profile("prod", "qa", "test")
    fun credentialsProvider(): AWSCredentialsProvider = WebIdentityTokenCredentialsProvider()
}
```

이렇게해서 서버를 띄워봤는데 정상적으로 서버 기동이 안되었다.

원인은 sts 의존성이 빠진것이었는데 아래 의존성을 추가하면 잘된다.

```
implementation("com.amazonaws:aws-java-sdk-sts:1.12.132")
```

그렇다면 해당 의존성 때문에 `DefaultAWSCredentialsProviderChain`이 동작하지 않았던게 아닐까하는 의심을 해보았다. 그래서 위 설정코드인 `credentialsProvider` Bean을 추가하지 않고 다시 테스트 해보았고 자동으로 provider가 잡혔다. 결론은 `spring-cloud-starter-aws`를 사용할 때 `com.amazonaws:aws-java-sdk-sts`의존성을 추가해주지 않으면 `DefaultAWSCredentialsProviderChain`이 제대로 동작하지 않을 수 있으니 함께 의존성을 추가해주어야 한다.

## ShedLock

분산환경에서 스케줄링을 이용한 배치 어플리케이션을 만들고자 할때 Scale-out을 통한 성능향 상 시 고민해야 할 부분이 바로 스케줄 마다 한번만 실행되도록 하는 것이다. Sacle-out된 각 서버마다 배치잡이 실행되어버리면 안되기 때문이다. 이 때 선택할 수 있는 방법 중 하나가 `ShedLock`이다. `ShedLock`은 스케줄 설정마다 고유 이름을 설정하고 스케줄 실행 시 해당 이름을 가진 각 서버들 중 하나의 서버에서만 실행되도록 할 수 있다. 무엇보다 대안인 `Quartz`보다 쉽고 간단한 설정이 큰 장점이다.

```
@Component
class BaeldungTaskScheduler {

    @Scheduled(cron = "0 0/15 * * * ?")
    @SchedulerLock(name = "TaskScheduler_scheduledTask",
      lockAtLeastForString = "PT5M", lockAtMostForString = "PT14M")
    public void scheduledTask() {
        // ...
    }
}
```

스케줄러를 한번만 실행하도록 하기 위해서는 DB의 도움이 필요하기 때문에 이를 위한 테이블이 추가되어야한다. 자세한 내용은 [블로그 글](https://www.baeldung.com/shedlock-spring){: target="\_blank" }을 참고하자.

## JPA Entity equals 구현 시 주의할점

JPA에서 Entity간에 동일함을 표현하기 위해 아래와 같이 equals를 재정의하였다. (Entity는 ID가 같으면 동일 객체로 판단한다.)

```kotlin
override fun equals(other: Any?): Boolean {
    if (other == null || this::class != other::class) {
        return false
    }

    return (other as PrimaryKeyEntity).id == id
}

override fun hashCode() = Objects.hashCode(id)
```

그런데 QA시 발생한 이슈 중에 equals가 제대로 동작하지 않는 이슈가 있어서 좀 살펴보게 되었다.

JPA에서 OneToMany 상황에서 Lazy 로딩이 발생하면 반환되는 객체는 HibernateProxy객체이다. 그러다 보니 `this::class != other::class`에서 조건을 만족하지 않았고 그래서 위에서 재정의한 equals가 제대로 동작하지 않았다.

```kotlin
test("test") {
    val m1 = ManagerFactory().produce()
    val m2 = ManagerFactory().produce()
    val m3 = ManagerFactory().produce()
    val store = StoreFactory().produce()
    m1.addManagedStore(store)
    m2.addManagedStore(store)
    m3.addManagedStore(store)

    testEntityManager.persistAll(store, m1, m2, m3)
    testEntityManager.flush()
    testEntityManager.detach(store)
    testEntityManager.detach(m1)
    testEntityManager.detach(m2)
    testEntityManager.detach(m3)

    val actual = storeRepository.findById(store.id).get()
    actual.managers.forEach {
        println(it::class.java.toString() + " <> " + m1::class.java.toString())
        println(it::class.java == m1::class.java)

        /*
            class com.spoqa.cart.domain.manager.Manager$HibernateProxy$N3HsnR9U <> class com.spoqa.cart.domain.manager.Manager
            false
            class com.spoqa.cart.domain.manager.Manager$HibernateProxy$N3HsnR9U <> class com.spoqa.cart.domain.manager.Manager
            false
            class com.spoqa.cart.domain.manager.Manager$HibernateProxy$N3HsnR9U <> class com.spoqa.cart.domain.manager.Manager
            false
        */
    }
}
```

그래서 아래와 같이 equals를 재정의해주어서 문제를 해결해 주었다.

```kotlin
override fun equals(other: Any?): Boolean {
    if (other == null) {
        return false
    }

    if (other !is HibernateProxy && other !is PrimaryKeyEntity) {
        return false
    }

    return id == getIdentifier(other)
}

private fun getIdentifier(obj: Any): Serializable {
    return if (obj is HibernateProxy) {
        obj.hibernateLazyInitializer.identifier
    } else {
        (obj as PrimaryKeyEntity).id
    }
}
```

## Kubernetes에서 Spring Application의 JVM 메모리 설정

Kubernetes 환경에서는 JVM의 메모리 설정이 Pod의 메모리 설정가 차이가 있어 이슈(파드가 뜨지 않는 등)가 있을 수 있다. 그래서 구체적인 값을 정해주기 보다 비율로 정해주는 방법을 사용하면 좋다. 아래 코드는 Docker image를 생성해주는 JIB에 대한 설정코드이다.

```kotlin
jib {
    from {
        image = "openjdk:17-alpine"
    }
    to {
        image = "repository 경로"
        tags = setOf(getBuildVersion())
    }
    container {
        jvmFlags = listOf(
            "-XX:MinRAMPercentage=50.0",
            "-XX:MaxRAMPercentage=80.0",
        )
    }
}
```

참고: [https://jogeum.net/32](https://jogeum.net/32){: target="\_blank" }

## kotest 5.1.0 버전 이슈

kotest 5.1.0 버전 이후부터 Spring에서 kotest를 그대로 실행하면 오류가 발생하는데 아래와 같이 gradle.properties에 설정해주면 해결된다.

```
kotlin-coroutines.version=1.6.0
```

참고: [https://github.com/kotest/kotest/issues/2782](https://github.com/kotest/kotest/issues/2782){: target="\_blank" }

## gradle exclude 이슈

프로젝트에 tomcat 대신 undertow를 사용하려고 `spring-boot-starter-web`의존성에 `tomcat`모듈을 제외하려고 아래와 같이 `exclude`를 사용하였다.

```kotlin
implementation("org.springframework.boot:spring-boot-starter-undertow")
implementation("org.springframework.boot:spring-boot-starter-web") {
    exclude(group = "org.springframework.boot", module = "spring-boot-starter-tomcat")
}
```

exclude를 이용한 교체 예제가 많은데 무슨이유에서인지 undertow로 교체 되지 않는 이슈가 있었다. 그래서 아래와 같이 module을 replace해주었고 해당 설정으로는 잘 동작하였다.

```kotlin
implementation("org.springframework.boot:spring-boot-starter-undertow")
implementation("org.springframework.boot:spring-boot-starter-web")
modules {
    module("org.springframework.boot:spring-boot-starter-tomcat") {
        replacedBy("org.springframework.boot:spring-boot-starter-undertow")
    }
}
```

## ConditionalOnProperty

프로퍼티 값에 따라서 빈을 생성할 수 있도록 하는건데 @Profile이랑 비슷한 동작을 하는 듯 하다.

참고:

- [https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnProperty.html](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnProperty.html){: target="\_blank" }
- [https://www.baeldung.com/spring-conditionalonproperty](https://www.baeldung.com/spring-conditionalonproperty){: target="\_blank" }

## PG에서 Database의 모든 세션을 강제 종료하는 쿼리

```
SELECT
    pg_terminate_backend(pid)
FROM
    pg_stat_activity
WHERE
    pid <> pg_backend_pid()
    AND datname = 'mydb';
```

## Terraformer

적용된 Cloud 리소스를 Terraform 코드로 생성해주는 툴킷이다. Terraform 잘 모를때 생성된 코드를 참고해서 사용하면 아주 유용하다.

참고: [https://github.com/GoogleCloudPlatform/terraformer](https://github.com/GoogleCloudPlatform/terraformer){: target="\_blank" }
