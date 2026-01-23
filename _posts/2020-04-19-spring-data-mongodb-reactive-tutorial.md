---
layout: single
title: "Spring WebFlux - Spring Data MongoDB Reactive tutorial"
categories:
  - TUTORIALS
tags:
  - Spring WebFlux
  - kotlin
  - Spring Data
  - MongoDB

toc: true
toc_sticky: true
excerpt: 이전 글에서 RDBMS를 Spring WebFlux에서 사용하기 위해 Spring Data R2DBC 튜토리얼을 작성했었다. 이번 글에서는 Spring WebFlux에서 NoSQL중 하나인 MongoDB를 사용하기 위한 튜토리얼을 작성해 보고자 한다.
---

## Introduction

이전 글에서 RDBMS를 Spring WebFlux에서 사용하기 위해 [Spring Data R2DBC 튜토리얼](/tutorials/spring-data-r2dbc-tutorial/){: target="\_blank" }을 작성했었다. 이번 글에서는 Spring WebFlux에서 NoSQL중 하나인 MongoDB를 사용하기 위한 튜토리얼을 작성해 보고자 한다.

## Spring Data MongoDB

Spring Data에서 Document Database인 MongoDB 사용하기 위한 추상화 기능을 제공해 준다. Spring Data MongoDB에서 제공하는 많은 Repository중에서 우리는 비동기 작업을 위한 [ReactiveMongoRepository](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/repository/ReactiveMongoRepository.html){: target="\_blank" }를 사용할 예정이다.

**ReactiveMongoRepository**는 Spring Data 2.0 버전부터 추가된 Repository로 Spring WebFlux 튜토리얼을 보면 항상 Spring Data MongoDB를 이용하는 것을 볼 수 있듯이 활발하게 사용되고 있는 Repository이다.

## Getting Start

이제 **ReactiveMongoRepository**를 어떻게 사용하는지 알아보자. (프로젝트 생성은 [Spring initializer](https://start.spring.io/){: target="\_blank" })를 이용하였으며 GradleProject, Kotlin, SpringBoot 2.2.6 버전을 설정하여 생성하였다. 데이터베이스는 [Embedded MongoDB](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo){: target="\_blank" }를 선택하였다.)

### Gradle

**ReactiveMongoRepository**를 사용하기 위해서는 2개의 의존성 설정이 필요하다. 하나는 [Srping Data Reactive MongoDB 의존성](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-mongodb-reactive){: target="\_blank" }이고 다른 하나는 [Embedded MongoDB 의존성](https://mvnrepository.com/artifact/de.flapdoodle.embed/de.flapdoodle.embed.mongo)이다.

```gradle
implementation("org.springframework.boot:spring-boot-starter-data-mongodb-reactive")
implementation("de.flapdoodle.embed:de.flapdoodle.embed.mongo")
```

### Configuration

**Embedded MongoDB**를 사용하기 위해서는 별도의 설정은 필요하지 않다. 하지만 Repository들의 경로를 지정하거나 Property를 이용한 설정이 아닌 직접 클래스 파일로 MongoDB에 대한 설정을 하고 싶으면 Congifuation 클래스를 추가하여 설정할 수 있다. 자세한 사항은 [링크](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#mongo.reactive.repositories.usage){: target="\_blank" }를 참고 하자.

여기에서는 **Embedded MongoDB**를 사용할 예정이기 때문에 별도 설정을 하지 않겠다.

### Persistence Model

**Spring Data MongoDB**에서는 영속성 모델을 지정하기 위해서 `@Document` 어노테이션을 사용한다.
Primary Key를 지정해 주기 위해서 `@Id` 어노테이션을 사용한다.

```kotlin
@Document
data class Person(
    @Id
    val id: String?,
    val name: String,
    val age: Int
)
```

### Repository

다른 **Spring Data**와 비슷하게 **Spring Data MongoDB**도 인터페이스 정의만으로 레파지토리를 추상화 하고 있다. `ReactiveMongoRepository`인터페이스를 구현해서 새로운 인터페이스를 정의하면 된다.

```kotlin
interface PersonRepository : ReactiveMongoRepository<Person, String>
```

### Testing

사실 **Spring Data MongoDB**의 레파지토리에서 제공하는 기본 함수에 대한 테스트는 검증이 이미 된 함수이기 때문에 테스트 코드가 필요하지 않다. 하지만 우리는 **Practice Testing**을 통해 **Spring Data MongoDB**에서 제공하는 기능을 익힐 수 있으므로 테스트 코드를 작성해 보면 좋을 것 같다. (**Practice Testing**에 대한 장점은 [여기 글](https://www.ernweb.com/educational-research-articles/learning-techniques-effective-study/){: target="\_blank" }을 참고하자.)

먼저 테스트 클래스를 생성하자. Spring Boot 2.2 버전에서는 테스트용 도구로 [junit5](https://junit.org/junit5/docs/current/user-guide/){: target="\_blank" }를 사용하고 있다.
테스트 클래스임을 설정하기 위해 `@ExtendWith(SpringExtension::class)` 주석을 추가한다.
다음으로 **Spring Data MongoDB**에서는 유닛테스트를 위한 `@DataMongoTest`어노테이션을 지원해준다. 해당 어노테이션을 사용하면 내가 작성한 Repository와 Document들 그리고 [`ReactiveMongoTemplate`](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/ReactiveMongoTemplate.html){: target="\_blank" }을 사용할 수 있으며 별다른 설정이 없다면 기본적으로 **Embedded Mongo DB**를 사용한다.
`ReactiveMongoTemplate`는 데이터를 검증하는 용도로 사용될 객체이다.

```kotlin
@ExtendWith(SpringExtension::class)
@DataMongoTest
class PersonRepositoryTest(
    @Autowired
    private val template: ReactiveMongoTemplate,
    @Autowired
    private val repository: PersonRepository
) {

}
```

<br/>

각 테스트 별로 데이터에 따른 영향을 제거하기 위해 매 테스트 함수 실행 전에 데이터를 모두 지우는 함수를 추가한다.

```kotlin
@BeforeEach
fun beforeEach() {
    template.dropCollection(Person::class.java).subscribe()
}
```

<br/>

테스트를 도와주기 위한 함수를 먼저 추가한다. 해당 함수는 Document 데이터를 추가하는 기능을 하는 함수이다.

```kotlin
private fun generateDocument(): Person {
    return template.insert(
        Person(
            null,
            UUID.randomUUID().toString(),
            Random.nextInt()
        )
    ).block()!!
}
```

<br/>

insert 테스트를 추가해보자. repository의 `save`를 실행 한 후 저장된 데이터가 반환 되는지 확인한다.

```kotlin
@Test
fun `save will create data correctly`() {
    // Arrange
    val name = UUID.randomUUID().toString()
    val age = Random.nextInt()
    val document = Person(
        id = null,
        name = name,
        age = age
    )

    // Act
    repository.save(document)
        .`as` { StepVerifier.create(it) }
        .assertNext {
            assertThat(it.id).isNotNull()
            assertThat(it.name).isEqualTo(name)
            assertThat(it.age).isEqualTo(age)
        }
        .verifyComplete()
}
```

<br/>

update 테스트를 추가해보자. 데이터를 먼저 저장한 후 동일한 PK를 가진 다른 데이터를 저장하는 방식으로 진행하였다.
`ReactiveMongoTemplate`객체를 이용하여 실제로 데이터가 변경되었는지 확인한다.

```kotlin
@Test
fun `save will update data correctly`() {
    // Arrange
    val savedPerson = generateDocument()

    val name = UUID.randomUUID().toString()
    val age = Random.nextInt()

    val id = savedPerson.id.orEmpty()
    val updateDocument = Person(
        id = id,
        name = name,
        age = age
    )

    // Act & Assert
    repository.save(updateDocument)
        .`as` { StepVerifier.create(it) }
        .expectNextCount(1)
        .verifyComplete()

    template.findById(id, Person::class.java)
        .`as` { StepVerifier.create(it) }
        .assertNext {
            assertThat(it.id).isEqualTo(id)
            assertThat(it.name).isEqualTo(name)
            assertThat(it.age).isEqualTo(age)
        }
        .verifyComplete()
}
```

<br/>

전체 Document를 조회하는 기능을 테스트 해보자. 데이터를 먼저 저장한 후 반환되는 데이터 건수가 맞는지를 테스트한다.

```kotlin
@Test
fun `findAll will return flow of document correctly`() {
    // Arrange
    generateDocument()
    generateDocument()
    generateDocument()

    // Act & Assert
    repository.findAll()
        .`as` { StepVerifier.create(it) }
        .expectNextCount(3)
        .verifyComplete()
}
```

<br/>

PK를 이용하여 단일 Document를 조회하는 함수를 테스트해보자. 저장한 데이터와 조회한 데이터가 일치하는지 테스트 한다.

```kotlin
@Test
fun `findById will return document correctly`() {
    // Arrange
    val savedData = generateDocument()

    // Act & Assert
    repository.findById(savedData.id.orEmpty())
        .`as` { StepVerifier.create(it) }
        .expectNext(savedData)
        .verifyComplete()
}
```

<br/>

전체 Document의 개수를 조회하는 함수를 테스트해보자. 저장한 Document 수와 조회한 카운트가 일치하는지 확인한다.

```kotlin
@Test
fun `count will return count of document correctly`() {
    // Arrange
    generateDocument()
    generateDocument()
    generateDocument()

    // Act & Assert
    repository.count()
        .`as` { StepVerifier.create(it) }
        .expectNext(3)
        .verifyComplete()
}
```

<br/>

delete 테스트를 추가해보자. 데이터를 저장한 후 해당 PK를 이용하여 데이터를 삭제한 다음 검증하는 방식으로 진행하였다.

```kotlin
@Test
fun `deleteById will delete document correctly`() {
    // Arrange
    val document = generateDocument()

    // Act
    repository.deleteById(document.id.orEmpty())
        .`as` { StepVerifier.create(it) }
        .expectNextCount(0)
        .verifyComplete()

    // Assert
    template.findById(document.id.orEmpty(), Person::class.java)
        .`as` { StepVerifier.create(it) }
        .expectNextCount(0)
        .verifyComplete()
}
```

## GIT

해당 튜토리얼에 대한 전체 코드는 Github을 참고하기 바란다. ([https://github.com/veluxer62/toy/tree/spring-data-mongodb-reactive](https://github.com/veluxer62/toy/tree/spring-data-mongodb-reactive){: target="\_blank" })

## Wrap up

**Spring Data MongoDB**는 **Spring Data R2DBC**에 비해 훨씬 안정된 기능을 제공해준다. 이는 아마 Database 자체적으로 비동기 Connection을 제공해 주고 있기 때문에 오래전부터 Reactive Repository를 제공해주고 있지 않나 생각된다. Document DB를 이용한 어플리케이션 개발 사례가 늘어나고 있는 만큼 Spring WebFlux와 Spring Data MongoDB를 이용한 개발 사례도 많이 늘었으면 하는 바램이다.
