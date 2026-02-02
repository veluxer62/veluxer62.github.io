---
layout: single
title: "코딩 컨벤션을 테스트 하자(feat. Konsist)"
categories:
  - TUTORIALS
tags:
  - konsist
  - kotlin
  - junit
  - kotest
toc: true
toc_sticky: true
excerpt: "개발자로서 팀 단위로 서비스 개발을 위한 코드를 작성할 때 유지보수성을 높이기 위해 일관된 코드 작성, 의존성 방향 제약 등을 위한 팀 내 코드 컨벤션을 작성하거나 본 경험이 있을 것이다. 보통 README.md나 사내 문서에 코딩 컨벤션을 작성해서 개발자들이 숙지하도록 하거나 Lint나 Sonarqube와 같은 정적분석 도구를 이용하여 일관된 코드 스타일을 지킬 수 있도록 조치한다. 다만, 위와 같은 방식은 아무래도 아래와 같은 한계가 존재할 수 밖에 없다."
---

## Intro

개발자로서 팀 단위로 서비스 개발을 위한 코드를 작성할 때 유지보수성을 높이기 위해 일관된 코드 작성, 의존성 방향 제약 등을 위한 팀 내 코드 컨벤션을 작성하거나 본 경험이 있을 것이다.

보통 README.md나 사내 문서에 코딩 컨벤션을 작성해서 개발자들이 숙지하도록 하거나 [Lint](https://en.wikipedia.org/wiki/Lint_(software)){: target="\_blank" }나 [Sonarqube](https://www.sonarsource.com/products/sonarqube/){: target="\_blank" }와 같은 정적분석 도구를 이용하여 일관된 코드 스타일을 지킬 수 있도록 조치한다.

다만, 위와 같은 방식은 아무래도 아래와 같은 한계가 존재할 수 밖에 없다.

## 기존 코딩 컨벤션의 한계

### 문서를 이용한 코딩 컨벤션

문서를 이용한 코딩 컨벤션은 처음에는 작성하기 쉽다. 코드 관리에 책임을 지고 있는 개발자나 의지가 있는 특정 개발자가 문서를 작성하고 팀원들에게 배포하면 되기 때문이다. 하지만 시간이 지날수록 개선되거나 변경되는 코딩 컨벤션이 업데이트되지 않는 경우를 많이 볼 수 있다. 어떻게 보면 코드 주석에서 나타나는 문제와 동일한 문제이다. 누군가가 신경 써서 챙긴다고 해도 결국 코드와 문서는 따로 움직이기 때문에 결국 코딩 컨벤션 문서는 유지보수에 대한 이슈를 항상 가지고 있게 된다.

### Lint

개인적으로 가장 좋아하는 도구이다. 아주 간단한 설정만으로 여러 팀원이 유사한 코드 스타일을 가지도록 만들어주기 때문이다. 하지만 lint는 개행, 최대 글자 수, EOF등 정말 코드 스타일만 잡아준다. 그렇기 때문에 의도를 나타내는 클래스 명이나 변수명, 유지보수성을 위한 코드간 의존성 방향 정책 등을 잡아주긴 힘들다.

### Sonarqube

Sonarqube는 유명한 정적 분석 도구로 Lint를 포함해서 중복코드, 효율성, 복잡도, 테스트 커버리지, 보안 이슈 등 코드 퀄리티를 위한 것보다 다양한 분석 데이터를 제공해 준다. 코드 퀄리티를 위한 일반적인 수준의 가이드를 제공해 주기 때문에 나쁘지 않지만, 우리만의 컨벤션이 있을 때 적용하기에는 쉽지 않다. (물론 Plugin을 통해 적용해 볼 수 있지만 상당히 복잡하다)

<br/>

이와 같은 이유로 Konsist를 처음 알게 되었을 때 무척이나 기뻤다. 문서로 관리하는 컨벤션은 사람이 하는 일이라 누락되는 경우가 비일비재하고 Sonarqube와 같이 복잡한 설정은 누구나 손쉽게 컨벤션 설정을 하기에 부담감이 크다는 단점이 있었기 때문이었다.

Konsist는 단순히 테스트 코드에 우리가 원하는 컨벤션을 정의해두고 누군가 해당 컨벤션을 어긴 경우 테스트 코드에서 바로 잡아주기 때문에 빠르게 조치가 가능하다. 그리고 아래에 소개하겠지만 손쉬운 설정은 누구든 손쉽게 컨벤션을 추가하고 보정해 나갈 수 있다는 장점이 있다.

그럼, 이제 본격적으로 Konsist를 알아보자.

## Konsist의 특징

[Konsist](https://docs.konsist.lemonappdev.com/){: target="\_blank" }는 Kotlin을 위한 코드 정적분석기로 API를 이용한 단위 테스트를 통해 팀에서 정의한 여러 가지 코딩 규칙들을 검증할 수 있는 특징이 있다. 즉, 개발자는 코드 통합 전, 테스트 프레임워크를 통해 팀에서 정의한 코딩 규칙에 맞게 코드를 작성했는지 검증할 수 있고, 이는 통합된 우리의 코드가 항상 우리의 코딩 컨벤션에 맞게 작성되어 있음을 보장할 수 있다는 장점을 가진다. 개인적으로 Konsist를 적용해 보았을 때 가장 크게 느꼈던 장점은 바로 아래와 같은 “단순함”이었다.

### 간단한 적용

Konsist는 Lint와 같이 단순히 Gradle이나 Maven에서 의존성을 추가하는 것만으로도 사용이 가능하다. 그리고 JUnit이나 Kotest와 같은 테스트 프레임워크에 손쉽게 통합해서 사용할 수 있기 때문에 적용이 아주 쉽다는 큰 장점을 가진다.

### 단순한 구조

아래 Konsist의 구조를 보면 아주 단순한 구조를 가지는 것을 볼 수 있다. [architecture check](https://docs.konsist.lemonappdev.com/writing-tests/architecture-assert){: target="\_blank" }을 보면 4개의 step으로 나뉘어져 있지만 크게 보면 단순히 2개의 step으로 되어 있다고 보면 된다.

![konstis-architecture](/assets/images/posts/getting-started-with-konsist/konstis-architecture.png)

이렇게 단순한 구조는 개발자들이 Konsist를 사용할 때 복잡한 구조에 따른 학습 및 사용 패턴 구현에 집중하기보다 검증해야 할 코딩 컨벤션을 표현하는 데 집중할 수 있다는 장점이 있다.

### 손쉬운 통합

Konsist는 [JUnit](https://junit.org/junit5/){: target="\_blank" }, [Kotest](https://kotest.io/){: target="\_blank" }와 같은 테스트 프레임워크에서 손쉽게 통합해서 사용할 수 있다. 즉, 처음부터 Konsist를 적용하지 않았어도 언제든지 손쉽게 기존 프로젝트에 Konsist를 적용할 수 있다.

## Konsist 시작하기

Konsist의 장점을 알아보았다면 이제 본격적으로 프로젝트에 Konsist를 적용해 보자.

### 의존성 추가하기

Konsist는 아래와 같이 간단히 Gradle 파일에 의존성을 추가해 주는 것만으로도 사용할 수 있다. (해당 글에서는 Gradle - Kotlin과 JUnit5를 기준으로 설명하고 있으니, Maven이나 Kotest를 사용하고 싶다면 [공식 홈페이지](https://docs.konsist.lemonappdev.com/getting-started/getting-started/kotest-support){: target="\_blank" }를 참고하자.)

```kotlin
// build.gradle.kts

repositories {
    mavenCentral()
}

dependencies {
    // 생략...
    testImplementation("com.lemonappdev:konsist:0.15.1")
}
```

### 단계

Konsist는 크게 범위 설정(Create the scope), 쿼리 및 필터(Query and filter the declarations), 검증(Assert) 3개의 단계를 가진다.

![konsist-steps](/assets/images/posts/getting-started-with-konsist/konsist-steps.png)

공식 문서에서 보면 Declaration Check은 3단계, Architectural Check은 4단계로 나누어진다고 이야기하고 있지만 크게 보면 Architectural Check은 쿼리 및 필터 단계가 생략되었다고 봐도 무방할 듯하다.

#### 1. Create The Scope

![konstis-architecture](/assets/images/posts/getting-started-with-konsist/konstis-architecture.png)

Konsist의 첫 번째 단계로 검증할 파일들의 범위를 나타낸다. 개인적으로 자주 사용할 만한 범위를 나타내는 함수들은 아래와 같다. 더 자세한 범위 지정 함수를 알아보려면 공식 문서를 참고하자.

- 프로젝트 전체 파일들
  ```kotlin
  Konsist.scopeFromProject()
  ```

- 모듈 단위 파일들
  ```kotlin
  Konsist.scopeFromModule("app")
  ```

- 운영 코드 파일
  ```kotlin
  Konsist.scopeFromProduction()
  ```

- 테스트 코드 파일
  ```kotlin
  Konsist.scopeFromTest()
  ```

- 패키지 단위 파일들
  ```kotlin
  Konsist.sourceFromPackage("com.usecase..")
  ```

#### 2. Declaration Filtering

![filter-step](/assets/images/posts/getting-started-with-konsist/filter-step.png)

범위를 지정하였다면 다음 단계는 필터링 단계이다. Declaration Check에서 사용하며 Architectural Check에서는 사용하지 않는다. 1단계에서 조회한 범위에서 파일들, 프로퍼티들, 함수들과 같이 원하는 항목들만 필터링하는 역할을 수행한다.

```kotlin
koScope
    .classes()
    .properties()
```

기본적으로 제공하는 함수 외에도 직접 필터를 정의해서 사용할 수 있다.

```kotlin
koScope
    .classes()
    .filter { it.hasAnnotationOf<UseCase>() }
```

#### 3. Assertion

앞서 정의한 코드들을 검증하는 단계이다. Declaration Check과 Architectural Check은 검증하는 방법이 나뉜다.

##### Declaration Assertion

![assert-declaration](/assets/images/posts/getting-started-with-konsist/assert-declaration.png)

코드 베이스를 검증하는 단계로 `assertTrue`, `assertFalse`, `assertEmpty`, `assertNotEmpty` 등의 함수를 제공해 준다.

```kotlin
koScope
    .interfaces()
    .assertTrue { it.hasPublicModifier() }
```

##### Architecture Assertion

![assert-architecture](/assets/images/posts/getting-started-with-konsist/assert-architecture.png)

아키텍처를 검증하는 단계로 `assertArchitecture` 함수를 이용하여 검증을 수행한다.

```kotlin
koScope
    .assertArchitecture {
        val presentation = Layer("Presentation", "com.myapp.presentation..")
        val data = Layer("Data", "com.myapp.data..")
    
        presentation.dependsOn(data)
        data.dependsOnNothing()
    }
```

## 활용 사례

시작하기를 보면 너무 단순해서 감이 안 올 수도 있다고 생각한다. 그래서 내가 생각하기에 활용해 봄 직한 여러 사례를 소개하면서 좀 더 이해를 돕고자 한다.

### 클래스명 규칙 테스트

REST API의 요청을 라우팅하기 위한 Controller 들은 `~Controller`라는 클래스 명을 가져야 한다는 컨벤션을 사용한다고 가정해 보자. Declaration Check은 우리가 위에서 정의한 규칙에 맞게 잘 선언되었는지 검증할 수 있다.

```kotlin
@RestController
class HomeController {
    @GetMapping("/")
    fun home() {}
}

@Test
fun `Router 클래스명은 'Controller' 접미사를 가진다`() {
    Konsist.scopeFromProject()
        .classes()
        .withAllAnnotationsOf(RestController::class)
        .assertTrue { it.name.endsWith("Controller") }
}
```

만약 규칙에 어긋나는 클래스 명을 정의한 경우 아래와 같이 테스트가 실패하게 된다. 어느 클래스가 규칙을 위반하였는지 정확히 알려주기 때문에 손쉽게 위반 사항을 수정할 수 있다.

```kotlin
@RestController
class FooRouter {
    @GetMapping("/foo")
    fun foo() {}
}

@Test
fun `Router 클래스명은 'Controller' 접미사를 가진다`() {
    Konsist.scopeFromProject()
        .classes()
        .withAllAnnotationsOf(RestController::class)
        .assertTrue { it.name.endsWith("Controller") }
}
```

```
Assert 'Router 클래스명은 'Controller' 접미사를 가진다' was violated (1 time). Invalid declarations:
/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/FooRouter.kt:6:1 (FooRouter ClassDeclaration)
com.lemonappdev.konsist.core.exception.KoAssertionFailedException: Assert 'Router 클래스명은 'Controller' 접미사를 가진다' was violated (1 time). Invalid declarations:
/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/FooRouter.kt:6:1 (FooRouter ClassDeclaration)
	at com.lemonappdev.konsist.core.verify.KoDeclarationAndProviderAssertCoreKt.getResult(KoDeclarationAndProviderAssertCore.kt:259)
	at com.lemonappdev.konsist.core.verify.KoDeclarationAndProviderAssertCoreKt.assert(KoDeclarationAndProviderAssertCore.kt:54)
	at com.lemonappdev.konsist.api.verify.KoDeclarationAndProviderAssertKt.assertTrue(KoDeclarationAndProviderAssert.kt:78)
	at com.lemonappdev.konsist.api.verify.KoDeclarationAndProviderAssertKt.assertTrue$default(KoDeclarationAndProviderAssert.kt:72)
	at com.example.konsistdemo.DeclarationTest.Router 클래스명은 'Controller' 접미사를 가진다(DeclarationTest.kt:15)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
```

### 예외처리

만약 위 예제에서 `FooRouter`를 부득이하게 사용해야 한다면 어떻게 할 수 있을까? Konsist에서는 이와 같은 사례에 대응하기 위해 [예외 처리 기능](https://docs.konsist.lemonappdev.com/writing-tests/suppressing-konsist-test){: target="\_blank" }을 지원한다.

아래와 같이 `@Supress` 를 `FooRouter`에 추가해 주고 테스트 명과 동일한 이름을 넣어주면 예외 처리가 되는 것을 볼 수 있다. (`konsist.`접두사를 붙이지 않아도 정상동작 하지만 공식 문서에서는 해당 접두사를 기재해 주는 것을 권장하고 있다.)

```kotlin
@RestController
@Suppress("konsist.Router 클래스명은 'Controller' 접미사를 가진다")
class FooRouter {
    @GetMapping("/foo")
    fun foo() {}
}

@Test
fun `Router 클래스명은 'Controller' 접미사를 가진다`() {
    Konsist.scopeFromProject()
        .classes()
        .withAllAnnotationsOf(RestController::class)
        .assertTrue { it.name.endsWith("Controller") }
}
```

![declaration-test-result](/assets/images/posts/getting-started-with-konsist/declaration-test-result.png)

### 3 계층 구조 테스트

이전부터 많은 개발자들이 고민하였겠지만, 클린아키텍쳐 책이 나온 이후부터 개발자들의 코드 아키텍처에 대한 고민은 더욱더 활발해졌다고 생각한다. 클린아키텍쳐에서는 특히 의존성에 대해 강조하고 있는데 Architectural Check은 우리가 자칫 놓치기 쉬운 의존성 규칙에 대한 검증을 수행해 준다.

아래는 간단한 [Three-tier architecture](https://en.wikipedia.org/wiki/Multitier_architecture#Three-tier_architecture){: target="\_blank" }에 대한 규칙을 검증한 코드이다.

![architecture-test-example](/assets/images/posts/getting-started-with-konsist/architecture-test-example.png)

```kotlin
// com.example.konsistdemo.data

interface BoardRepository : JpaRepository<Board, UUID>

@Entity
class Board(
    title: String,
    content: String,
) {
    @Id
    val id: UUID = UUID.randomUUID()

    @Column(nullable = false)
    val title: String = title

    @Column(nullable = false)
    val content: String = content
}
```

```kotlin
// com.example.konsistdemo.application

@Service
@Transactional(readOnly = true)
class BoardService(
    private val boardRepository: BoardRepository,
) {
    fun findAll(): List<Board> = boardRepository.findAll()
}
```

```kotlin
// com.example.konsistdemo.presentation

@RestController
class BoardController(
    private val boardService: BoardService,
) {
    @GetMapping("/boards")
    fun findAll(): List<Board> = boardService.findAll()
}
```

```kotlin
@Test
fun `presentation 레이어는 application 레이어를, application 레이어는 data 레이어를 의존한다`() {
    Konsist.scopeFromProject()
        .assertArchitecture {
            val presentation = Layer("Presentation", "com.example.konsistdemo.presentation..")
            val application = Layer("Application", "com.example.konsistdemo.application..")
            val data = Layer("Data", "com.example.konsistdemo.data..")

            presentation.dependsOn(application, data)
            application.dependsOn(data)
            data.dependsOnNothing()
        }
}
```

만약 의존성 규칙을 위반한 경우에도 역시나 아래와 같이 테스트가 실패한다. 어느 계층에서 잘못된 의존성을 가졌는지 알려주므로 손쉽게 찾아 보정할 수 있다.

```kotlin
// com.example.konsistdemo.data

interface BoardRepository : JpaRepository<Board, UUID>

@Entity
class Board(
    title: String,
    content: String,
) {
    @Id
    val id: UUID = UUID.randomUUID()

    @Column(nullable = false)
    val title: String = title

    @Column(nullable = false)
    val content: String = content
}
```

```kotlin
// com.example.konsistdemo.application

@Service
@Transactional(readOnly = true)
class BoardService(
    private val boardRepository: BoardRepository,
) {
    fun findAll(): List<Board> = boardRepository.findAll()

    @Transactional
    fun create(command: BoardCreationCommand) {
        val entity = command.toEntity()
        boardRepository.save(entity)
    }
}
```

```kotlin
// com.example.konsistdemo.presentation

@RestController
class BoardController(
    private val boardService: BoardService,
) {
    @GetMapping("/boards")
    fun findAll(): List<Board> = boardService.findAll()

    @PostMapping("/boards")
    fun create(
        @RequestBody command: BoardCreationCommand,
    ) {
        boardService.create(command)
    }
}

data class BoardCreationCommand(
    val title: String,
    val content: String,
) {
    fun toEntity() = Board(title, content)
}
```

```kotlin
@Test
fun `presentation 레이어는 application 레이어를, application 레이어는 data 레이어를 의존한다`() {
    Konsist.scopeFromProject()
        .assertArchitecture {
            val presentation = Layer("Presentation", "com.example.konsistdemo.presentation..")
            val application = Layer("Application", "com.example.konsistdemo.application..")
            val data = Layer("Data", "com.example.konsistdemo.data..")

            presentation.dependsOn(application, data)
            application.dependsOn(data)
            data.dependsOnNothing()
        }
}
```

```
'presentation 레이어는 application 레이어를, application 레이어는 data 레이어를 의존한다' test has failed.
Application depends on Data assertion failure:
A file /Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/application/BoardService.kt in a Application layer depends on Presentation layer, imports:
	com.example.konsistdemo.presentation.BoardCreationCommand (/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/application/BoardService.kt:5:1)
com.lemonappdev.konsist.core.exception.KoAssertionFailedException: 'presentation 레이어는 application 레이어를, application 레이어는 data 레이어를 의존한다' test has failed.
Application depends on Data assertion failure:
A file /Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/application/BoardService.kt in a Application layer depends on Presentation layer, imports:
	com.example.konsistdemo.presentation.BoardCreationCommand (/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/application/BoardService.kt:5:1)
	at com.lemonappdev.konsist.core.verify.KoArchitectureAssertKt.assert(KoArchitectureAssert.kt:65)
	at com.lemonappdev.konsist.core.architecture.KoArchitectureAssertionCore.assertArchitecture(KoArchitectureAssertionCore.kt:19)
	at com.lemonappdev.konsist.core.architecture.KoArchitectureAssertionCore.assertArchitecture(KoArchitectureAssertionCore.kt:10)
	at com.lemonappdev.konsist.api.architecture.KoArchitectureCreator.assertArchitecture(KoArchitectureCreator.kt)
	at com.example.konsistdemo.ArchitecturalTest.presentation 레이어는 application 레이어를, application 레이어는 data 레이어를 의존한다(ArchitecturalTest.kt:12)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
```

### 파라미터 개수 규칙 테스트

논란이 있는 주제이지만 Konsist의 기능을 소개하기 위해 우선 팀 내에서 가독성(?)을 위해 함수의 파라미터가 3개 이상이면 DTO을 만들어서 사용하자는 규칙을 정했다고 가정해 보겠다. 이와 같은 규칙이 있는 경우 아래와 같이 테스트할 수 있다.

```kotlin
@Test
fun `함수 파라미터는 3개 이상 선언할 수 없다`() {
    Konsist.scopeFromProject()
        .functions()
        .assertTrue {
            it.parameters.size <= 3
        }
}
```

만약 아래와 같이 3개 이상의 매개변수를 가진 함수를 생성하면 오류가 발생함을 볼 수 있다.

```kotlin
@Service
@Transactional(readOnly = true)
class BoardService(
    private val boardRepository: BoardRepository,
) {
    fun findAll(): List<Board> = boardRepository.findAll()

    @Transactional
    fun create(command: BoardCreationCommand) {
        val entity = command.toEntity()
        boardRepository.save(entity)
    }

    fun foo(
        p1: String,
        p2: String,
        p3: String,
        p4: String,
    ) {}
}
```

```
Assert '함수 파라미터는 3개이상 선언할 수 없다' was violated (1 time). Invalid declarations:
/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/application/BoardService.kt:22:5 (foo FunctionDeclaration)
com.lemonappdev.konsist.core.exception.KoAssertionFailedException: Assert '함수 파라미터는 3개이상 선언할 수 없다' was violated (1 time). Invalid declarations:
/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/application/BoardService.kt:22:5 (foo FunctionDeclaration)
	at com.lemonappdev.konsist.core.verify.KoDeclarationAndProviderAssertCoreKt.getResult(KoDeclarationAndProviderAssertCore.kt:259)
	at com.lemonappdev.konsist.core.verify.KoDeclarationAndProviderAssertCoreKt.assert(KoDeclarationAndProviderAssertCore.kt:54)
	at com.lemonappdev.konsist.api.verify.KoDeclarationAndProviderAssertKt.assertTrue(KoDeclarationAndProviderAssert.kt:78)
	at com.lemonappdev.konsist.api.verify.KoDeclarationAndProviderAssertKt.assertTrue$default(KoDeclarationAndProviderAssert.kt:72)
	at com.example.konsistdemo.DeclarationTest.함수 파라미터는 3개이상 선언할 수 없다(DeclarationTest.kt:22)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
```

### Entity 규칙 테스트

Entity를 정의하는 방법은 회사마다 팀마다 다양할 것이다. 모든 사례를 다 소개할 순 없으니 여기서는 내가 생각하기에 누락하면 치명적일 수 있는 실수를 방지하기 위한 한 가지 규칙을 소개해 보고자 한다.

Kotlin에서는 Java와 다르게 언어 레벨에서 null-safety를 지원한다. 하지만 Java 언어 기반으로 되어있는 JPA의 경우 데이터베이스 데이터를 매핑할 때 nullable 타입과 JPA 어노테이션 선언을 정확하게 일치시켜 선언해 주지 않으면 런타임에서 예상치 못한 오류를 발견할 수 있다. 이럴 때 아래와 같이 Class의 Property와 @Column의 nullable 파라미터값이 반드시 일치하도록 정의하는 규칙을 사용할 수 있다.

```kotlin
@Test
fun `Entity 클래스의 Column 프로퍼티가 non-nullable 타입이라면 '@Column(nullable=false)'가 선언되어야 한다`() {
    Konsist.scopeFromProject()
        .classes()
        .withAllAnnotationsOf(Entity::class)
        .properties()
        .withAllAnnotationsOf(Column::class)
        .filter { it.type?.isNullable == false }
        .assertTrue {
            it.hasAnnotation { annotation ->
                annotation.hasArgument { arg ->
                    arg.name == "nullable" && arg.value == "false"
                }
            }
        }
}
```

아래와 같이 규칙에 어긋나는 컬럼을 정의한 경우 오류가 발생함을 볼 수 있다.

```kotlin
@Entity
class Board(
    title: String,
    content: String,
) {
    @Id
    val id: UUID = UUID.randomUUID()

    @Column(nullable = false)
    val title: String = title

    @Column(nullable = false)
    val content: String = content

    @Column
    val foo: String = ""
}
```

```
Assert 'Entity 클래스의 프로퍼티가 non-nullable 타입이라면 '@Column(nullable=false)'가 선언되어야 한다' was violated (1 time). Invalid declarations:
/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/data/Board.kt:22:5 (foo PropertyDeclaration)
com.lemonappdev.konsist.core.exception.KoAssertionFailedException: Assert 'Entity 클래스의 프로퍼티가 non-nullable 타입이라면 '@Column(nullable=false)'가 선언되어야 한다' was violated (1 time). Invalid declarations:
/Users/kei/workspace/personal/KonsistDemo/src/main/kotlin/com/example/konsistdemo/data/Board.kt:22:5 (foo PropertyDeclaration)
	at com.lemonappdev.konsist.core.verify.KoDeclarationAndProviderAssertCoreKt.getResult(KoDeclarationAndProviderAssertCore.kt:259)
	at com.lemonappdev.konsist.core.verify.KoDeclarationAndProviderAssertCoreKt.assert(KoDeclarationAndProviderAssertCore.kt:54)
	at com.lemonappdev.konsist.api.verify.KoDeclarationAndProviderAssertKt.assertTrue(KoDeclarationAndProviderAssert.kt:78)
	at com.lemonappdev.konsist.api.verify.KoDeclarationAndProviderAssertKt.assertTrue$default(KoDeclarationAndProviderAssert.kt:72)
	at com.example.konsistdemo.DeclarationTest.Entity 클래스의 프로퍼티가 non-nullable 타입이라면 '@Column(nullable=false)'가 선언되어야 한다(DeclarationTest.kt:38)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
```

## 마무리

Konsist는 코드 컨벤션을 더욱 간편하게 관리하고 팀 내 일관성을 유지하는 강력한 도구라 생각한다. 이를 통해 우리는 보다 효율적이고 안정적인 코드 베이스를 구축할 수 있을 것이다. Konsist의 단순한 설정과 강력한 검증 기능을 활용하여 여러분의 프로젝트에 적용해 보면 좋을 것 같다. 아마도 기존에 코딩 컨벤션을 어기지 않았는지 확인하는 등의 비용을 상당히 줄일 수 있어 코드 리뷰 시 비즈니스 요구사항에 대한 검증과 버그 가능성, 아키텍처 등에 대한 리뷰에 더욱더 집중하고 불필요한 리뷰 시간을 획기적으로 줄일 수 있을 것이다.

이 글에서 작성한 코드는 [KonsistDemo 저장소](https://github.com/veluxer62/KonsistDemo){: target="\_blank" }에서 확인할 수 있으니 참고하기를 바란다.
