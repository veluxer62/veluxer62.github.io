---
layout: single
title: "2022년 9월 개발일지"
categories:
  - RETROSPECTIVE
tags:
  - 개발일지
toc: true
---

처음으로 코로나에 걸렸다. 코로나가 유행한지 3년이 다되어가는데 이때까지 걸리지 않길래 나는 슈퍼유전자인가 내심 뿌듯해하고 있었는데 이번에 걸리면서 그냥 운이 좋았던걸로 밝혀졌다. 요즘엔 역학조사를 빡세게 하지도 않기 때문에 어디에서 정확하게 감염이 되었는지 알지도 못하겠다. 잠복기가 있다고 하니 한참전에 감염되었다가 증상이 나타나는 것일수도 있다. 주말에 증상이 나타났는데 그냥 오한이들고 잠이 너무 오길래 집에서 푹 쉬었었다. 하지만 월요일에 자가키트로 진단해보니 양성이 나와 병원에서 최종적으로 코로나 감염 확인을 받고 집에서 자가 격리를 하였다.

코로나가 유행하는 초기에는 같이 일하던 동료가 감염확인 되었을 때에도 집에서 자가격리를 했었다. 휴대폰 어플을 통해 격리를 잘 하고 있는지 감시도 하였고 일주일 정도 먹을거리도 보내주고 나중에 동사무소에서 격리로 인한 지원금(?)도 받을 수 있었다. 하지만 요즘에는 확진이 받았다고해서 지원금은 커녕 격리 중에도 먹을거리를 사는등 특별한 이유가 있으면 외부활동을 할 수 있다고 한다. 이제 점점 감기처럼 인식을 하려고 하나보다. 성동구에서는 코로나 감염시 햇반이나 죽, 간식거리등을 한박스 보내 주었다. 다른 구에서도 그런가 지인들에게 물어보니 그렇지 않다고 하길래 구마다 다른가보다. (역시 성동구!)

하필이면 추석 연휴에 코로나에 감염되어서 이번 추석은 시골에 가지 않고 집에서 보내게 되었다. 명절때마다 교통체증으로 인해 고생을 하였는데 이번엔 그런 고생을 하지 않아도 되어서 다행이다라고 생각하면서도 자주 찾아뵙지 못하는데 명절에도 못가게 되니 부모님께 죄송스럽기도 하였다.

<br/>

최근에 KPI를 달성하기 위해 영업부서의 요청으로 인해 영업부서의 요구사항을 가장 높은 우선순위로 처리해주는 것으로 팀 운영방식이 바뀌었다. 그렇게되면서 여러가지 기능을 산발적이고 동시다발적으로 개발해야 하는 상황으로 개발 진행방식이 바뀌게 되었는데, 그러다보니 백엔드 챕터에서도 전체적인 작업의 현황관리 및 우선순위를 결정해야하는 필요가 생기게 되었다.

챕터장으로 활동하면서 언어 전환이 끝나고 주문/채팅 프로젝트를 거치게 되면서 굵직한 결정과 작업 현황 및 일정관리를 주로 해오다가 주문/채팅 프로젝트가 끝나게 되면서 큰 프로젝트가 3개로 나뉘게 되면서 프로젝트 3개 모두 관리를 하는 것이 현실적으로 불가능하다고 판단해서 최근까지는 챕터의 관리보다는 내가 맡은 프로젝트에 집중하고 이슈에 대한 굵직한 결정만 해주는 역할만 수행하고 있었다.

하지만 다시 팀의 운영방식이 바뀌게 되면서 챕터원 전체의 업무관리를 수행해야 하는 관리방식으로 변하게 된 것이다. 이러한 상황을 겪으면서 이전까지는 조직의 장은 하나의 매니징 방식을 쭉 유지할것이라 생각했는데 관리 방식은 회사의 상황에 따라, 프로젝트에 따라서 계속 변화하는구나라는 생각이 들었다.

개인적으로는 매니저는 팔방미인이어야 한다고 생각한다. 물론 개발자라면 자기가 속한 챕터의 개발적인 깊이도 있어야 한다. 하지만 개발만 잘해서는 관리자로 생활하기는 쉽지 않을거란 생각이 든다. 회사의 상황에 맞게 챕터의 운영방식을 유연하게 변화시키고 구성원들이 업무를 함에 있어 방해요소를 최대한 줄이는 것이 매니저의 역할이지 않을까. 아직 부족한게 많고 지금과 같은 이슈들을 겪으면서 많은 것들을 배우고 느끼고 있다. 앞으로도 많은 이슈들이 생길 것이고 실수도 하고 실패도 하면서 많은 것들을 배울 수 있으면 좋겠다.

아래는 9월동안 정리한 이슈 내용들이다.

## mockMvc를 사용할 때 Test Double을 주입시켜주는 방법

`@WebMvcTest`로 Controller에 대한 테스트 코드를 작성할때 버릇처럼 `@MockBean`을 사용해왔다. 하지만 이전에 `@MockBean`을 사용하면서 테스트 코드가 많아지면 많아질수록 테스트 실행속도가 급격하게 길어지는 이슈를 겪으면서 `@WebMvcTest`로 Controller에 대한 테스트 코드 작성 시 `@MockBean`을 활용하지 않고 다른 방법으로 테스트 하는 방법을 생각하게 되었다.

`TestConfiguration`은 테스트 코드에서만 사용되는 설정으로 운영 코드에 정의되어있는 `fooService`의 빈을 테스트가 실행되는 동안 대체해서 주입시켜준다. 아래 코드를 보자.

```kotlin
@RestController
class FooController(
    private val fooService: FooService
) {
    @GetMapping("/test")
    fun test(): String = fooService.test()
}

@Service
class FooService {
    fun test() = "test"
}

@WebMvcTest
internal class FooControllerTest {
    @Autowired
    private lateinit var mockMvc: MockMvc

    @Autowired
    private lateinit var fooService: FooService

    @Test
    fun test1() {
        // Given
        Mockito.`when`(fooService.test()).thenReturn("test1")

        // When
        val actual = mockMvc.get("/test").andReturn()

        // Then
        assertEquals("test1", actual.response.contentAsString)
    }

    @TestConfiguration
    class TestConfig {
        @Bean
        @Primary
        fun fooService(): FooService = Mockito.mock(FooService::class.java)
    }
}
```

다른방법으로는 `@Mock`과 `@InjectMock`을 활용하는 방법이다. 이 방법은 `MockMvc`를 직접 생성해주어야 해서 코드가 장황해지는 경향이 있으나 테스트 실행시 Spring에서 사용하는 Bean들을 많이 로드하지 않으므로 가볍다는 장점이 있다. 아래 코드를 보자

```kotlin
@ExtendWith(MockitoExtension::class)
internal class FooControllerTest {
    @Mock
    private lateinit var fooService: FooService

    @InjectMocks
    private lateinit var fooController: FooController

    private lateinit var mockMvc: MockMvc

    @BeforeEach
    fun setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(fooController).build()
    }

    @Test
    fun test1() {
        Mockito.`when`(fooService.test()).thenReturn("test1")

        val actual = mockMvc.get("/test")
            .andReturn()

        assertEquals("test1", actual.response.contentAsString)
    }
}
```

둘다 일장일단이 있으니 상황에 맞게 쓰면좋겠다. 프로젝트 내 Controller 설정에 대한 다른 의존성이 없고 순수하게 Controller에 대한 동작 테스트만 한다면 MockitoExtension만을 사용하는 아래 방식이 가볍고 빠르므로 좋아보이고 그외 다른 Spring 관련 기능들을 사용해야 한다면 위에 방식을 사용할 수 있겠다.

## Test Double

테스트 더블은 매번 볼때마다 헷갈리고 정확한 정의를 내리기가 모호한 경우가 많다. 아마도 내가 정확하게 이해를 하고 있지 않기 때문이라 생각된다. 최근 글을 쓰다가 테스트 더블에 대해 잘 정리된 글을 발견하게 되어 소개해보고 간단하게 요약해보고자 한다.

[https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/](https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/){: target="\_blank" }

테스트 더블은 여러가지가 있다. 위 글을 토대로 종류별로 간단하게 정리만 해보고자 한다.

- Dummy: 가짜 객체로 인스턴스화된 객체가 필요하지만 기능은 필요하지 않는 경우 사용.
- Fake: 동작의 구현을 가지고 있지만 실제 프로덕션에는 적합하지 않은 기능을 수행.
- Stub: 테스트에서 호출된 요청에 대해 미리 준비해둔 결과를 제공.
- Spy: Stub 역할을 가지면서 실제 객체처럼 사용할수도 있음. 특정 함수의 호출여부를 기록함.
- Mock: 호출에 대한 기대를 명세하고 내용에 따라 동작하도록 프로그래밍된 객체.

가만히 살펴보면 위에서부터 아래로 갈수록 테스트 더블이 제공하는 기능이 많아지고 복잡해짐을 볼 수 있다. 지금까지 거의 Mock 객체만 주로 사용해왔는데 다른 테스트 더블도 사용해보면 유용할듯하다.

## Factory 함수명

[이펙티브 코틀린](https://product.kyobobook.co.kr/detail/S000001033129){: target="\_blank" }을 읽다가 팩토리 함수명에 주로 사용하는 네이밍이 있다는것을 알게되어 적어둔다. 사실 구글링을 하거나 프레임워크의 코드를 보면서 팩토리 함수의 네이밍에 대해 이해가 되지 않거나 왜 `of`와 같은 함수명을 지었는지 궁금했었는데 관행이었다는 것을 이번에 알게되어서 재미있었다.

### from

파라미터를 하나 받고 같은 타입의 인스턴스 하나를 리턴하는 타입변환함수를 나타냄.

```kotlin
val date: Date = Date.from(instant)
```

### of

파라미터를 여러개 받고, 이를 통합해서 인스턴스를 만들어주는 함수를 나타냄.

```kotlin
val faceCards: Set<Rank> = EnumSet.of(JACK, QUEEN, KING)
```

### valueOf

`from` 또는 `of`와 비슷한 기능을 하면서도 의미를 조금 더 쉽게 읽을 수 있게 이름을 붙인 함수.

```kotlin
val prime: BigInteger = BigInteger.valueOf(Integer.MAX_VALUE)
```

### instance 또는 getInstance

싱글턴으로 인스턴스 하나를 리턴하는 함수. 파라미터가 있을 경우 아규먼트를 기반으로 하는 인스턴스를 리턴함. 일반적으로 같은 아규먼트를 넣으면 같은 인스턴스를 리턴하는 형태로 동작.

```kotlin
val luke: StackWalker = StackWalker.getInstance(options)
```

### createInstance 또는 newInstance

`getInstance`처럼 동작하지만 싱글턴이 적용되지 않아서 함수를 호출할때마다 새로운 인스턴스를 만들어서 리턴.

```kotlin
val newArray = Array.newInstance(classObject, arrayLen)
```

### getType

`getInstance`처럼 동작하지만 팩토리 함수가 다른 클래스에 있을때 사용하는 이름. 타입은 팩토리 함수에서 리턴하는 타입.

```kotlin
val fs: FileStore = Files.getFileStore(path)
```

### newType

`newInstance`처럼 동작하지만 팩토리 함수가 다른 클래스에 있을 때 사용하는 이름. 타입은 팩토리 함수에서 리턴하는 타입.

```kotlin
val br: BufferedReader = Files.newBufferedReader(path)
```

## Gradle dynamic version

Python이나 node진영에서 버전관리 도구를 사용할 때 dynamic version을 사용하는 것을 종종 볼 수 있었다. 그런데 지금까지 스프링을 사용하면서는 Gradle이나 Maven에서는 dynamic version을 사용하는 경우를 보지는 못햇는데 아마도 빌드시마다 버전이 변경될 수 있다는 점은 위험성이 있으니 그러리라 생각된다.

하지만 나는 Maven Repository에 있는 라이브러리들은 어느정도 Semantic Versioning의 룰을 잘 지켜주고 있다고 생각하기에 fetch version의 경우에는 dynamic version 기능을 적용해도 괜찮지 않을까 생각이든다.

[Gradle의 dynamic version](https://docs.gradle.org/current/userguide/dynamic_versions.html#sub:declaring_dependency_with_dynamic_version){: target="\_blank" }은 아래와 같이 적용할 수 있다.

```kotlin
dependencies {
    implementation("org.springframework:spring-web:5.+")
}
```

문서내에서도 아래와 같이 경고를 하고 있으니 회사 코드에 적용할 땐 위험에 대해 인지를 하고 사용하면 좋겟다.

> Using dynamic versions in a build bears the risk of potentially breaking it. As soon as a new version of the dependency is released that contains an incompatible API change your source code might stop compiling.

## ULID Creator 5.x.x 버전

ULIC Creator가 5.x.x버전으로 업그레이드 된것을 확인하게 되었다. API의 큰 변경은 없는것 같은데 어떤게 바꼇나 보니 신뢰성과 속도를 상당히 개선한 것으로 보인다.

[https://github.com/f4b6a3/ulid-creator/issues/20](https://github.com/f4b6a3/ulid-creator/issues/20){: target="\_blank" }

## PostgreSQL 유일 제약 조건 설정 시 주의사항

PostgreSQL에 다중 컬럼에 대해 유일 제약 조건을 설정할 때 nullable 컬럼에 대해서는 제약조건이 설정되지 않는다. 그래서 만약 다중 컬럼에서 유일 제약 조건을 설정하려면 아래와 같이 `NULLS NOT DISTINCT`를 함께 적어줘야 한다.

```
ALTER TABLE foo
ADD CONSTRAINT foo_uk UNIQUE NULLS NOT DISTINCT (a, b, c);
```

다만 PostgreSQL 버전이 15 버전 이상인 경우에만 위 쿼리를 사용할 수 있고 15 버전 미만인 경우에는 아래와 같이 해주어야 한다.

```
CREATE UNIQUE INDEX foo_3col_uk ON foo (a, b, c)
WHERE b IS NOT NULL;

CREATE UNIQUE INDEX foo_2col_uk ON foo (a, c)
WHERE b IS NULL;
```

유일 제약조건 걸때 놓치기 쉬운 조건이므로 잘 챙겨두자.
