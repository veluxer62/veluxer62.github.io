---
layout: single
title: "2021년 10월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2021-october-dev-log/
---

10월은 안심번호 서비스를 만들고 서버 이슈가 좀 있었고, 결혼식 참가등 사적인 일도 많이 있는 등 바쁜 한달을 보냈던것 같다.

안심번호 서비스는 내가 잘 할 수 있는 Kotlin + Spring으로 개발하였다. 처음에는 webflux와 MongoDB 스텍으로 개발하려고 하였는데 해당 스텍이 익숙하지 않는 동료들이 유지보수할때 어려움이 있을 것 같아서 MVC와 RDBMS로 스텍을 변경하였다. 예전이었다면 새로운 스텍을 사용하는 것에 주저함이 없었을 테지만 요즘엔 꼭 필요한 경우가 아니라면 주변 동료들이 유지보수에 용이한 스텍을 선택하는게 나아보인다고 생각이 들어 이부분은 많이 고려하는 편이다.

안심번호 서비스는 단순한 기능을 제공함에도 고민해야 될것들이 생각보다 많이 있었다. 그리고 오랜만에 Spring을 사용하다보니 이런저런 이슈도 많이 겪게 되었다. 덕분에 이런저런 공부도 많이 하게 되었다. 자세한 내용은 아래에서 다루어 보겠다.

쿠버네티스에 새로운 서비스를 추가하는 작업도 처음 해보았는데, 이전까지는 있는 코드를 복사하거나 일부를 수정하는 작업을 해오다가 처음부터 세팅을 하려고 하니 많이 해멨던것 같다. 동료의 도움을 받아서 세팅을 잘 마무리 했는데 덕분에 많이 배웠다.

아래는 10월동안 정리한 이슈 내용들이다.

## if-else switch 를 사용하기보다 팩토리를 사용하자

[엉클밥 선생님의 블로그](https://blog.cleancoder.com/uncle-bob/2021/03/06/ifElseSwitch.html){: target="\_blank" }에서 if-else와 switch중 어떤 걸 사용하는게 좋은 지에 대한 답변을 정리해 두신 글을 읽었다.

결론부터 말하자면 [Factory 메소드 패턴](https://ko.wikipedia.org/wiki/%ED%8C%A9%ED%86%A0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C_%ED%8C%A8%ED%84%B4){: target="\_blank" }을 사용하는 것이 좋다였다. 개인적으로는 질문의 의도에서 살짝 벗어난(?) 감은 있지만 저자의 의견은 if-else와 switch 중 어느것을 쓰는게 중요한 것이 아니라 저수준의 코드를 감추면서 고수준의 코드를 좀더 이해하기 쉽고 변경에 유연하도록 설계하는 것이 중요하다는 것이다.

이부분은 개인적으로 많이 공감하는 부분이라 후에 동료들에게 소개하면 좋을 것 같아서 기록해 둔다.

## Kotlin ULID 라이브러리

Long type의 Primary Key는 값의 크기가 한정적이고 중복 가능성이 커서 마이그레이션등의 이슈가 있어 단점이지만 순서를 가지기 때문에 정렬등으로 이용할때 성능상 장점이 있다. 반면 UUID는 고유값을 생성하기때문에 limit에 대한 고민을 하지 않아도 되고 중복 가능성이 적어 마이그레이션 시 유리한 반면 순서를 가지지 않기 때문에 PK를 이용한 정렬을 사용할 수 없다는 문제가 있다.

[ULID](https://github.com/ulid/spec){: target="\_blank" }는 UUID의 대체제로 순서를 보장하면서 유일한 값을 생성해준다. (자세한 특징은 링크된 사이트에서 확인하자.) 그래서 위 두개의 타입에 대한 대체제로 적합해 보여 사용하기로 했다.

현재 Python으로 개발하는 프로젝트에서는 [ulid-py](https://github.com/ahawker/ulid){: target="\_blank" }를 Kotlin으로 개발하는 프로젝트에서는 [kULID](https://github.com/JonasSchubert/kULID){: target="\_blank" }를 사용하고 있는데 나름 만족하고 사용하고 있다.

## flyway 환경별 설정

Spring을 이용한 개발시 마이그레이션 도구로 flyway를 사용하고 있다. 이번 안심번호 개발 시 특정 데이터를 환경별로 넣어줘야 하는 이슈가 있었는데, flyway에서 `locations`를 통해 환경별로 마이그레이션 폴더를 지정할 수 있어서 적용해 보았다.

```yml
#---
spring:
  config:
    activate:
      on-profile: dev
  flyway:
    locations: classpath:db/migration/common, classpath:db/migration/dev
#---
spring:
  config:
    activate:
      on-profile: prod
  flyway:
    locations: classpath:db/migration/common, classpath:db/migration/prod
```

단, 주의할점은 마이그레이션 파일의 버전 정책을 잘 잡아야 하는데, 공통 버전을 1, 2, 3, 4... 로 간다면 프로필별 마이그레이션은 1.1, 2.1, 2.2... 형태로 가야한다. 그렇지 않다면 마이그레이션 시 오류가 발생할 수 있다.

## Java 16 mockStatic 이슈

JAVA 16에서 `mockkStatic`을 이용하여 ZonedDateTime.now()를 고정하여 테스트를 수행하면 오류가 발생한다.
mockk API에서 최근에 이슈가 제기된것 같고 이슈 확인중이라고 한다. (11월 4일경 해당 이슈는 이슈 아님의 사유로 닫힌듯 하다.)

이슈 링크: [https://github.com/mockk/mockk/issues/681](https://github.com/mockk/mockk/issues/681){: target="\_blank" }

## test logger plugin

CI에서 테스트 로그가 남으면 좋을 것 같아서 찾아봤는데 괜찮은 플러그인이 있어서 적어본다. [Gradle Test Logger Plugin](https://github.com/radarsh/gradle-test-logger-plugin){: target="\_blank" }이다. 다양한 테마 및 설정 들을 제공해주는데 Nested Test에 대한 지원도 해주면 참 좋겠다.

## Circle CI OOM 이슈

Circle CI에서 Spring Boot 어플리케이션의 빌드 테스트를 실행시킬때 아래와 같이 오류가 발생하는 이슈가 있었다.

```
... 생략

* What went wrong:
Execution failed for task ':test'.
> Process 'Gradle Test Executor 4' finished with non-zero exit value 137
  This problem might be caused by incorrect test process configuration.
  Please refer to the test execution section in the User Manual at https://docs.gradle.org/7.2/userguide/java_testing.html#sec:test_execution

... 생략
```

별도로 테스트 코드가 실패하는 것도 아니고 간헐적으로 발생하길래 원인을 찾아보니 OOM 이슈였다. [StackOverflow 참고](https://stackoverflow.com/questions/38967991/why-are-my-gradle-builds-dying-with-exit-code-137){: target="\_blank" }

Heap 메모리 설정도 해보고 `--no-daemon` 설정도 해보았는데 근본적으로 해결이 되지 않아 결국 가상환경의 사양을 올려서 해결했다 ㅠ

## Mockk 라이브러리 사용 시 Spring Data의 `save` 함수 이슈

Mockk 라이브러리에서 JPA repository의 save 함수를 이용하여 테스트를 할때 오류가 발생한다.

```kotlin
test("안심번호 연결 함수는 전화번호가 주어지면 안심번호를 생성하고 연결이력을 저장한 후 안심번호를 반환한다.") {
    val originNumber = "01012341234"
    val safetyNumber = "050437973400"
    every { generator.generate(originNumber) } returns safetyNumber

    val actual = sut.linkSafetyNumber(originNumber)

    actual shouldBe safetyNumber
    verify {
        val entityMatch = match<SafetyNumberLinkageHistory> {
            it.originNumber == originNumber &&
                it.safetyNumber == safetyNumber &&
                it.status == ASSIGNED
        }
        repository.save(entityMatch)
    }
}
```

```
class java.lang.Object cannot be cast to class com.spoqa.safetynumberservice.domain.SafetyNumberLinkageHistory (java.lang.Object is in module java.base of loader 'bootstrap'; com.spoqa.safetynumberservice.domain.SafetyNumberLinkageHistory is in unnamed module of loader 'app')
java.lang.ClassCastException: class java.lang.Object cannot be cast to class com.spoqa.safetynumberservice.domain.SafetyNumberLinkageHistory (java.lang.Object is in module java.base of loader 'bootstrap'; com.spoqa.safetynumberservice.domain.SafetyNumberLinkageHistory is in unnamed module of loader 'app')
	at com.spoqa.safetynumberservice.domain.SafetyNumberLinkageHistoryRepository$Subclass5.save(Unknown Source)
	at com.spoqa.safetynumberservice.domain.SafetyNumberLinkageHistoryRepository$Subclass5.save(Unknown Source)
	at com.spoqa.safetynumberservice.domain.SimpleSafetyNumberLinkageService.linkSafetyNumber(SafetyNumberLinkageService.kt:21)
	at com.spoqa.safetynumberservice.domain.SimpleSafetyNumberLinka

... 이하생략
```

이런 경우 `every`를 통해 해결이 가능하다.

```kotlin
test("안심번호 연결 함수는 전화번호가 주어지면 안심번호를 생성하고 연결이력을 저장한 후 안심번호를 반환한다.") {
    val originNumber = "01012341234"
    val safetyNumber = "050437973400"
    every { generator.generate(originNumber) } returns safetyNumber

    // 여기를 추가하자!
    every { repository.save(any()) } returns SafetyNumberLinkageHistory(originNumber, safetyNumber, ASSIGNED)

    val actual = sut.linkSafetyNumber(originNumber)

    actual shouldBe safetyNumber
    verify {
        val entityMatch = match<SafetyNumberLinkageHistory> {
            it.originNumber == originNumber &&
                it.safetyNumber == safetyNumber &&
                it.status == ASSIGNED
        }
        repository.save(entityMatch)
    }
}
```

verify로만 테스트를 검증할 수 있음에도 every를 사용하는게 조금 맘에 들진 않지만 예상컨데 Spring Data의 `save`함수가 Generic을 이용해서 매개변수의 타입을 추론하기 때문으로 추측된다.
`hint`를 통해서 해결할 수 있다고는 하는데 이것도 가이드 대로 해보니 잘 안되어서 일단 넘어가야겠다. 관련한 이슈는 제기되어 있는 상태인데 2019년에 제기된 이슈가 아직도 해결되지 않은걸로 봐서는 쉽지 않은 듯 하다.

관련 이슈: [https://github.com/mockk/mockk/issues/321](https://github.com/mockk/mockk/issues/321){: target="\_blank" }

## Kotest Trasactional 이슈

예전에도 한번 밟은것 같은데 Transactional 이 필요한 환경에서는 Nested Test를 쓸수 없다. 코루틴 때문이지 않을까 생각되는데...어쩔수 없지뭐...

관련 이슈: [https://github.com/kotest/kotest/issues/950](https://github.com/kotest/kotest/issues/950){: target="\_blank" }

## Intellij show heap

[Intellij에서 heap 메모리 현황을 보는 방법](https://www.jetbrains.com/help/idea/increasing-memory-heap.html){: target="\_blank" }

매번 까먹는것 같다. 적어놓고 기억해둬야지.

## JPA Entity isNew 활용법

JPA에서 Entity를 save할때 ID가 null이거나 0인경우(Long 타입에 해당)가 아니라면 무조건 새로운 데이터를 생성하는 로직으로 구현하려고 하더라도 DB 채번을 실행한다. 이는 `isNew` 함수를 오버라이딩 함으로써 해결할 수 있는데, 아래 예시와 같이 `Pesistable`인터페이스를 구현하여 작성하면 된다.

```kotlin
@Entity
class SafetyNumberLinkageHistory(
    originNumber: String,
    safetyNumber: String,
    status: SafetyNumberLinkageStatus,
) : Persistable<String> {
    @Id
    private val id: String = ULID.random()

    @Column(nullable = false)
    var createdAt: ZonedDateTime = ZonedDateTime.now()
        protected set

    @Column(nullable = false, length = 50)
    var originNumber: String = originNumber
        protected set

    @Column(nullable = false, length = 50)
    var safetyNumber: String = safetyNumber
        protected set

    @Column(nullable = false, length = 100)
    @Enumerated(STRING)
    var status: SafetyNumberLinkageStatus = status
        protected set

    override fun getId(): String = id
    override fun isNew(): Boolean = true
}
```

## 테라폼에서 RDS 비밀번호 KMS로 하기

테라폼에서 RDS의 비밀번호를 생성할때 KMS를 이용하여 비밀번호를 암호화 하려고 하는 경우 아래과 같이 하면된다.

```shell
$ echo -n 'master-password123!' > plaintext-password

$ aws kms encrypt \
--key-id $key_id \
--plaintext fileb://plaintext-password \
--output text --query CiphertextBlob

$ aws kms encrypt \
--key-id $key_id \
--plaintext fileb://plaintext-password \
--output text --query CiphertextBlob
```

실행 후 반환된 암호화된 문구를 Terraform에 설정하면 된다.

참고: [ryanpark.dev님 블로그](https://ryanpark.dev/419fe316-5049-4c03-bc50-87582855faa1){: target="\_blank" }

## AWS mariadb parameter group family 조회방법

Terrform에서 MariaDB RDS 설정 시 parameter group family를 참고해야 할 일이 있었는데 기록을 위해 이때 사용했던 스크립트를 적어본다.

```
aws rds describe-db-engine-versions --query "DBEngineVersions[].DBParameterGroupFamily" --engine mariadb
```

## Intellij EOF 설정

Lint나 Github에서 PR시 파일에 EOF 경고 문구가 표시되지 않도록 하기 위해 매번 직접 파일 끝에 엔터를 입력하는게 번거로워 Intellij 설정을 찾아보니 다행히 해당 설정이 존재했다.

Preferences > Editor > General 메뉴에서 `Ensure every saved file ends with a line break`를 체크하면 된다.

![line-break-preference](/assets/images/retrospective/line-break-preference.png)

## WebClient Block 이슈

Webflux에서 suspend function이나 Java Reactor를 사용하지 않고 일반함수를 사용하더라도 Webclient의 block 함수를 사용 시 오류를 반환한다. 원인을 좀 찾아보니 서브루틴에서 blocking을 호출하면 예외를 던지도록 설계되어 있다고 한다. 이문제는 아직 깊이 뜯어보지 못했다. 좀더 찾아봐야 할 것 같다.

꼭 필요해서 non-blocking api를 만들어야 하는 경우가 아니라면 맘편하게 WebMVC를 사용하자...

## Jackson에서 날짜 형식이 빈값으로 넘어올 때

외부 API에서 날짜 형태의 Request Body를 수신할 때 아래와 같이 DTO 객체를 정의할 수 있다.

```kotlin
data class CallLogDto(
    @JsonProperty("ss_time")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    val serviceStartAt: LocalDateTime,

    @JsonProperty("se_time")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    val serviceEndAt: LocalDateTime,
)
```

해당 Request Body에서 날짜 값이 빈값으로 입력될 수도 있는 상황에서는 어떻게 해야 할까? 바로 `JsonInclude`를 선언해주면 된다.

```kotlin
data class CallLogDto(
    @JsonProperty("ss_time")
    @JsonInclude(value = JsonInclude.Include.NON_EMPTY)
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    val serviceStartAt: LocalDateTime?,

    @JsonProperty("se_time")
    @JsonInclude(value = JsonInclude.Include.NON_EMPTY)
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    val serviceEndAt: LocalDateTime?,
)
```

단, 빈값인 경우 null로 변환되기 때문에 nullable로 타입을 바꿔줘야 한다.

## Spring Rest Controller에서 content-type을 text/plain으로 수신하는 경우

그럴일이 많지 않지만 부득이하게 Spring Boot에서 api를 content-type을 text/plain으로 하려면 @requestbody의 객체 타입을 String으로 해야한다. 다른 객체 타입은 수신하지 못한다.

```kotlin
@RestController
class SampleController {
    @PostMapping("/sample", produces = [MediaType.TEXT_PLAIN_VALUE], consumes = [MediaType.TEXT_PLAIN_VALUE])
    fun sample(@RequestBody data: String) {
        println(data)
    }
}
```

`ContentNegotiationConfigurer`을 이용해서 변환되도록 할 수 있을 것 같은데, 아직 안해봐서 설정해보고 잘되면 적어봐야겠다.

## Custom Jackson Mapper 날짜타입 이슈

jackson 쓸대 localDateTime 파싱 에러 뜨면 JavaTimeModule을 추가하면된다.

```kotlin
val mapper = jacksonMapperBuilder()
            .addModule(JavaTimeModule())
            .build()
```

## Graphql 사용시 동일한 필드에 여러 타입을 다루고자 하는 경우

Graphene 라이브러리를 이용하여 Graphql의 동일한 필드에 여러 타입을 두는 스키마를 다루고자 하는 경우 아래와 같이 필드를 정의하면 된다.

```python
class AdvertisementVendorBannerField(Interface):
    image_url = String(required=True, description='이미지 URL')
    action_type = Field(AdvertisementVendorBannerActionTypeEnum, required=True, description='배너 선택 시 행동 유형')


class LinkTypeAdvertisementVendorBannerField(ObjectType):
    class Meta:
        interfaces = (AdvertisementVendorBannerField,)

    action = Field(AdvertisementVendorBannerLinkActionField, required=True, description='이동 링크')


class GalleryTypeAdvertisementVendorBannerField(ObjectType):
    class Meta:
        interfaces = (AdvertisementVendorBannerField,)

    action = Field(AdvertisementVendorBannerGalleryActionField, required=True, description='이미지 정보')
```

Graphql Query는 아래와 같이 작성하면 된다.

```graphql
query {
  banners {
    imageUrl
    actionType
    __typename
    ... on LinkTypeAdvertisementVendorBannerField {
      action {
        link
      }
    }
    ... on GalleryTypeAdvertisementVendorBannerField {
      action {
        imageUrls
      }
    }
  }
}
```
