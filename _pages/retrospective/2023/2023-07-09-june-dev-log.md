---
layout: single
title: "2023년 6월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2023-july-dev-log/
---

6월 중순이 넘어가면서부터 날씨가 많이 더워졌다. 이제는 너무 더워져서 매일 출퇴근시 타던 자전거를 더이상 못타고 있다. 타려면 아침일찍 출발해서 탈순 있지만 땀을 많이 흘린 상태에서 씻지도 않고 일하면 주변에 민폐가 될것 같아서 안하고 있다. 아내가 같이 헬스장을 다니자고 하지 않았더라면 회사 주변의 헬스장을 등록하고 출근할때마다 씻고 출근할 수 있을텐데 이부분은 조금 아쉬운 마음이다.

어서빨리 날씨가 시원해져서 다시 자전거로 출퇴근할 수 있었으면 좋겠다.

<br/>

달력을 보니 9월~10월에 황금연휴가 많아보인다. 이때 날씨도 시원할 때라 아직 가보지못한 자전거 종주길들을 가볼 수 있을 것같다. 올해 목표로했던 그랜드 슬램 달성을 위해서 바쁘더라도 꼭 다녀와야지. 내년은 국내에서 개최하는 자전거 대회에 나가보는걸 목표로 해보아야 겠다.

<br/>

잇몸이 내려앉아서 오랜만에 치과를 가보니 학생때 신경치료 후 크라운으로 씌웠던 치아의 뿌리가 썩어서 잇몸이 다 녹았다고 한다. 그래서 발치 후 임플란트를 해야한다고 한다 ㅠㅠ 그동안 건강검진 때 간단하게 치아 검진을 받아만 왔었고 항상 큰 충지 없다고 진단을 받았었는데, 알고보니 잇몸에서 계속 썩어가고 있었던 것이다. 이래서 치과를 자주가야한다고 하나보다. 이가 썩어서 치과에 큰돈이 들어갈때만 이러한 다짐을 하는거 같긴하다 ㅎㅎ

동네 치과에서 진단받고 아내와 이야기도할 겸 고민 후 결정한다고 나왔는데 아내가 기왕이면 임플란트 치료를 잘하는 곳으로 가보라고 해서 강남에 유명한 곳으로 갔다. 사람이 많아 대기시간이 길고 뭔가 정신없긴 하였지만 동네 치과보다 훨씬 체계적이고 견적가도 훨씬 저렴해서 강남에 있는 유명한 치과에서 하기로 결정했다. 역시 사람들이 많이 가는 곳에는 이유가 있는것 같다.

그나저나 술을 많이 마시면 치아에 좋지 않다는 이야기가 있다. 최근 지인들과 술자리를 가질일이 많아 술을 자주 마셨는데 술을 조금 줄여야 겠다.

<br/>

다음달에 여러 프로젝트를 모아서 한번에 배포하는 큰 배포건이 있다. 여러프로젝트들이 우선순위에 의해서 배포일이 밀리다보니 어찌하다 배포가 커진것이다. 그래서 QA도 거의 전체 검증 형태로 테스트르 할 것 같아서 이번에 미루고 미루었던 Spring Boot를 3버전으로 올리기로 마음먹었다. 이전에 막 3.0.0이 나왔을 때 시도해보았지만 Spring Cloud에서 대응이 거의 되어있지 않아서 접었었다. 현재는 3.1.1버전으로 어느정도 안정기에 접어들었다고 생각이 들어서 버전을 업그레이드 하기로 마음먹었다.

사실 Spring Boot 2 버전을 사용해도 문제가 없긴하다. 팀장님이나 개발과 관련없는 챕터에서는 굳이 큰 배포건에 리스크를 가중시킬 필요가 있는가에 대한 불만이 있을 수도 있다고 생각한다. 하지만 나는 제품의 성능과 품질을 향상시키기 가장 쉬운 방법이 버전업그레이드라고 생각한다. 남들이 고민해서 향상시켜둔 것들을 버전만 업그레이드 하면 누릴 수 있으니 얼마나 좋은가. 그래서 나는 짬이 날때마다 종종 버전을 업그레이드하곤 하였다. 하지만 Spring Boot 3 으로 업그레이드 하지 못하면서 한동안 의존성 버전을 적극적으로 올리지 못하였다. 왜냐하면 최신버전은 Spring Boot 3에서 지원하는 경우가 종종 있었기 때문이었다.

여러가지 우여곡절(?)이 있었지만 다행히 테스트 서버에 큰 문제없이 배포할 수 있었고 앞으로 QA할 일만 남았다. 업그레이드 하면서 겪었던 이슈들은 차차 적어가 보겠다.

<br/>

아래는 6월동안 정리한 이슈 내용들이다.

## Swagger에서 Pageable 매개변수를 표현하는 방법

Swagger에서 `Pageable`을 매개변수로 받는 API를 표현할 때 아래와 같이 JSON입력 값으로 표시되는 현상을 발견할 수 있다.

![swagger-pageable-before](/assets/images/retrospective/swagger-pageable-before.png)

이렇게 표시되어도 JSON에 값을 제대로 입력한다면 원하는대로 API를 호출할 순 있다. 하지만 좀더 사용자 친화적으로 Swagger 문서를 표현하고 싶다면 `@PageableAsQueryParam`을 사용해보자.

단, 해당 어노테이션을 적어주어도 여전히 위 사진과 같이 JSON 입력항목이 표시되니 `@Parameter(hidden = true)`을 pageable 매개변수에 선언해주어야 한다.

```kotlin
@GetMapping
@PageableAsQueryParam
fun getList(
    @Parameter(hidden = true)
    pageable: Pageable,
): PageDto<FooDto> {
    //... 생략
}
```

위와 같이 설정해주면 아래 사진처럼 문서가 나타남을 볼 수 있다.

![swagger-pageable-after](/assets/images/retrospective/swagger-pageable-after.png)

참고: [https://github.com/springdoc/springdoc-openapi/issues/317](https://github.com/springdoc/springdoc-openapi/issues/317){: target="\_blank" }

## Swagger에서 중첩 DTO를 표현하는 방법

API개발을 하다보면 자주 접하게 되는 문제가 바로 DTO의 네이밍이다. `ListDto`, `ViewDto`, `CreationCommand` 등과 같이 서로 다른 Context임에도 유사한 행위들로 인해서 `PersonListDto`, `PersonViewDtom`, `PersonCreationCommand`, `OrganizationListDto`,
`OrganizationViewDto`, `OrganizationCreateCommand` 등과 같이 장황한 클래스명들을 만들어가곤 한다.

이름을 고민하고 장황하기까지한 DTO들을 만들기 싫어서 아래와 같이 Controller안에 중첩 클래스로 DTO들을 구현해보았다.

```kotlin
class PersonController {
    // ... 생략

    data class ListDto(
        val id: UUID,
        val name: String,
        // ... 생략
    )

    data class ViewDto(
        // ... 생략
    )

    data class CreationCommand(
        // ... 생략
    )
}

class OrganizationController {
    // ... 생략

    data class ListDto(
        val id: UUID,
        val name: String,
        // ... 생략
    )

    data class ViewDto(
        // ... 생략
    )

    data class CreationCommand(
        // ... 생략
    )
}
```

어차피 위 DTO들은 View 레이어에서 사용될 DTO들이라 재사용성이 낮고 위에서 고민했던 네이밍에 대한 고민을 상당수 줄일 수 있게 된다. 막약 테스트코드를 작성할 때 필요하다면 아래와 같이 명시적으로 문맥을 표현해줄 수도 있다.

```kotlin
fun test() {
    // ... 생략
    val actual = testClient.get<PersonController.ListDto>(
        uri = url,
    )
}
```

다만 이렇게 사용하는 경우 Swagger를 사용할 때 주의가 필요하다. 아무런 설정없이 중첩된 클래스를 사용하는 경우 Swagger에서 오류는 발생하지 않지만 마지막에 읽어들인 클래스의 유형에 맞게 Schema가 덮어쓰이게 된다. 그래서 예상치 못하게 정의한 것과 다른 API 문서가 만들어질 가능성이 있다.

그래서 이렇게 중첩클래스를 사용하는 경우 아래와 같이 설정을 해주어서 중첩된 클래스명 전체가 Schema로 정의되도록 할 수 있다.

```yml
springdoc:
  use-fqn: true
```

참고: [https://github.com/springdoc/springdoc-openapi/issues/548](https://github.com/springdoc/springdoc-openapi/issues/548){: target="\_blank" }

## float에서 double로 변환시 이슈

float타입을 double타입으로 변홚시켜주기 위해 `123.456f.toDouble()`을 사용하면 예상했던 `123.456`값으로 변환되지 않고 `123.45600128173828`로 변환되는것을 볼 수 있다. 그래서 이러한 경우를 방지하기 위해서는 float타입을 double로 변환해줄때 문자열로 한번 변환해주고 변경해주는게 좋다.

```kotlin
123.456f.toString().toDouble()
// 123.456
```

이런 코드는 함수로 정의해두면 더 유용하게 사용할 수 있다.

```kotlin
fun Float.toDoubleExactly(): Double = this.toString().toDouble()
```

## Repository 패턴과 DI(Dependency inversion) 원칙

회사에서 팀원이 BigQuery를 이용한 데이터 저장 구현체를 Client로 표현한 사례가 있었다. 외부 API와 통신하는 용도로 주로 Client를 사용해왔는데 BigQuery도 하나의 외부 API로 본것이었다.

하지만 나는 BigQuery도 DB의 한 종류라고 생각하기 때문에 Client 보다는 Repository로 보는게 더 좋다고 생각했다. 그래서 Client를 Repository로 변경해주기를 요청했다.

참고: [https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design){: target="\_blank" }

리펙터링한 내용을 보면서 추가적인 의견을 내게 되었는데 바로 클래스의 패키지 위치였다. 현재 우리 서비스는 Application과 Domain으로 크게 두개의 패키지로 나누고 Application이 Domain을 의존하고 있는 형태를 가지고 있다. Application은 주로 Presentation과 외부의존성 등을 다루고 Domain은 주로 내부 비지니스 로직을 다은 객체인 Service, Repository, Entity등을 두었다.

여기서 팀원이 Domain 로직을 담은 Service를 외부 의존성을 가진 BigQueryRepository를 바라보게 구현한것이었다. 의존성이 반대방향을 바라보고 있었기에 Repository 인터페이스를 두고 Service는 Repository 인터페이스를 바라보도록 하고 BigQueryRepository는 Repository 인터페이스를 구현하도록 하는 의존성 역전을 제안하였다.

참고: [https://en.wikipedia.org/wiki/Dependency_inversion_principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle){: target="\_blank" }

비록 동료가 나의 리뷰로 인해 전체적인 구조를 많이 변경하고 리펙터링을 해야했지만 이러한 활동이 코드리뷰의 힘이 아닐까라는 생각이 든다. 덕분에 나도 알고 있었던 디자인패턴을 다시한번 상기할 수 있었다.

## 엘레베이터 피치

엘리베이터 피치란 엘리베이터를 타고 올라가는 시간만큼 짧은 시간(약 30~50초) 내에 중요한 이야기를 간단하게 요약하여 설명하는 것을 하는데 주로 투자를 받기 위해 투자자에게, 승인 또는 보고를 하기 위해 결정권자에게 발표할 때 사용한다고 한다.

최근에 사업을 시작하는 친구가 엘레베이터 피치를 위한 영상을 만들었다고 보여주었는데 처음 들어보는 단어라 기억해두려고 적어본다. 

참고: [https://www.jobkorea.co.kr/goodjob/tip/View?News_No=19824](https://www.jobkorea.co.kr/goodjob/tip/View?News_No=19824){: target="\_blank" }

## Spring Data Redis로 저장한 캐시 데이터를 CLI로 조회하는 방법

spring data redis를 이용하여 entity를 저장한다음 redis cli를 통해 조회하려면 아래와 같이 조회하면 된다.

```
HGETALL {entityName}:{id}
```

example
```
127.0.0.1:6379> keys *
1) "UserPermissionCache"
2) "UserPermissionCache:f2034bb3-405f-4417-9f2d-4bf451fb529f"
127.0.0.1:6379> hgetall UserPermissionCache:f2034bb3-405f-4417-9f2d-4bf451fb529f
 1) "_class"
 2) "com.tatum.backenduser.domain.user.UserPermissionCache"
 3) "groupIds.[0]"
 4) "b6ee4c65-3d86-4482-b63a-91ed05155e82"
 5) "groupIds.[1]"
 6) "ec51b5b3-7424-43a5-b606-37bfced6bef1"
 7) "groupIds.[2]"
 8) "0cd29857-3122-403b-90f2-86a0d6aaa71f"
 9) "groupIds.[3]"
10) "2f1b9636-f262-4f4b-ae8e-779b30716367"
11) "groupIds.[4]"
12) "af8226e0-61cc-4099-9e95-6daf0adb23a1"
13) "id"
14) "f2034bb3-405f-4417-9f2d-4bf451fb529f"
15) "roleIds.[0]"
16) "51de7a6f-c0b5-4e54-82e3-d8d9ad99aecd"
17) "roleIds.[1]"
18) "51a0fb53-17c4-4d46-ac2a-05fa917f2e84"
19) "roleIds.[2]"
20) "c7344a5f-a7db-4d68-b2d1-6d3b41e84090"
21) "roleIds.[3]"
22) "07c45632-b942-495f-a204-887948cf2968"
23) "roleIds.[4]"
24) "9f4a32d3-05d8-4dc7-8616-67c84ea8e09e"
```

## Thread 별 캐시를 구현하는 방법

API Request마다 캐시를 이용해야하는 로직을 구현해야해서 찾아보다가 `ThreadLocal`이라는 구현체가 있다는 것을 알게 되었다. JDK 1.2 부터 나온 (~~공부좀 하자~~) ThreadLocal은 Thead 단위로 생명주기를 가지는 값을 저장하고 다룰 수 있도록 도와준다. `get`, `set`, `remove` 세가지 함수를 제공하며 순서대로 값을 가져오고 저장하고 제거하는 기능을 수행한다.

Web Application에서 API Request는 하나의 Thread로 동작하므로 앞서 말한 Request마다 캐시를 이용해야하는 상황에 적합한 구현체라고 할 수 있다. 다만, Tomcat가 같이 WAS에서는 Thread를 효율적으로 관리하기 위해서 Thread Pool을 사용할 수 있다. 그래서 원치않게 Request가 종료되었음에도 데이터가 초기화 되지 않아 캐시가 이상하게 동작할 수 있으니 아래와 같이 Request가 종료되면 초기화해주는 로직을 반드시 추가하자.

아래 코드는 Spring MVC에서 제공해주는 Interceptor를 이용하여 캐시를 다루는 코드이다.

```kotlin
@Component
class ContextManageInterceptor : HandlerInterceptor {
    private val cache = ThreadLocal<LoginUser>()
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        cache.set("some cache")
        return true
    }

    override fun postHandle(
        request: HttpServletRequest,
        response: HttpServletResponse,
        handler: Any,
        modelAndView: ModelAndView?,
    ) {
        cache.remove()
    }
}
```

## Jira에서 Epic의 Story들에 Task를 쉽게 연결해주는 방법

Jira를 사용할 때 가장 불편한점이 바로 Story와 Task가 동등한 관계라는 것이다. 물론 Jira에서 추구하는 바는 이해한다. 하지만 우리가 주로 작업할 때 Story를 두고 해당 Story와 관련있는 Task들을 생성하는 경우가 많다. (솔직히 내가 지금 까지 개발하면서 대부분이 그랬다.) 하지만 Jira에서 Story와 Task는 동등한 관계이기 때문에 Story의 하위로 Task들을 생성할 수 없다. 물론, Sub-Task를 생성하여 작업을 할 순 있지만 Sub-Task는 Jira의 Report 생성 시 데이터로 잡히지 않는 경우가 많기에 불편한점이 있다.

우리팀에서는 Story와 Task들의 관계를 맺어줌으로써 특정 작업이 특정 Story를 만족시키기 위한 작업임을 표시하기 위해 노력하고 있다.

![jira-task-store-link](/assets/images/retrospective/jira-task-store-link.png)

하지만 매 작업을 생성할 때마다 이러한 링크를 연결해주는 작업은 상당히 번거롭다. 에픽에서 Task를 만들어주고 해당 Task가 어떤 스토리와 관련있는지 찾은다음 연결을 해주어야 하기 때문이다.

그래서 우리가 찾은 방법은 Story에서 관련있는 Task를 생성해준다면 Epic을 연결해주는 방법이었다.

![jira-create-task-in-story](/assets/images/retrospective/jira-create-task-in-story.png)

이렇게 생성하면 적어도 Task 생성 후에 Story ID를 찾아헤맬 필요는 없다. 다만 Task들 생성해주고 Epice들을 매번 연결해주는 것은 여전히 번거로운 작업이었다. 이를 해결해줄 좋은 방법이 있었는데 바로 Jira의 Automation이었다.

Task가 생성될 때 연결된 Story의 Epic을 복사해서 연결해주도록 하면 되는것이었다.

![auto-epic-link-automation](/assets/images/retrospective/auto-epic-link-automation.png)

Automation에 대해 조금더 설명하자면 아래와 같다.

- `When: Issue Linked`는 이슈에 링크에 연결되면 Automation이 트리거 된다.
- `For: Current Issue`는 현재 트리거된 이슈에 대한 작업을 수행한다는 의미이다.
- `Then: Edit issue fields`는 현재 트리거된 이슈에 대한 이슈 필드를 수정한다는 의미이다. 다만 Jira에 미리 정의된 템플릿이 없었기 때문에 `advanced field editing`을 이용하여 아래와 같이 Story의 에픽링크를 복사하도록 하였다.
  ```
  {
    "fields": {
        "Epic Link": "{% raw %}{{destinationIssue.Epic Link}}{% endraw %}"
    }
  }
  ```

이와 같이 설정하면 Story에서 Task를 생성하여 동일한 에픽 내 해당 스토리에 링크된 Task를 손쉽게 생성할 수 있게 된다.

## `EnableAsync`시 참고사항

Spring Boot에서 비동기 작업을 위해 [EnableAsync](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableAsync.html){: target="\_blank" }를 사용하면 기본적으로 [SimpleAsyncTaskExecutor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/SimpleAsyncTaskExecutor.html){: target="\_blank" }를 사용한다.

> By default, Spring will be searching for an associated thread pool definition: either a unique TaskExecutor bean in the context, or an Executor bean named "taskExecutor" otherwise. If neither of the two is resolvable, a SimpleAsyncTaskExecutor will be used to process async method invocations. Besides, annotated methods having a void return type cannot transmit any exception back to the caller. By default, such uncaught exceptions are only logged.

`SimpleAsyncTaskExecutor`문서에도 나와있다시피 해당 구현체는 Thread를 재사용하지 않기 대문에 성능적으로 좋지 않은 구현체이다.

> NOTE: This implementation does not reuse threads! Consider a thread-pooling TaskExecutor implementation instead, in particular for executing a large number of short-lived tasks.

그러므로 아래와 같이 [ThreadPoolTaskExecutor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html){: target="\_blank" }를 사용하자.

```kotlin
@Configuration
@EnableAsync
class AsyncConfig : AsyncConfigurer {
    override fun getAsyncExecutor(): Executor {
        return ThreadPoolTaskExecutor().apply {
            corePoolSize = 2
            maxPoolSize = 5
            queueCapacity = 10
            initialize()
        }
    }
}
```

## `Transactional`의 `noRollbackFor`활용

Spring Boot에서 도메인 서비스 구현 시 원자성을 보장하기 위해서 `@Transactional`을 선언하여 함수를 동작시킨다. 해당 어노테이션이 선언된 함수에서는 Runtime Exception이 발생한다면 모든 영속화가 Rollback된다.

심지어 try-catch로 처리하였다고 하더라도 영속화는 Rollback된다. [우아한 기술블로그 참고](https://techblog.woowahan.com/2606/){: target="\_blank" }

다만, 특정한 예외에 대해서는 Rollaback을 하고싶지 않은 경우도 생길 수 있다. 그럴경우 아래와 같이 `noRollbackFor`를 선언해주면 Rollback이 되지 않도록 할 수 있다.

아래 예제는 비밀번호 실패시 예외를 던지지만 실패 카운트를 증가시키고 영속화시키기 위한 로직을 구현한 것이다.

```kotlin
@Transactional(noRollbackFor = [AuthenticationException::class])
fun login(command: LoginCommand): LoginResult {
    // ... 생략

    if (!passwordEncoder.matches(command.password, user.password)) {
        user.addLoginFailHistory(command.username)
        throw PasswordInvalidException(user.loginFailHistories.size, LOGIN_ATTEMPT_LIMIT)
    }

    // ... 생략
}
```

만약 `noRollbackFor`를 선언하지 않았다면 `throw PasswordInvalidException(user.loginFailHistories.size, LOGIN_ATTEMPT_LIMIT)`로 인해서 `user.addLoginFailHistory(command.username)`코드가 의미없어질 것이다.

## kotlin version 1.8.22 업그레이드 이슈

Kotlin을 1.8.22로 업그레이드하니까 QueryDSL의 코드젠이 제대로 되지 않는 이슈가 발생했다.

```
org.jetbrains.kotlin.backend.common.BackendException: Backend Internal error: Exception during psi2ir
File being compiled: /Users/nicklas/Repositories/personal/kotshi/tests/src/main/kotlin/se/ansman/kotshi/TestFactory.kt
The root cause java.lang.RuntimeException was thrown at: org.jetbrains.kotlin.psi2ir.generators.GeneratorKt.getTypeInferredByFrontendOrFail(Generator.kt:52)
	at org.jetbrains.kotlin.backend.common.CodegenUtil.reportBackendException(CodegenUtil.kt:253)
	at org.jetbrains.kotlin.backend.common.CodegenUtil.reportBackendException$default(CodegenUtil.kt:237)
	at org.jetbrains.kotlin.psi2ir.generators.DeclarationGenerator.generateMemberDeclaration(DeclarationGenerator.kt:78)
	at org.jetbrains.kotlin.psi2ir.generators.ModuleGenerator.generateSingleFile(ModuleGenerator.kt:73)
	at org.jetbrains.kotlin.psi2ir.generators.ModuleGenerator.generateModuleFragment(ModuleGenerator.kt:53)
	at org.jetbrains.kotlin.psi2ir.Psi2IrTranslator.generateModuleFragment(Psi2IrTranslator.kt:97)
	at org.jetbrains.kotlin.backend.jvm.JvmIrCodegenFactory.convertToIr(JvmIrCodegenFactory.kt:224)
	at org.jetbrains.kotlin.backend.jvm.JvmIrCodegenFactory.convertToIr(JvmIrCodegenFactory.kt:57)
	at org.jetbrains.kotlin.codegen.KotlinCodegenFacade.compileCorrectFiles(KotlinCodegenFacade.java:41)
	at org.jetbrains.kotlin.kapt3.AbstractKapt3Extension.contextForStubGeneration(Kapt3Extension.kt:280)
	at org.jetbrains.kotlin.kapt3.AbstractKapt3Extension.analysisCompleted(Kapt3Extension.kt:174)
	at org.jetbrains.kotlin.kapt3.ClasspathBasedKapt3Extension.analysisCompleted(Kapt3Extension.kt:10
// ... 생략
```

QueryDSL 때문은 아닌것 같고 [비슷한 이슈](https://youtrack.jetbrains.com/issue/KT-57946){: target="\_blank" } 가 리포팅된게 있길래 보고 아래와 같이 수정하니 해결되었다.

```properties
// gradle.properties

kapt.use.jvm.ir=false
```
