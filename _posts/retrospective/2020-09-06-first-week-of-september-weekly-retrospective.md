---
layout: single
title: "2020년 9월 1주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

벌써 9월이 시작되었다. 코로나로 인해서 수영도 헬스장도 가기 힘들어서 운동을 한동안 쉬었었는데, 살도 많이 찌는거 같고 운동이 하고 싶어서 뭘 하면 좋을까 생각하다가 자전거는 혼자서 운동하기도 쉽고 재밌게 할 수 있을거 같아서 자전거로 운동을 하기로 했다. 그래서 로드 바이크 바이크를 하나 장만했다!!

![my-bike](/assets/images/retrospective/my-bike.jpeg)

앞으로 열심히 타고 다녀야 겠다!

아래는 금주동안 배우거나 발생한 이슈에 대한 내용을 정리해 보았다.

## Java Currency

회사에서 한국 뿐 아니라 미국, 일본 등에 클래스를 오픈하면서 새롭게 개발하는 서비스에 통화 단위를 입력하도록 개발하고 있다. 통화를 따로 다루어본적이 없어서 열거형 클래스를 만들어서 단위를 입력하는 정도로만 하였었는데, Java에 [Currency](https://docs.oracle.com/javase/8/docs/api/java/util/Currency.html){: target="\_blank" }를 제공한다는 것을 알게 되었다. 1차적으로 개발이 완료되고나면 임의로 만들었던 열거형 클래스를 Currency 클래스로 변경해야겠다.

## kotlin 1.4 업그레이드 시 klint 이슈

현재 진행중인 프로젝트에서 코드 형식의 일관성을 위해서 [ktlint-gradle](https://github.com/JLLeitschuh/ktlint-gradle){: target="\_blank" }를 사용중이다. 얼마전에 kotlin 1.4버전이 발표되면서 프로젝트에서 사용하는 kotlin의 버전을 1.4버전으로 변경하였는데, ktlint-gradle 라이브러리에서 오류가 생겨서 정상 동작하지 않았다. 확인해보니 ktlint-gradle에서 사용중인 [ktlint](https://github.com/pinterest/ktlint){: target="\_blank" }의 의존성 버전이 kotlin 1.4를 대응하기 이전 버전을 사용하기 때문이었다. 해당 문제는 아래와 같이 ktlint 버전을 명시해 줌으로써 해결할 수 있다.

관련 이슈 : [https://github.com/JLLeitschuh/ktlint-gradle/issues/383](https://github.com/JLLeitschuh/ktlint-gradle/issues/383){: target="\_blank" }

```gradle
ktlint {
    version.set("0.38.1")
}
```

## 쿠버네틱스 스터디

회사에서는 쿠버네틱스로 운영중인 서비스의 서버를 관리하고 있다. 쿠버네틱스에 대한 지식이 전무해서 회사에서 쿠버네틱스를 관리해주시는 개발자분께 조금씩 배워나가야 겠다. 목표는 쿠버네틱스에 대한 기본적인 개념을 잡고, 간단한 어플리케이션을 띄워보는것으로 하고 틈틈히 공부해 나가야 겠다.

## kotlin sealed class

Kotlin의 [sealed class](https://kotlinlang.org/docs/reference/sealed-classes.html){: target="\_blank" } 클래스는 쉽게 말해 Enum 클래스의 확장판이라고 생각하면 된다. 개발할때 특정 인터페이스를 구현한 구현클래스들의 타입 체크를 할때 when절을 사용하는데 default를 사용할 수 밖에 없어서 사용하지 않는 방법이 없을까 해서 찾은게 해당 클래스이다. 한번 적용해봐야겠다.

## Cron 표현식

스케줄러를 개발할때 다양한 스케줄링 라이브러리들이 있는데, 대부분의 이런 라이브러리들이 Cron 표현식을 지원하고 어느정도 표현식을 숙지하고 있으면 다양한 케이스의 스케줄러를 구현할 수 있다. 얼마전에 평소에는 사용하지 않아 몰랐던 표현 식이 있어 적어본다.

바로 `L`과 `W`인데, `L`은 일에서 사용하면 마지막 일, 요일에서는 마지막 요일(토요일)을 의미하고, `W`는 가장 가까운 평일을 의미한다.

Cron 표현식은 모두 외우지는 못해도 사용가능한 표현식이 어떤게 있는지 정도는 숙지해 두면 좋을것 같다.

Cron 표현식을 테스트하기 좋은 유용한 사이트 : [https://crontab.guru/](https://crontab.guru/){: target="\_blank" }

## ApplicationEventPublisher Mocking

현재 구현중인 어플리케이션에서 [`ApplicationEventPublisher`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationEventPublisher.html){: target="\_blank" }를 이용한 이벤트 형식으로 기능을 구현하고 있다.

유닛테스팅을 위해 `ApplicationEventPublisher`를 Mocking하여 테스트를 하려고 하였는데 왠일인지 테스트 코드가 정상적으로 동작하지 않았다. 이유를 찾아보니 이런 이슈를 볼 수 있었다. 

[https://github.com/spring-projects/spring-boot/issues/6060](https://github.com/spring-projects/spring-boot/issues/6060){: target="\_blank" }

스프링 프레임워크으 제약사항으로 인해 `ApplicationEventPublisher`는 Ragular Bean으로 등록되지 않는다는 것이다. 그래서 테스팅을 위해서 취할 수 있는 방법이 `SpyBean`을 활용하는 것이다. 아래 코드는 `SpyBean`을 활용하여 유닛테스트를 실행한 코드이다. [`kotest`](https://github.com/kotest/kotest){: target="\_blank" }와 [`SpringMockk`](https://github.com/Ninja-Squad/springmockk){: target="\_blank" }를 사용하였다.

```kotlin
@SpringBootTest(classes = [FooService::class])
@ContextConfiguration(classes = [FooService.TestConfig::class])
class FooServiceTest : FreeSpec() {

    override fun listeners() = listOf(SpringListener)

    @Autowired
    private lateinit var fooService: FooService

    @SpykBean
    private lateinit var publisher: ApplicationEventPublisher

    init {
        "create 함수는" - {
            "data를 입력받아" - {
                val dto = fixture<DataDto>()
                fooService.create(dto)
                "PackCreatedEvent 이벤트를 발행한다" {
                    verify { publisher.publishEvent(any<FooCreatedEvent>) }
                }
            }
        }
    }

    @TestConfiguration
    class TestConfig {
        @Primary
        @Bean
        fun eventPublisherSpy(): ApplicationEventPublisher {
            return spyk()
        }
    }
}
```

## ElementCollection

사용하는 Entity 중에서 Tag 기능을 구현하기 위해서 [`ElementCollection`](https://en.wikibooks.org/wiki/Java_Persistence/ElementCollection){: target="\_blank" }를 사용하였다. `OneToMany` 관계를 맺어주는 것과 비슷해 보이지만 `OneToMany` 관계를 맺을려면 관계를 맺어줄 Entity를 추가할 필요가 있는데 `ElementCollection`은 Entity를 추가할 필요가 없는 부분이 차이점이다.

## DynamicUpdate

Entity를 영속화시 update 쿼리 최적화를 위해서 [`DynamicUpdate`](https://docs.jboss.org/hibernate/orm/5.4/javadocs/org/hibernate/annotations/DynamicUpdate.html){: target="\_blank" }를 사용하였다. Dirty Checking에서만 동작할 줄 알았는데 Repository의 save 함수에서도 동일하게 동작함을 새롭게 알 수 있었다.

## JPA Auditing ZonedDateTime 이슈

JPA 사용시 등록일시, 수정일시 등 변경이력을 기록하기 위한 용도로 [`Auditing`](https://docs.spring.io/spring-data/jpa/docs/1.7.0.DATAJPA-580-SNAPSHOT/reference/html/auditing.html){: target="\_blank" }를 사용하였다. (등록일시와 수정일시를 많이들 사용하고 있으나 등록자나 수정자도 Spring Security를 연동하여 사용할 수 있다.)

현재 진행중인 프로젝트에서 Entity의 등록일시와 수정일시를 `ZonedDateTime`클래스로 사용하였는데, 기본적으로 JPA의 Auditing은 `LocalDateTime`을 사용하므로 별도 설정이 없으면 오류가 발생한다.

설정 방법은 아래와 같다. ([Stack overflow 참고](https://stackoverflow.com/questions/49170180/createdby-and-lastmodifieddate-are-no-longer-working-with-zoneddatetime){: target="\_blank" })

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.auditing.DateTimeProvider;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;
import java.time.ZonedDateTime;
import java.util.Optional;

@Configuration
@EnableJpaAuditing(dateTimeProviderRef = "auditingDateTimeProvider")
public class PersistenceConfig {

    @Bean // Makes ZonedDateTime compatible with auditing fields
    public DateTimeProvider auditingDateTimeProvider() {
        return () -> Optional.of(ZonedDateTime.now());
    }

}
```

## datadog

회사에서 서비스 이력추적이나 MSA간의 로그 분석을 위한 모니터링 도구로 [Datadog](https://www.datadoghq.com/lpg/?utm_source=Advertisement&utm_medium=GoogleAdsNon1stTier&utm_campaign=GoogleAdsNon1stTier-BrandCV&utm_content=Brand&utm_keyword=%2Bdata%20%2Bdog&utm_matchtype=b&gclid=EAIaIQobChMIipjotNTU6wIVZZ_CCh13XwyCEAAYASAAEgK99PD_BwE){: target="\_blank" }를 사용하기로 하였다. 별도 설정없이 기본적인 로그를 제공하긴 하지만 MSA 내에서 좀더 자세한 로그를 남기기 위해서는 별도의 `Logback`을 설정하여야 한다. 해당 설정방법은 연동 후에 적용방법을 별도로 적어볼 예정이다.

## JPA 타임존 변경

Spring Application과 DB의 타임존이 다른 경우에 대비하기 위해서 시간 관련 클래스를 `ZonedDateTime`으로 사용하였다. 이때 별도로 타임존 설정을 하지 않아도 알아서 타임존이 바껴서 DB에 저장될줄 알았는데 구현하고 보니 Application의 타임존에 맞게 DB에 시간값이 입력되는 것이었다. 그래서 Application과 DB가 타임존이 다른 경우 아래와 같이 설정을 추가하여야 한다.

```
## application.properties
spring.jpa.properties.hibernate.jdbc.time_zone=UTC
```

jdbc url에 `?serverTimezone=UTC`을 설정하는 경우를 보았는데 실제로 적용해보았으나 동작하지 않으니 주의하자.

## flyway 이슈

해결은 하였지만 아직 원인 파악을 하지 못한 케이스가 있어 적어본다.

JPA Auditing을 적용한 후 특정 테이블에 데이터 수정 시 계속헤서 등록일시가 변경되길래 확인해보니 DDL에 적어놓지도 않은 `on update current_timestamp()`가 추가되어 있는 것을 확인할 수 있었다. 해당 기능이 추가되지 않도록 이것저것 해보았는데 결국 원인은 찾지 못했고 해당 컬럼에 `default current_timestamp()`를 적용하니 `on update current_timestamp()`가 사라져서 문제를 해결할 수 있었다.

## Querydsl JPA custom repository 추가할때 네이밍 패턴

JPA Querydsl 사용할 시 custom repository를 추가하여 사용하는데, 이때 구현체를 추가할 때 명명 규칙이 있다. 바로 꼬릿글자로 `Impl`을 추가해 주어야 하는 것이다. ([관련 문서](https://docs.spring.io/spring-data/jpa/docs/2.1.3.RELEASE/reference/html/#repositories.custom-implementations){: target="\_blank" }) 

개인적으로는 구현체에 `Impl`이라는 글자를 붙이는 규칙을 별로 좋아하지 않는데, 정해진 규칙이니 어쩔수 없이 따라야 겠다.