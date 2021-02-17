---
layout: single
title: "Spring retry"
categories:
  - POST
---

# Kotest

[kotest/kotest](https://github.com/kotest/kotest)

Kotlin으로 작성된 코드의 테스트를 위한 테스팅 도구이다. Junit에 비해 간결한 문법과 다양한 테스팅 방법을 제공해주며 Spring 지원도 해준다.

# TDD

Test Class

```kotlin
object Calculator {
    fun sum(a: Int, b: Int): Int {
        return a + b
    }
}
```

## Kotlin으로 작성해본 Junit Testing

Simple Testing

```kotlin
class CalculatorTest {

    @Test
    fun `sum함수는 숫자 a와 b가 주어지면 a와 b가 더해진 값을 반환한다`() {
        // Given
        val a = 1
        val b = 2
        val expected = 3

        // When
        val actual = Calculator.sum(a, b)

        // Then
        Assertions.assertEquals(expected, actual)
    }

}
```

Data Driven Testing

```kotlin
class CalculatorTest {

    @ParameterizedTest
    @ArgumentsSource(SumArguments::class)
    fun `sum함수는 숫자 a와 b가 주어지면 a와 b가 더해진 값을 반환한다`(
        a: Int, b: Int, expected: Int
    ) {
        // When
        val actual = Calculator.sum(a, b)

        // Then
        Assertions.assertEquals(expected, actual)
    }

    class SumArguments : ArgumentsProvider {
        override fun provideArguments(context: ExtensionContext?): Stream<out Arguments> {
            return Stream.of(
                Arguments.of(1, 2, 3),
                Arguments.of(4, 5, 9),
                Arguments.of(100, 999, 1099)
            )
        }
    }

}
```

## Kotest로 작성해본 Testing

Simple Testing

```kotlin
class CalculatorTest : StringSpec({

    "sum함수는 숫자 a와 b가 주어지면 a와 b가 더해진 값을 반환한다" {
        // Given
        val a = 1
        val b = 2
        val expected = 3

        // When
        val actual = Calculator.sum(a, b)

        // Then
        actual shouldBe expected
    }
    
})
```

Data Driven Testing

```kotlin
class CalculatorTest : StringSpec({

    "sum함수는 숫자 a와 b가 주어지면 a와 b가 더해진 값을 반환한다" {
        forall(
            row(1, 2, 3),
            row(4, 5, 9),
            row(100, 999, 1099)
        ) { a, b, expected ->
            // When
            val actual = Calculator.sum(a, b)

            // Then
            actual shouldBe expected
        }
    }

})
```

# BDD

[Behavior-driven development](https://en.wikipedia.org/wiki/Behavior-driven_development)

BDD란 Behaviour-Driven Development의 약자로 TDD를 기반으로 하고 있으며 TDD와는 거의 차이가 없다고 보면 되지만 테스트 케이스 작성 시 좀더 우리가 사용하는 언어와 친화적으로 작성을 하는 방식을 말한다. 

시나리오 작성 패턴은 여러가지가 있지만 대표적으로 **Given-When-Then 패턴**과 **Describe-Context-It 패턴**이 있다. Given-When-Then 패턴은 Spock에서 흔히 볼수 있는 패턴이고 Describe-Context-It 패턴은 Jest에서 흔히 볼 수 있는 패턴이다.

## Junit

Given-When-Then

```kotlin
class CalculatorTest {

    @Test
    fun `sum함수는 숫자 a와 b가 주어지면 a와 b가 더해진 값을 반환한다`() {
        // Given
        val a = 1
        val b = 2
        val expected = 3

        // When
        val actual = Calculator.sum(a, b)

        // Then
        Assertions.assertEquals(expected, actual)
    }

}
```

Describe-Contxt-It

```kotlin
@DisplayName("Calculator 클래스")
class CalculatorBDDTest {

    @Nested
    @DisplayName("sum 함수는")
    inner class Describe_sum {

        @Nested
        @DisplayName("숫자 1과 2가 주어지면")
        inner class Context_with_1_and_2 {
            private val a = 1
            private val b = 2

            @Test
            @DisplayName("3을 반환한다")
            fun `it return sum number`() {
                val actual = Calculator.sum(a, b)
                Assertions.assertEquals(3, actual)
            }
        }

        @Nested
        @DisplayName("숫자 3과 5 주어지면")
        inner class Context_with_3_and_5 {
            private val a = 3
            private val b = 5

            @Test
            @DisplayName("8을 반환한다")
            fun `it return sum number`() {
                val actual = Calculator.sum(a, b)
                Assertions.assertEquals(8, actual)
            }
        }

    }

}
```

## Kotest

[kotest/kotest](https://github.com/kotest/kotest/blob/master/doc/styles.md)

BehaviorSpec

```kotlin
class CalculatorGWDTest : BehaviorSpec({
    given("sum함수는") {
        `when`("숫자 a와 b가 주어지면") {
            val a = 1
            val b = 2
            then("a와 b가 더해진 값을 반환한다") {
                val expected = 3
                val actual = Calculator.sum(a, b)
                actual shouldBe expected
            }
        }
    }
})
```

WordSpec

```kotlin
class CalculatorDCITest : WordSpec({
    "sum 함수는" `when` {
        "숫자 a와 b가 주어지면" should {
            val a = 1
            val b = 2
            "a와 b가 더해진 값을 반환한다" {
                val expected = 3
                val actual = Calculator.sum(a, b)
                actual shouldBe expected
            }
        }
    }
})
```

# 느낀점

- BDD를 사용한다면 Kotest, 아니라면 Junit을 사용해도 큰 차이가 없어 보인다. (Spock과 너무 비교된다)
- Kotlin을 사용하지 않는다면 Spock을 사용했을 것이다.
- BDD를 사용해보자!

# ETC

Spek 이라는 프레임워크도 있지만 좀 마이너한 느낌과 문서화, 스프링 지원 등이 Kotest에 비해 약하다고 판단해서 사용을 고려하진 않았다.

[Spek Framework](https://www.spekframework.org/)

# 샘플코드

[https://github.com/veluxer62/comparison-testing-library](https://github.com/veluxer62/comparison-testing-library)