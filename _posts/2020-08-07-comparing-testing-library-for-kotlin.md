---
layout: single
title: "Comparing Testing Library for Kotlin"
categories:
  - EXPLANATION
tags:
  - junit
  - kotlin test
  - spock
  - kolin
  - java

toc: true
toc_sticky: true
excerpt: Test의 중요성이 부각되면서 요즘 개발을 할때 테스트 코드를 많이 작성한다. 어쩌면 오퍼레이션 코드보다 테스트 코드가 프로젝트에서 더 많이 작성되기도 할지 모르겠다. 테스트 코드는 오퍼레이션 코드 못지 않게 아니 어쩌면 더 높은 가독성을 목표로 작성되어야 한다. JVM환경에서 자동화된 테스트를 도와주는 많은 도구가 있는데, 이 글은 Kotlin으로 Spring Framework에서 사용할 수 있는 테스트 도구를 비교하고 각각 사용해 보면서 느꼈던 장단점을 적어보고자 한다.
---

## Introduce

Test의 중요성이 부각되면서 요즘 개발을 할때 테스트 코드를 많이 작성한다. 어쩌면 오퍼레이션 코드보다 테스트 코드가 프로젝트에서 더 많이 작성되기도 할지 모르겠다. 테스트 코드는 오퍼레이션 코드 못지 않게 아니 어쩌면 더 높은 가독성을 목표로 작성되어야 한다. JVM환경에서 자동화된 테스트를 도와주는 많은 도구가 있는데, 이 글은 Kotlin으로 Spring Framework에서 사용할 수 있는 테스트 도구를 비교하고 각각 사용해 보면서 느꼈던 장단점을 적어보고자 한다.

### JUnit

[JUnit](https://junit.org/junit5/){: target="\_blank" }은 Java 또는 Kotlin을 이용하여 Spring Framework로 어플리케이션을 개발해본 개발자라면 누구나 알고 있는 테스팅 도구라 해도 과언이 아닐 정도로 유명한 테스팅 도구이다. Spring Boot를 사용할때 [initializer](https://start.spring.io/){: target="\_blank" }를 이용하여 WEB 프로젝트를 구성하면 기본적으로 의존성이 추가될 정도로 보편적인 도구이다.

### Spock

[Spock](http://spockframework.org/){: target="\_blank" }은 Java와 Groovy 어플리케션을 위한 테스트 프레임워크이다. 동적 타이핑 언어인 그루비를 사용하며 테스트 코드가 JUnit에 비해서 많이 간결해지고 BDD 테스트가 용이하다는 장점등을 가지고 있어 많은 개발자가 테스트를 위한 도구로 많이 활용하고 있다.

### Kotest

[Kotest](https://github.com/kotest/kotest){: target="\_blank" }는 현재 각광받고 있는 언어인 Kotlin을 위한 테스팅 도구이다. Kotlin-Test에서 Kotest로 이름이 변경되었으며 이름에서 알 수 있듯이 Kotlin언어로 작성된 코드를 테스트하기에 아주 용이하다. Spock와 유사하게 간결한 코드와 다양한 유형의 테스트 방식을 지원한다. Kotlin을 위한 테스팅 도구로 [Speck](https://www.spekframework.org/){: target="\_blank" }도 있지만 Kotest와 거의 비슷하고 Kotest가 좀더 문서화가 잘되어 있고 Spring의 지원등을 이유로 Kotest를 사용하기로 하였다.

## Code Sample

거두 절미 하고 코드로 각 테스팅 도구별로 작성해보자. Spring 환경에서 사용되는 대표적인 테스트 방식 몇가지를 소개할 것이고 BDD 방식의 테스트도 함께 소개해 보겠다.

### JUnit

Simple Testing

```kotlin
class CalculatorTest {
    @Test
    fun `sum함수는 숫자 1과 2 주어지면 3을 반환한다`() {
        // Given가
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

<br>

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

<br>

Spring Mock Testing

```kotlin
@SpringBootTest(classes = [SimplePersonService::class])
class PersonServiceTest {

    @Autowired
    private lateinit var personService: PersonService

    @MockBean
    private lateinit var personRepository: PersonRepository

    @Test
    fun `update함수는 UpdatePersonData데이터가 주어지면 PersonRepository.save 함수를 올바르게 호출한다`() {
        // given
        val id = "test-id"
        val given = UpdatePersonData("김삿갓", "foo@gmail.com", "01022221111", 40)

        Mockito.`when`(personRepository.findById(id))
            .thenReturn(
                Optional.of(Person("test-id", "홍길동", "test@gmail.com", "01011112222", 30))
            )

        val expected = Person("test-id", "김삿갓", "foo@gmail.com", "01022221111", 40)

        // when
        personService.update(id, given)

        // then
        Mockito.verify(personRepository).save(expected)
    }
}
```

<br>

BDD Testing

```kotlin
@DisplayName("Calculator 클래스")
class CalculatorTest {

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
            fun `it return 3`() {
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
            fun `it return 8`() {
                val actual = Calculator.sum(a, b)
                Assertions.assertEquals(8, actual)
            }
        }

    }

}
```

### Spock

Simple Testing

```groovy
class CalculatorTest extends Specification {

    def "sum함수는 숫자 1과 2 주어지면 3을 반환한다"() {
        given:
        def a = 1
        def b = 2
        def expected = 3

        when:
        def actual = Calculator.INSTANCE.sum(a, b)

        then:
        actual == expected
    }

}
```

<br>

Data Driven Testing

```groovy
class CalculatorTest extends Specification {

    def "sum함수는 숫자 a와 b 주어지면 a와 b가 더해진 값을 반환한다"() {
        when:
        def actual = Calculator.INSTANCE.sum(a, b)

        then:
        actual == expected

        where:
        a   | b   || expected
        1   | 2   || 3
        4   | 5   || 9
        100 | 999 || 1099
    }

}
```

<br>

Spring Mock Testing

```groovy
@SpringBootTest(classes = [SimplePersonService.class])
class PersonServiceTest extends Specification {

    @SpringBean
    private PersonRepository personRepository = Mock()

    @Autowired
    private PersonService personService

    def "update함수는 UpdatePersonData데이터가 주어지면 PersonRepository.save 함수를 올바르게 호출한다"() {
        given:
        def id = "test-id"
        def data = new UpdatePersonData("김삿갓", "foo@gmail.com", "01022221111", 40)

        personRepository.findById(id) >>
                Optional.of(new Person("test-id", "홍길동", "test@gmail.com", "01011112222", 30))

        def expected = new Person("test-id", "김삿갓", "foo@gmail.com", "01022221111", 40)

        when:
        personService.update(id, data)

        then:
        1 * personRepository.save(expected)
    }
}
```

<br>

BDD Testing

```groovy
class CalculatorTest extends Specification {

    def "sum함수는 숫자 a 와 b가 주어지면 a와 b의 합계를 반환한다"() {
        given: "숫자 1이 주어지고"
        def a = 1
        and: "숫자 2가 주어지면"
        def b = 2

        when: "sum 함수는"
        def actual = Calculator.INSTANCE.sum(a, b)

        then: "3을 반환한다"
        actual == expected
    }

}
```

### Kotest

Simple Testing

```kotlin
class CalculatorTest : StringSpec({

    "sum함수는 숫자 1과 2 주어지면 3을 반환한다" {
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

<br>

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

<br>

Spring Mock Testing

```kotlin
@SpringBootTest(classes = [SimplePersonService::class])
class PersonServiceTest : StringSpec() {

    override fun listeners() = listOf(SpringListener)

    @MockBean
    private lateinit var personRepository: PersonRepository

    @Autowired
    private lateinit var personService: PersonService

    init {

        "update will update person correctly" {
            // given
            val id = "test-id"
            val data = UpdatePersonData("김삿갓", "foo@gmail.com", "01022221111", 40)

            `when`(personRepository.findById(id))
                .thenReturn(
                    Optional.of(Person("test-id", "홍길동", "test@gmail.com", "01011112222", 30))
                )

            val expected = Person("test-id", "김삿갓", "foo@gmail.com", "01022221111", 40)

            // when
            personService.update(id, data)

            // then
            verify(personRepository).save(expected)
        }

    }
}
```

<br>

BDD Testing

```kotlin
class CalculatorDCITest : WordSpec({
    "sum 함수는" `when` {
        "숫자 1과 2가 주어지면" should {
            val a = 1
            val b = 2
            "3을 반환한다" {
                val expected = 3
                val actual = Calculator.sum(a, b)
                actual shouldBe expected
            }
        }
    }
})
```

## Pros & Cons

개인적으로 느꼈던 각 도구 별 장단점은 아래와 같다.

### JUnit

- 보편적이다. 그렇기 때문에 많은 레퍼런스가 있으며 JUnit 5로 오면서 JUnit 4에서 느꼈던 불편함들을 많이 해소하였다.
- 비교 대상들에 비해 가독성이 많이 떨어진다. Spock이나 Kotest에 비해서 JUnit은 문법적으로 작성해야할 코드가 많아 가독성이 떨어질 수 있다.
- 다양한 Assertion 라이브러리가 존재하므로 처음 테스트를 하는 개발자라면 동일한 함수명을 가진 패키지 중 어떤것을 선택해야 하는지 어려움이 있다.
- BDD 테스트가 번거롭다. 위의 코드에서도 보았지만 BDD 테스트를 위해서 inner class를 만들어 줘야 하는 등 번거러움이 있다. 이는 BDD 테스트를 주저하게 만드는 요인이 된다.

### Spock

- 간결하다. 코드를 보면 알수 있지만 테스트 코드가 엄청 간결해서 가독성을 엄청나게 높일 수 있다. 특히 Data Driven Testing에서 간결함은 3개의 테스트 도구중에 가장 좋아 보인다.
- DBB 테스트가 용이하다. 기본적으로 Given-When-Then 패턴을 사용하도록 강제하고 있다. 이는 개발자가 좀더 가독성이 높은 테스트를 작성하도록 유도한다는 점에서 높은 점수를 주고 싶다.
- Groovy를 배워야 한다. 하나의 어플리케이션에 2개의 언어를 사용한다는 부분은 개발자에게 부담이 될 수 있다. 물론 Groovy 언어가 Java를 사용하는 개발자에게 높은 진입장벽이 있는 것은 아니지만 테스트 코드를 작성할 때 테스트를 위한 여러 기능을 만들어야 할 수도 있는데 Groovy로 작성하는 것이 조금 아쉬운 부분이다.
- Kotlin을 사용한다면 Spock의 사용을 고민해보아야 한다. Java를 Spock로 테스트하는 것은 큰 어려움이 없지만 Kotlin을 Spock로 테스트할 때 Extension function, inline function, Kotlin 언어에서 지원하는 기본 function 등이 지원되지 않아 테스트 자체를 할 수 없는 경우가 발생할 수 있다. 이러한 문제는 Junit이나 Kotest로 작성해도 되지만 테스트 도구를 하나 이상 사용하는 것은 오히려 테스트 코드를 봐야하는 동료들에게 더 큰 부담을 줄 수 있을 거란 생각이 든다.
- 추가 의존성이 필요하다. Spring 환경에서 테스트를 하기 위해서는 JUnit은 필수로 추가되어야 하기 때문에 JUnit외에 추가적인 의존성 추가는 큰 불편함은 아니지만 단점이 될 수 있다.

### Kotest

- JUnit보다 간결하다. 하지만 Spock 만큼 임팩트가 크진 않지만 테스트 함수를 작성하는 부분과 특히 Assertion 코드가 간결해 지는 부분은 장점이다.
- BDD 용이하다. Kotest에서는 다양한 [테스팅 스타일](https://github.com/kotest/kotest/blob/master/doc/styles.md){: target="\_blank" }을 지원한다. 개발자는 원하는 테스팅 스타일을 선택해서 테스트 코드를 작성할 수 있고 JUnit과 비교했을때 무척 간결하게 BDD 테스트 코드를 작성할 수 있으므로 BDD 테스트에 대한 부담을 많이 줄일 수 있다.
- Kotlin과 궁합이 아주 잘 맞다. 태생이 Kotlin을 위한 테스트 도구이므로 Kotlin으로 작성된 어플리케이션에서 테스트 작성이 아주 용이하다.
- 추가 의존성이 필요하다. Spock과 마찬가지로 JUnit 이외에 Kotest를 위한 의존성을 추가해야 한다.

## Wrap Up

지금까지 3개의 테스팅 도구를 비교해 보았다. 개인적으로는 Java로 어플리케이션을 개발한다면 Spock을 Kotlin으로 어플리케이션을 개발한다면 Kotest를 사용해보면 어떨까 생각된다. 기존에 이미 JUnit으로 개발되어 있다면 새로운 도구로 전부다 전환하는게 아니라면 JUnit을 그대로 사용하되 BDD 방식으로 테스트를 도입해보는 것도 좋아 보인다. 어떤 도구가 더 좋은지는 각자 취향에 따라 다른것 같다. 상황에 맞게 적절한 테스팅 라이브러리를 선택하면 좋을 것 같다.

> [https://junit.org/junit5/](https://junit.org/junit5/){: target="\_blank" }
>
> [https://start.spring.io/](https://start.spring.io/){: target="\_blank" }
>
> [http://spockframework.org/](http://spockframework.org/){: target="\_blank" }
>
> [https://github.com/kotest/kotest](https://github.com/kotest/kotest){: target="\_blank" }
>
