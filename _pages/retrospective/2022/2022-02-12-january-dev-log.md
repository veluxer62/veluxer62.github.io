---
layout: single
title: "2022년 1월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2022-january-dev-log/
---

2022년의 개발일지를 적어본다. 2022년을 축하한지가 어제같은데 벌써 한달이라는 시간이 지났다. 나이가 들수록(?) 시간이 빨리간다는 말이 진짜 실감이 될 정도로 시간이 정말 잘가는 것 같다.

현재 진행중인 언어 전환 작업은 어느정도 예상은 했었으나 역시나 계획했던 일정에 비해 지연이 되고 있다. 1월안에 1/3을 마무리 지었어야 하는데 마무리 짓지 못하였고 2월 초순에 마무리가 될 듯하다. 지연된 이유는 여러가지가 있는데, 전환 시작 시점이 마지막 스프린트의 이슈로 인해 지연되었고 옆팀의 인수, 그리고 백엔드 팀원의 퇴사 등의 이슈 등이 있다. 앞으로 남은 작업이 좀 더 복잡한 것들이라 작업 일정 예측이 더욱 쉽지 않은지라 일정 관리가 좀더 쉽지 않을 것 같아 불안함이 있다. 일정이 너무 지연이 되는것도 작업자들의 번아웃이 올까 염려가 되는 부분도 있다. 잘 마무리 할 수 있는 방법을 좀 더 모색해서 모쪼록 무탈하게 마무리가 되었으면 한다.

[야곰 닷넷](https://yagom.net/tech-casts/){: target="\_blank" }으로 부터 테크 캐스트 연사 요청을 받았다. 제목은 [소통하는 개발자로 협UP하기](https://yagom.net/courses/techcast-9/){: target="\_blank" }인데 내가 작성한 블로그 중 [개발자의 소통의 중요성](https://veluxer62.github.io/discussions/importance-of-communication-for-developer/){: target="\_blank" } 글을 보고 연락을 주셨다고 한다. 개인적으로 이런 발표는 좋은 경험이 될 것 같아서 참가하기로 마음먹었는데 잘 되었으면 좋겠다.

아래는 1월동안 정리한 이슈 내용들이다.

## JPA `deleteById`

jpa에서 `deleteById`를 쓰면 아래와 같이 내부적으로 find로직이 존재한다.

```java
@Transactional
@Override
public void deleteById(ID id) {

  Assert.notNull(id, ID_MUST_NOT_BE_NULL);

  delete(findById(id).orElseThrow(() -> new EmptyResultDataAccessException(
      String.format("No %s entity with id %s exists!", entityInformation.getJavaType(), id), 1)));
}
```

findById를 명시적으로 사용한다면 아래와 같이 delete 함수를 쓰는게 성능적으로 더 이득일듯 하다.

```kotlin
fun deleteById(id: UUID) {
  val entity = repository.findById(id).orElseThrow()
  repository.delete(entity)
}
```

다만 JPA에서 Transaction내에서 캐싱을 하기 때문에 아래와 같이 수행해도 성능상 큰 차이는 없을 것이다.

```kotlin
fun deleteById(id: UUID) {
  repository.findById(id).ifPresent {
    repository.deleteById(id)
  }
}
```

## 접근제어

특정 기능을 구현하면서 접근제어 부분에 놀란 부분이 있어서 적어본다. (나만 신기한 거일 수도 있다...)
먼저 코드를 보자.

```kotlin
class Foo(
    private var a: String,
    protected var b: Int,
) {
    fun merge(foo: Foo) {
        this.a += foo.a
        this.b += foo.b
    }

    override fun toString(): String {
        return "Foo(a=$a, b=$b)"
    }
}

class FooTest {
    @Test
    fun test() {
        val f1 = Foo("hello, ", 1)
        val f2 = Foo("world", 2)

        // f1.a can't access
        // f1.b can't access
        // f2.a can't access
        // f2.b can't access

        f1.merge(f2)

        println(f1)
        // Foo(a=hello, world, b=3)
    }
}
```

`merge`함수를 보면 알겠지만 분명 매개변수인 `foo`의 변수 `a`와 `b`는 공개 필드가 아니기때문에 접근할 수 없어야 함에도 접근이 가능한 것을 볼 수 있다. 이는 해당 함수가 같은 클래스안에서 동작하는 것이기 때문일텐데 이런 것에 놀라는것을 보니 공부를 다시해야하지 않나 싶다.

## runatlantis

[runatlantis](https://www.runatlantis.io/){: target="\_blank" }는 Terraform 적용 이력을 Github의 Pull Request에 자동으로 기록해주는 도구이다. 현재 우리회사에서는 [Terraform Cloud](https://cloud.hashicorp.com/products/terraform){: target="\_blank" }를 사용하고 있기 때문에 필요성은 없지만 Terraform Cloud를 사용하지 않는다면 나쁘지 않은 도구처럼 보인다. 가격 정책 문서가 보이지 않는걸로 보니 무료인듯 하다.

## 주접 생성기

[주접 생성기](https://ju-jeob.com/){: target="\_blank" }라는 사이트가 있다. 참 재밌는 주제로 사이트까지 만들어서 운영하시는 부분에서 존경스럽다. 개발자분의 [블로그](https://jong-hui.github.io/){: target="\_blank" } 글도 몇몇 보았는데 개발에 대한 철학도 있으신것 처럼 보여서 언제 같이 일할 수 있는 기회가 생기면 좋겠다.

## `Clock.systemDefaultZone().millis()`

현재 시간을 long 타입으로 가져오는 방법중에 흔하게 사용했던 방법이 바로 `System.currentTimeMillis()`이다. 하지만 `System.currentTimeMillis()`을 쓸때 단점이 Mocking을 이용한 시간을 고정 시키는 방법이 없다는 것인데(내가 못찾았을 수도 잇다.) 그에 대한 대안으로 찾은 방법이 `Clock.systemDefaultZone().millis()`이다. 일단 사용했을 때 성능상 차이는 크게 모르겠다. 시간을 고정시키는 방법은 아래와 같이 할 수 있다.

```kotlin
mockkStatic(Clock::class) {
    every { Clock.systemDefaultZone().millis() } returns 36000000
    // Some Code...
}
```

## `mockkStatic`

[Kotest](https://kotest.io/){: target="\_blank" }를 이용한 테스팅 시 static 함수를 mocking 하는 용도로 아래와 같이 `mockkStatic`을 주로 활용한다.

```kotlin
fun test() {
    mockkStatic(OffsetDateTime::class)
    val now = OffsetDateTime.now()
    every { OffsetDateTime.now() } returns now

    // Some Test Code...


    unmockkStatic(OffsetDateTime::class)
}
```

만약 `unmockkStatic`을 사용하지 않으려면 아래와 같이 block을 잡아주면 된다.

```kotlin
fun test() {
    mockkStatic(OffsetDateTime::class) {
        val now = OffsetDateTime.now()
        every { OffsetDateTime.now() } returns now

        // Some Test Code...
    }
}
```

## JWT refresh token 이슈

사내에서 사용하는 JWT 토큰에 이슈가 있어서 좀 적어본다.

먼저 JWT(Json Web Token)란 Json 포맷을 이용하여 사용자에 대한 속성을 저장하는 Claim 기반의 Web Token을 말하는데 JWT는 토큰 자체를 정보로 사용하는 Self-Contained 방식으로 정보를 안전하게 전달하는 용도로 사용한다.

무상태성을 가지기 때문에 서버의 부하를 줄이고 확장성이 용이하다는 장점을 가지지만 아래와 같은 단점도 존재한다.

- claim에 넣는 데이터가 많아질 수록 JWT토큰이 길어지기에 그만큼 통신비용이 늘어난다.
- 그리고 JWT는 암호화를 하지 않기 때문에 패킷이 중간에 탈취당하는 경우 payload를 모두 볼 수 있다. 중요한 데이터는 넣으면 안된다.
- 표준 구현에서는 새로 고침 토큰(refresh token)을 지정하지 않는다. 따라서 만료 시 사용자는 다시 인증해야한다.
- 토큰의 상태관리를 하지 않기 때문에 만료시간이 되지 않는한 발행된 토큰을 임의로 로그아웃을 처리할 수 없다.

JWT를 구현할때 refresh token은 표준 구현이 아니기 때문에 구현을 놓치는 경우가 많다. 그래서 인증토큰의 만료시간을 길게 주어 사용자가 자주 로그인을 해야하는 번거로움을 줄이기도 하는데 이러한 방법은 보안 위험성을 가지고 있다. 왜냐하면 토큰이 혹시 탈취당하는 경우 서버에서는 토큰이 유효한 사용자인지 알 수 없기 때문이고 그리고 혹시나 알았더라도 임의로 로그아웃을 강제로 할 수 없기 때문이다.

refresh token을 사용하면 위와 같은 위험성을 대폭 낮출 수 있는데, 사용 플로우는 아래 그림과 같다.

![jwt-refresh-token-arch](/assets/images/retrospective/jwt-refresh-token-arch.png)

## kfactory produceMany 주의사항

[kfactory](https://github.com/bluegroundltd/kfactory){: target="\_blank" }는 test fixture 생성 도구로 [kotlinFixture](https://github.com/appmattus/kotlinfixture){: target="\_blank" }와 비교했을 때 보다 많은 유틸성 기능들을 제공해 주어서 현재 사내 테스트 fixture 생성도구로 사용되고 있다. kfactory의 유틸성 기능중에 다수의 fixture를 생성해주는 `produceMany` 함수가 있는데 해당 함수를 쓸때 겪었던 이슈를 적어본다.

먼저 Factory를 만들어주자. 2개의 변수에 난수값으로 fixture를 생성한다.

```kotlin
class FooFactory(
    private val a: String = UUID.randomUUID().toString(),
    private val b: Int = Random.nextInt(),
) : Factory<Foo> {
    override fun produce() = Foo(a, b)
}

data class Foo(
    val a: String,
    val b: Int,
)
```

이제 `produceMany`함수를 이용하여 여러개의 fixture를 생성해보자

```kotlin
class FooFactoryTest {
    @Test
    fun test() {
        val foos = FooFactory().produceMany().take(5).toList()
        println(foos)
        // Foo(a=e21a7dd1-e4d8-40fa-85e8-590c45af2ab5, b=-849739671)
        // Foo(a=e21a7dd1-e4d8-40fa-85e8-590c45af2ab5, b=-849739671)
        // Foo(a=e21a7dd1-e4d8-40fa-85e8-590c45af2ab5, b=-849739671)
        // Foo(a=e21a7dd1-e4d8-40fa-85e8-590c45af2ab5, b=-849739671)
        // Foo(a=e21a7dd1-e4d8-40fa-85e8-590c45af2ab5, b=-849739671)
    }
}
```

결과를 보면 알겠지만 예상과 달리 생성된 fixture들이 모두 똑같은 값을 가지고 있는 것을 볼 수 있다. 이는 최초 Factory 생성 후 produce()를 반복해서 실행하면서 fixture들을 만들기 때문에 Factory 생성 시점에 생성된 랜덤값들을 그대로 이용해서 발생한 결과일 것이다.

```kotlin
/**
  * Returns more than one object. Also allows for object modifications upon generation.
  *
  * @param tap modifies a produced object before returning it to the sequence generator
  * @return a Sequence of [produce]d objects
  */
fun produceMany(tap: T.() -> Unit = { }): Sequence<T> = generateSequence { produce().apply(tap) }
```

그렇다면 이 문제를 어떻게 해결할 수 있을까? `produce`시점에 값을 생성하도록 할 수 있지만 이는 factory 생성 시 `a`와 `b`의 변수를 임의로 지정할 수 없기 때문에 채택할 수 없다.

다음 방법은 바로 lambda를 이용하여 lazy하게 값을 생성하도록 하는 것이다. kfactory에서는 typealias로 `Yielded`라는 타입을 사용할 수 있도록 해준다. 변경된 코드는 아래와 같다.

```kotlin
class FooFactory(
    private val a: Yielded<String> = { UUID.randomUUID().toString() },
    private val b: Yielded<Int> = { Random.nextInt() },
) : Factory<Foo> {
    override fun produce() = Foo(a(), b())
}
```

테스트 코드를 실행해보자

```kotlin
class FooFactoryTest {
    @Test
    fun test() {
        val foos = FooFactory().produceMany().take(5).toList()
        println(foos)
        // Foo(a=c91684ba-7af9-40ba-9545-0417e18bf090, b=-1331977452)
        // Foo(a=a869b94f-5239-4071-bad8-27a996cc82a2, b=-2025852811)
        // Foo(a=e15a7187-133b-4cf3-b7c9-5764b8a34938, b=2094360656)
        // Foo(a=2e4eef9f-6071-4cfb-a2a1-65bf779c4e2e, b=40136848)
        // Foo(a=4d782dcc-3843-492e-adfe-ec57d3076de3, b=-1993056096)
    }
}
```

원하는대로 fixture가 모두 다르게 생성된 것을 확인할 수 있다.

## Github Pull Request View

Github에서 Pull Request 시 화면 표시 옵션이 있다는걸 [직방 기술블로그](https://medium.com/zigbang/%EC%8A%AC%EA%B8%B0%EB%A1%9C%EC%9A%B4-%EC%BD%94%EB%93%9C-%EB%A6%AC%EB%B7%B0-%EC%83%9D%ED%99%9C-with-github-pull-request-7932b5d47c70){: target="\_blank" }를 통해 알게 되었다.

코드 내용에 따라 보는 방법을 다르게 해서 좀더 원활하게 코드리뷰를 할 수 있을 듯 하다.

![github-pr-view](/assets/images/retrospective/github-pr-view.png)

## Spring Security를 이용한 JWT 구현

언어 전환을 하면서 기존에 JWT로 구현된 인증로직을 Spring Security를 이용하여 구현하게 되었다. 처음 구현해 보는 거라 여기저기 구현코드를 많이 찾아보았는데 하나같이 다들 DB에서 사용자 정보를 확인하는 로직으로 구현되어 있는것을 볼 수 있었다. (심지어 [JWT Wiki](https://sekurak.pl/jwt-security-ebook.pdf){: target="\_blank" }에서 조차)

JWT의 인증로직은 서버의 저장소를 요하지 않는다. 그렇기 때문에 트래픽의 증가로 인한 서버의 부하를 걱정하지 않을 수 있고 확장성이 뛰어나다는 장점을 가지는 것이다.

Spring Security에서 기본으로 제공해주는 구현체 중에는 이러한 요구사항을 충족하는 JWT Filter가 존재하지 않는듯 하여 새롭게 구현해 보았다. 더 자세한 내용은 [JWT를 이용한 Spring Security 인증 구현](https://veluxer62.github.io/tutorials/spring-security-with-jwt/){: target="\_blank" }글을 보면 좋겠다.

## coerceAtMost

Kotlin에서 제공해주는 함수중 최대값 또는 최소값을 구해주는 함수로 [coerceAtMost](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/coerce-at-most.html){: target="\_blank" }가 있다. 이런 기능은 직접 구현해 줄 수도 있지만 언어레벨에서 제공해주면 신뢰도 높게 유용하게 사용할 수 있어서 좋다.

## marimba

온라인 화이트보드 협업툴로 [marimba](https://www.marimba.team/kr/){: target="\_blank" }라는게 있다. 아직 베타긴 한데 무료버전으로 사용할 수 있는듯 하니 한번 써봐야겠다.

## Codecov

README에 Code coverage를 표시하기 위해서 [jacoco-badge-gradle-plugin](https://github.com/dawnwords/jacoco-badge-gradle-plugin){: target="\_blank" }을 사용중이었다. jacoco에서 제공하는 coverage를 모두 뱃지형태로 표현할 수 있다는 장점이 있지만 README를 수정할때마다 매번 커밋을 새로올려야 한다는 단점이 존재한다.

그래서 도입한 것이 [Codecov](https://about.codecov.io/){: target="\_blank" }이다. codecov는 ci를 codecov 서버에 코드를 전송하고 자체적으로 coverage를 생성하여 뱃지형태로 제공해준다. 무엇보다 뱃지 업데이트에 대한 수고스러움이 없다는 장점과 codecov 서버에 코드 상태를 표시해주는 대시보드도 함께 제공해주고 있어서 유용한점이 많다. 다만 [보안 이슈](https://blog.gitguardian.com/codecov-supply-chain-breach/amp/){: target="\_blank" }가 있었단 점과 coverage 뱃지가 line coverage 밖에 제공해주지 않다는 점은 아쉽다. (설정값이 있는진 좀 더 찾아봐야겠다.)

## spring-hibernate-query-utils

[spring-hibernate-query-utils](https://github.com/yannbriancon/spring-hibernate-query-utils){: target="\_blank" }는 JPA를 이용한 어플리케이션 개발 시 발생할 수 있는 N+1 문제를 감지해주는 도구이다. ORM으로 개발하다보면 N+1 문제는 쉽게 직면할 수 있는데, 이 문제를 개발 단계에서(테스트를 작성하면서 개발한다면) 조기에 발견할 수 있기 때문에 유용한 도구이다.

## neofetch

콘솔 실행 시 시스템의 상태를 보여주는 도구로 [neofetch](https://github.com/dylanaraps/neofetch){: target="\_blank" }가 있다.

## DGS Mapping existing types

DGS에서 Graphql custom type을 특정 클래스로 매핑 시켜주려면 아래와 같이 gradle 설정이 필요하다.

```kotlin
generateJava{
   typeMapping = ["MyGraphQLType": "com.mypackage.MyJavaType"]
}
```

참고: [https://netflix.github.io/dgs/generating-code-from-schema/#mapping-existing-types](https://netflix.github.io/dgs/generating-code-from-schema/#mapping-existing-types){: target="\_blank" }

## archunit

[Archunit](https://www.archunit.org/){: target="\_blank" }은 현재 구성된 어플리케이션에 팀내 정해진 아키텍쳐 정책을 지키는지 체크하고 강제하는 도구이다. 정해진 아키텍쳐에 따라 개발하도록 유도하는 것은 유지보수 측면에서 필요한 것이라 생각하지만 특정 패턴을 강제하는 것은 조금 위험 할 수 있다라는 생각이 든다. (예를 들어 서비스 명은 `**Service`라던지...) 그리고 아키텍쳐 정책은 코드리뷰를 통해서 유지하도록 하는게 더 좋지 않을까 라는 생각이 든다. (SI와 같이 외부 개발자와 함께 하는 경우라면 모를까)

## Kotlin String template에서 `$`문자열 사용

kotlin의 String template에서 `$`를 사용해야 하는 경우 어떻게 하면 좋을까? `\\$`를 사용하면 되겠지라는 생각을 했지만 잘되지 않았다. 다행스럽게도(?) [공식문서](https://kotlinlang.org/docs/basic-types.html#string-templates){: target="\_blank" }에 사용방법이 적혀있었다. 다만, 사용방법은 별론거 같다...

```kotlin
val price = """
${'$'}_9.99
"""
```
