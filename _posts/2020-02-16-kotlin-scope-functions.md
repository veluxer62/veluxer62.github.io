---
layout: single
title: "Kotlin Scope Functions"
categories:
  - EXPLANATION
tags:
  - kotlin
  - functions
  - let
  - run
  - with
  - apply
  - also
toc: true
toc_sticky: true
excerpt: Kotlin에서 제공하는 함수들 중에 `let`, `run`, `with`, `apply`, `also`라는 함수들에 대해서 알아보자.
---

Kotlin에서 제공하는 함수들 중에 `let`, `run`, `with`, `apply`, `also`라는 함수들이 있다. 함수명은 다르지만 얼핏 보면 비슷한 동작들을 하고 있고 실제로 사용할때 다른함수로 사용해도 기능이 동일하게 동작하기도 하는 함수도 있다. 그럼 Kotlin에서는 왜 하나의 함수로 만들지 않고 이 함수들을 구분해 놓았을까?

이 글에서는 Kotlin [DSL](https://kotlinlang.org/docs/reference/scope-functions.html#functions){: target="\_blank" } 에서 정의해 놓은 사용법을 보고 그 용도를 구분해 보고자 한다.

## let

`let`함수는 **확장함수**이다. `it`을 사용하며 **lambda 실행 결과**를 반환한다.

콜체인의 결과에 대한 하나 또는 그 이상의 기능을 수행하는데 사용된다.

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let {
    println(it)
    // and more function calls if needed
}
```

<br/>

Null이 아닌 코드 블록을 실행하는데 사용된다.

```kotlin
val str: String? = "Hello"
// compilation error: str can be null
//processNonNullString(str)
val length = str?.let {
    println("let() called on $it")
    // OK: 'it' is not null inside '?.let { }'
    processNonNullString(it)
    it.length
}
```

<br/>

코드의 가독성을 위해서도 사용된다. 아래 예제에서는 가독성을 위해 `it` 대신에 `firstItem`을 사용하였다.

```kotlin
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first().let { firstItem ->
    println("The first item of the list is '$firstItem'")
    if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
}.toUpperCase()
println("First item after modifications: '$modifiedFirstItem'")
```

## with

`with`는 **비확장 함수**이다. `this`를 사용하며 **lambda 실행 결과**를 반환한다.

Kotlin에서는 `with`사용 시 lambda 결과를 반환하지 않는 것을 권장한다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}
```

<br/>

속성이나 기능을 계산하는 도우미 역할을 하기도 한다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
println(firstAndLast)
```

## run

`run`함수는 **확장함수**이다. `this`을 사용하며 **lambda 실행 결과**를 반환한다. `let`과 유사하다.

개체를 초기화 하거나 계산한 값을 반화하는데 사용된다.

```kotlin
val service = MultiportService("https://example.kotlinlang.org", 80)

val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}
```

<br/>

비확장 기능으로도 사용할 수 있는데, 가독성을 위한 코드블록을 생성할때 사용되기도 한다.

```kotlin
val hexNumberRegex = run {
    val digits = "0-9"
    val hexDigits = "A-Fa-f"
    val sign = "+-"

    Regex("[$sign]?[$digits$hexDigits]+")
}

for (match in hexNumberRegex.findAll("+1234 -FFFF not-a-number")) {
    println(match.value)
}
```

## apply

`apply`함수는 **확장함수**이다. `this`을 사용하며 **개체 자신**을 반환한다.

개체에 값을 할당할때 주로 사용된다.

```kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"
}
println(adam)
```

## also

`apply`함수는 **확장함수**이다. `it`을 사용하며 **개체 자신**을 반환한다.

logging, print와 같이 개체를 변경하지 않는 추가작업에 주로 사용된다. 그래서 해당 구문을 제거해도 로직은 변경되지 않는다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")
```

## 함수의 선택

| function | Object reference |  Return value  | Is extension function | Purpose                                         |
| :------: | :--------------: | :------------: | :-------------------: | :---------------------------------------------- |
|   let    |        it        | Lambda Result  |          Yes          | Null이 아닌 코드블록 실행, 로컬변수 표현식 소개 |
|   run    |       this       | Lambda Result  |          Yes          | 개체 설정 또는 계산                             |
|   run    |        -         | Lambda Result  |          No           | 표현식의 범위 실행                              |
|   with   |       this       | Lambda Result  |          No           | 개체의 함수호출 그룹화                          |
|  apply   |       this       | Context object |          Yes          | 개체 설정                                       |
|   also   |        it        | Context object |          Yes          | 효과 추가                                       |

## it과 this의 차이

코드 블록 내 lambda 매개변수인 `it`의 사용과 receiver인 `this`를 구분해서 사용하는 이유가 따로 있지 않을까 해서 찾아보았으나 성능이나 다른 특별한 이유는 찾을 수 없었고 **가독성**을 위해서 라는 의견을 여러 글에서 볼 수 있었다. ([stackoverflow](https://stackoverflow.com/questions/47329716/what-is-a-purpose-of-lambdas-with-receiver){: target="\_blank" }, [medium](https://medium.com/tompee/idiomatic-kotlin-lambdas-with-receiver-and-dsl-3cd3348e1235){: target="\_blank" })

## Conclusion

**Kotlin DSL**에서 권장하는 방식대로 사용하지 않아도 기능동작에는 아무 문제가 되지 않는다. 하지만 서로 약속된 의미가 아닌 용도로 사용한다면 그 코드를 보는 동료들에게 큰 잘못을 하는 행동이 될 것이다. 이번 글을 정리하면서 단순히 정의된 함수를 생각없이 사용하기 보다 그 함수가 의도하는 바가 무엇인지 비교하고 찾아볼 수 있게 되어서 의미있는 시간이었다고 생각된다.

마지막으로 [@limgyumin 님께서 작성하신 블로그](https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29){: target="\_blank" }에서 사용하신 예제 코드가 인상 깊어서 인용하는 것으로 마무리한다.

```kotlin
private fun insert(user: User) = SqlBuilder().apply {
  append("INSERT INTO user (email, name, age) VALUES ")
  append("(?", user.email)
  append(",?", user.name)
  append(",?)", user.age)
}.also {
  print("Executing SQL update: $it.")
}.run {
  jdbc.update(this) > 0
}
```

> [https://kotlinlang.org/docs/reference/scope-functions.html#functions](https://kotlinlang.org/docs/reference/scope-functions.html#functions){: target="\_blank" }
>
> [https://stackoverflow.com/questions/47329716/what-is-a-purpose-of-lambdas-with-receiver](https://stackoverflow.com/questions/47329716/what-is-a-purpose-of-lambdas-with-receiver){: target="\_blank" }
>
> [https://medium.com/tompee/idiomatic-kotlin-lambdas-with-receiver-and-dsl-3cd3348e1235](https://medium.com/tompee/idiomatic-kotlin-lambdas-with-receiver-and-dsl-3cd3348e1235){: target="\_blank" }
>
> [https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29](https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29){: target="\_blank" }
