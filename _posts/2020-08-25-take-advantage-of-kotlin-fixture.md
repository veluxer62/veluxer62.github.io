---
layout: single
title: "Take advantage of Kotlin Fixture"
categories:
  - TUTORIALS
tags:
  - kotlin
  - fixture
  - kotlinfixture

toc: true
toc_sticky: true
excerpt: 테스트코드를 작성하다보면 테스트 데이터를 필수적으로 수없이 생성해야 한다. 테스트 데이터를 생성하는 작업은 상당히 번거롭고 지루한 작업이다. 하지만 테스트 데이터를 어떻게 잘 작성하느냐에 따라서 테스트 코드의 가독성이 좋아지기도 나빠지기도 하며 중복되는 코드로 인해 테스트 코드의 변경이 어려워 지기도 한다. 그래서 테스트 데이터는 테스트에서 중요한 역할을 한다. 그러면 어떻게 하면 이렇게 중요한 테스트 데이터를 번거롭지 않게 생성할 수 있을까? .NET 진영에서는 AutoFixture라는 도구가 많이 사용되는 것 처럼 보이고, Java 진영에서는 JFixture라고 AutoFixture를 모티브로 한 라이브러리가 존재한다. Kotlin 진영에서는 바로 이 글에서 소개할 KotlinFixture라는 라이브러리가 있다. 그럼 KotlinFixture에 대해서 알아보고 테스트 데이터를 손쉽게 생성하는 방법에 대해서 알아보자.
---

## Intro

테스트코드를 작성하다보면 테스트 데이터를 필수적으로 수없이 생성해야 한다. 테스트 데이터를 생성하는 작업은 상당히 번거롭고 지루한 작업이다. 하지만 테스트 데이터를 어떻게 잘 작성하느냐에 따라서 테스트 코드의 가독성이 좋아지기도 나빠지기도 하며 중복되는 코드로 인해 테스트 코드의 변경이 어려워 지기도 한다. 그래서 테스트 데이터는 테스트에서 중요한 역할을 한다.

그러면 어떻게 하면 이렇게 중요한 테스트 데이터를 번거롭지 않게 생성할 수 있을까? .NET 진영에서는 [AutoFixture](https://github.com/AutoFixture/AutoFixture){: target="\_blank" }라는 도구가 많이 사용되는 것 처럼 보이고, Java 진영에서는 [JFixture](https://github.com/FlexTradeUKLtd/jfixture){: target="\_blank" }라고 AutoFixture를 모티브로 한 라이브러리가 존재한다. Kotlin 진영에서는 바로 이 글에서 소개할 [KotlinFixture](https://github.com/appmattus/kotlinfixture){: target="\_blank" }라는 라이브러리가 있다. 그럼 KotlinFixture에 대해서 알아보고 테스트 데이터를 손쉽게 생성하는 방법에 대해서 알아보자.

## KotlinFixture

[KotlinFixture](https://github.com/appmattus/kotlinfixture){: target="\_blank" }는 Kotlin에서 객체를 무작위 데이터로 생성하는 도구이다. 

## Configure Dependencies

아래와같이 그레이들에 의존성을 추가하여 사용할 수 있다. (Repository가 `jcenter`인 점을 꼭 확인하자)

```kotlin
// build.gradle.kts

// ...

repositories {
    // ...
    jcenter()
}

dependencies {
    // ...
    testImplementation("com.appmattus.fixture:fixture:<latest-version>")
}

// ...
```

## Simple usage

간단한 사용방법은 아래와 같다. (JUnit을 활용하였다.)

```kotlin
package com.example.demo

import com.appmattus.kotlinfixture.kotlinFixture
import org.junit.jupiter.api.Test

class FixtureTest {
    val fixture = kotlinFixture()

    @Test
    fun test() {
        val list = fixture<List<String>>()
        println(list)
        // [ef155d42-9403-4eef-9b5b-a014263c7cd4, 220790a3-83c3-4672-8004-82dbea876e27, c3797302-0c84-452e-833b-d123993f17c3, a40abe71-27c5-425f-9d46-d4a0b4ca93da, a44fe37a-0b76-4e63-a74a-d3e6ec95c187]
        // [f4ffcb9c-978e-4027-9320-d48d9fdf8e39, e5fbe226-a49d-41e8-98dc-861cfa6f0f0d, 3482e128-0487-4c11-b88d-a43143053c08, 9cc99012-3312-40c7-89ec-af22a32563cc, 892578d4-08ad-4de7-866e-21b029c57230]

        val nullable = fixture<Int?>()
        println(nullable)
        // null
        // -446835767

        val foo = fixture<Foo>()
        println(foo)
        // Foo(id=-3090483292758919833, name=ec08f6b6-b787-409d-9e83-ca5d6a20c66a, age=-1461297163)
        // Foo(id=-5993394505703398996, name=31fe8a8a-1712-484f-837a-5ce453abfedb, age=-452298330)

        val bar = fixture<Bar>()
        println(bar)
        // Bar(id=-459462610977611708, name=7cf22b12-fb59-49d0-8762-db38983ea2d2, age=960104796)
        // Bar(id=-2890819688809070161, name=b366f341-5953-41f3-98d3-469d05f50f67, age=null)
    }
}

data class Foo(
    val id: Long,
    val name: String,
    val age: Int
)

class Bar {
    var id = 0L
    var name = ""
    var age: Int? = null

    override fun toString() = "Bar(id=$id, name=$name, age=$age)"
}
```

## Configuration options

간단한 사용법만으로도 테스트 데이터를 생성하는 것으로도 번거로운 작업을 많이 덜 수 있다. 하지만 만약에 예제에서 작성한 `Foo`라는 클래스에 `id`속성이 0보다 무조건 큰 값이어야 한다는 제약조건이 있다면 어떻게 해야할까? 이러한 경우 사용하는 방법이 KotlinFixture의 `factory`를 사용하는 방법이 있다. 또한 배열의 경우 배열에 담길 값의 개수를 `repeatCount`를 설정함으로써 조절이 가능하다.

설정하는 방법은 다양한데, 위 코드에서 보면 `kotlinFixture()`에서 설정해서 공통적으로 설정하거나 `fixture<T>()`에서 설정해서 개별적으로 설정이 가능하다.


### 공통 설정

아래 코드를 보면 `kotlinFixture`에서 `Int`형을 생성하는 경우 무조건 10이 반환되도록, `Foo`클래스를 생성하는 경우 무조건 `Foo(id = 1, name = "홍길동", age = 30)`를 생성하도록 설정해 두었고 아래 테스트 코드를 실행해보면 예상하는 바와 같이 생성됨을 확인 할 수 있다. 눈여겨 볼 것은 nullable한 타입도 null이 아닌 값으로 생성되는 경우 factory의 조건을 따른다는 것이다.

또, `List` 타입의 경우 아무 설정을 하지 않은 경우 5개의 데이터가 생성되는 것을 볼 수 있는데 `repeatCount`에서 설정한 값 만큼 생성 개수가 변경된 것을 확인할 수 있다.

```kotlin
package com.example.demo

import com.appmattus.kotlinfixture.kotlinFixture
import org.junit.jupiter.api.Test

class FixtureTest {
    val fixture = kotlinFixture {
        factory<Int> { 10 }
        factory<Foo> {
            Foo(id = 1, name = "홍길동", age = 30)
        }
        repeatCount { 3 }
    }

    @Test
    fun test() {
        val list = fixture<List<String>>()
        println(list)
        // [ef155d42-9403-4eef-9b5b-a014263c7cd4, 220790a3-83c3-4672-8004-82dbea876e27, c3797302-0c84-452e-833b-d123993f17c3]
        // [f4ffcb9c-978e-4027-9320-d48d9fdf8e39, e5fbe226-a49d-41e8-98dc-861cfa6f0f0d, 3482e128-0487-4c11-b88d-a43143053c08]

        val nullable = fixture<Int?>()
        println(nullable)
        // null
        // 10

        val foo = fixture<Foo>()
        println(foo)
        // Foo(id=1, name=홍길동, age=30)
        // Foo(id=1, name=홍길동, age=30)

        val bar = fixture<Bar>()
        println(bar)
        // Bar(id=5745977651577267862, name=f9f5b272-55e1-4eca-bbe8-2755c6ac8bd2, age=10)
        // Bar(id=-2890819688809070161, name=b366f341-5953-41f3-98d3-469d05f50f67, age=null)
    }
}

data class Foo(
    val id: Long,
    val name: String,
    val age: Int
)

class Bar {
    var id = 0L
    var name = ""
    var age: Int? = null

    override fun toString() = "Bar(id=$id, name=$name, age=$age)"
}
```

### 개별 설정

아래 코드를 보면 `kolinFixture` 설정외에 개별로도 설정을 할 수 있다. `String`타입인 경우 `test`라는 문구가 표시도록 수정하였다. 개별설정은 공통 설정을 기반으로 설정된다. 그래서 공통으로 설정된 값들도 함께 적용됨을 볼 수 있다. `Foo` 클래스의 경우 String 타입인 `name` 속성을 가지고 있지만 `Foo`클래스에 대한 factory가 설정되어 있기 때문에 개별 속성이 적용되지 않음을 확인할 수 있다.

```kotlin
package com.example.demo

import com.appmattus.kotlinfixture.kotlinFixture
import org.junit.jupiter.api.Test

class FixtureTest {
    val fixture = kotlinFixture {
        factory<Int> { 10 }
        factory<Foo> {
            Foo(id = 1, name = "홍길동", age = 30)
        }
    }

    @Test
    fun test() {
        val list = fixture<List<String>> {
            factory<String> { "test" }
        }
        println(list)
        // [test, test, test, test, test]
        // [test, test, test, test, test]

        val nullable = fixture<Int?>()
        println(nullable)
        // null
        // 10

        val foo = fixture<Foo> {
            factory<String> { "test" }
        }
        println(foo)
        // Foo(id=1, name=홍길동, age=30)
        // Foo(id=1, name=홍길동, age=30)

        val bar = fixture<Bar> {
            factory<String> { "test" }
        }
        println(bar)
        // Bar(id=5745977651577267862, name=test, age=10)
        // Bar(id=-2890819688809070161, name=test, age=null)
    }
}

data class Foo(
    val id: Long,
    val name: String,
    val age: Int
)

class Bar {
    var id = 0L
    var name = ""
    var age: Int? = null

    override fun toString() = "Bar(id=$id, name=$name, age=$age)"
}
```

## Wrap Up

위에서 살펴본 설정 말고도 값의 범위를 설정(`Range`)하거나, 특정 값들만 표시되도록 필터링(`filter`)하도록 하는 등 다양한 방법이 있다. 해당 방법은 [KotlinFixture Confituration Options](https://github.com/appmattus/kotlinfixture/blob/main/fixture/configuration-options.adoc){: target="\_blank" }링크를 참고하자.

KotlinFixture는 DTO나 Entity 객체 또는 상태를 가진 Domain 객체를 테스트할때 유용하게 사용할 수 있다. 특히 factory 설정을 통해 객체들이 가진 제약조건을 지키면서 객체를 생성할 수 있도록 하면 아주 효과적으로 테스트 데이터를 생성할 수 있고 테스트 코드가 높은 가독성을 유지할 수 있을 것이다. 단순히 의존성만 추가해서 바로 활용할 수 있으니 지금 바로 KotlinFixture 사용해 보는건 어떨까.

> [https://github.com/AutoFixture/AutoFixture](https://github.com/AutoFixture/AutoFixture){: target="\_blank" }
>
> [https://github.com/FlexTradeUKLtd/jfixture](https://github.com/FlexTradeUKLtd/jfixture){: target="\_blank" }
>
> [https://github.com/appmattus/kotlinfixture](https://github.com/appmattus/kotlinfixture){: target="\_blank" }
>
