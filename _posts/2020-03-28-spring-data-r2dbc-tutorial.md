---
layout: single
title: "Spring WebFlux - Spring Data R2DBC tutorial"
categories:
  - TUTORIALS
tags:
  - Spring WebFlux
  - kotlin
  - Spring Data
  - H2
  - R2DBC

toc: true
toc_sticky: true
excerpt: "**Spring WebFlux**에 대해서는 작년에 처음 내용을 접하고 나서 관심이 생겨서 개인 프로젝트로 만들어보기도 하면서 조금씩 익혀가고 있다. 현재는 공식 래퍼런스를 번역해가면서 내용을 제대로 익히기 위해 공부하고 있다. Spring WebFlux가 어떠한 컨셉을 가지고 기능을 제공하는 가는 공식 레퍼런스 문서에서 좀더 자세히 볼 수 있다. 이 글을 시작으로 **Spring WebFlux**에서 사용할 수 있는 기능들을 간단한 튜토리얼을 시리즈 형식으로 작성해가면서 다양한 사용법을 익혀가보고자 한다. (다음편이 작성되면 이전 페이지에 링크를 추가하면서 진행할 예정이다.)"
---

## Introduction

**Spring WebFlux**에 대해서는 작년에 처음 내용을 접하고 나서 관심이 생겨서 [개인 프로젝트](https://github.com/veluxer62/occupying)로 만들어보기도 하면서 조금씩 익혀가고 있다. 현재는 공식 래퍼런스를 번역해가면서 내용을 제대로 익히기 위해 공부하고 있다. Spring WebFlux가 어떠한 컨셉을 가지고 기능을 제공하는 가는 [공식 레퍼런스 문서](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }에서 좀더 자세히 볼 수 있다.

이 글을 시작으로 **Spring WebFlux**에서 사용할 수 있는 기능들을 간단한 튜토리얼을 시리즈 형식으로 작성해가면서 다양한 사용법을 익혀가보고자 한다. (다음편이 작성되면 이전 페이지에 링크를 추가하면서 진행할 예정이다.)

> 해당 튜토리얼은 [Spring boot](https://spring.io/projects/spring-boot){: target="\_blank" }, [Kotlin](https://kotlinlang.org/){: target="\_blank" }, [Gradle Kotlin DSL](https://docs.gradle.org/current/userguide/kotlin_dsl.html){: target="\_blank" }을 이용하여 진행된다.

## Spring Data R2DBC

Spring을 사용한다면 익숙한 Spring Data의 R2DBC 버전이다. JPA와 같이 관계형 데이터베이스를 다루지만 JPA와 다르게 반응형 드라이버를 사용해서 저장소의 사용을 추상화 한다.

[**Spring Data R2DBC**](https://github.com/spring-projects/spring-data-r2dbc){: target="\_blank" } 소개 문서에서 눈에 띄는 부분중 하나는 **Spring Data R2DBC**는 ORM이 아니라는 것이다. 그래서 [**Spring Data JPA**](https://spring.io/projects/spring-data-jpa){: target="\_blank" }에서 제공해주는 다양한 기능을 지원하지 않음을 인지하고 사용해야 한다.

## Getting Start

이제 **Spring Data R2DBC**를 어떻게 사용하는지 알아보자. (프로젝트 생성은 [Spring initializer](https://start.spring.io/){: target="\_blank" })를 이용하였으며 GradleProject, Kotlin, SpringBoot 2.2.6 버전을 설정하여 생성하였다. 데이터베이스는 [H2](https://github.com/r2dbc/r2dbc-h2){: target="\_blank" }를 선택하였다.)

### Gradle

**Spring Data R2DBC**를 사용하기 위해서는 2개의 의존성 설정이 필요하다. 하나는 [Srping Data R2DBC 의존성](https://github.com/spring-projects/spring-data-r2dbc#maven-configuration){: target="\_blank" }이고 다른 하나는 [R2DBC 의존성](https://github.com/r2dbc/r2dbc-h2#maven)이다.

```gradle
implementation("org.springframework.data:spring-data-r2dbc:${version}.RELEASE")
implementation("io.r2dbc:r2dbc-h2:${version}.RELEASE")
```

### Configuration

**Spring Data R2DBC**를 사용하기 위해서 Configuration을 위한 Class를 생성해주자. 그런 다음 해당 클래스가 설정을 위한 클래스인 것을 프레임워크에게 알려주기 위해 `@Configuration` 어노테이션을 클래스에 추가하고 R2DBC 레파지토리를 사용할 것임을 설정하기 위해 `@EnableR2dbcRepositories`를 클래스에 추가하고 `AbstractR2dbcConfiguration` 추상 클래스를 상속 받는다.

작성한 설정 클래스에서 R2DBC를 사용하기 위한 `connctionFactory` 빈을 등록해준다. (여기에서는 H2데이터베이스의 [Embedded Mode](https://h2database.com/html/features.html){: target="\_blank" }를 사용하였다.)
`R2bcConfiguration`클래스에 위에서 생성한 sql을 어플리케이션이 로드 될 때 실행되도록 `initializer` 빈을 등록한다.

```kotlin
@Configuration
@EnableR2dbcRepositories
class R2bcConfiguration : AbstractR2dbcConfiguration() {
  @Bean
  override fun connectionFactory() = H2ConnectionFactory(
      H2ConnectionConfiguration.builder()
          .inMemory("test")
          .property(H2ConnectionOption.DB_CLOSE_DELAY, "-1")
          .build()
  )

  @Bean
  fun initializer(connectionFactory: ConnectionFactory): ConnectionFactoryInitializer {
      val initializer = ConnectionFactoryInitializer()
      initializer.setConnectionFactory(connectionFactory)
      val populator = ResourceDatabasePopulator(ClassPathResource("sql/db-schema.sql"))
      initializer.setDatabasePopulator(populator)
      return initializer
  }
}
```

<br />

**Spring Data R2DBC**에서는 자동으로 DDL을 실행해주지 않는다. 그러므로 우리는 DDL을 생성해주는 코드를 추가해 주어야 하는데 추가하는 방법은 여러가지가 있지만 sql 파일을 생성하여 DDL을 한곳에서 관리하는 것이 가장 나아 보여서 해당 방법을 소개하고자 한다. **Spring Boot**에서는 `resource`라는 디렉토리를 제공해 주는데 이곳에서 필요한 설정이나 말그대로 어플리케이션에서 사용될 자원 정보들을 두는 공간으로 활용된다. 이곳에 `db-schema.sql`을 생성해놓은 후 DDL을 관리하는 방식을 사용하면 좋을 것 같다. 테이블 생성 쿼리는 테이블 존재하는 경우에만 테이블을 생성할 수 있는 옵션인 `IF NOT EXISTS`을 추가하였다. 해당 옵션이 없으면 어플리케이션이 최초 실행 이후에는 DDL이 오류가 발생해서 정상 동작하지 않는다.

```sql
CREATE TABLE IF NOT EXISTS PERSON (
	ID BIGINT AUTO_INCREMENT,
	FIRSTNAME VARCHAR(255) NOT NULL,
	LASTNAME VARCHAR(255) NOT NULL,
	CONSTRAINT PERSON_PK PRIMARY KEY (ID)
);
```

### Persistence Model

**Spring Data R2DBC**는 ORM이 아니다. 그래서 **Spring Data JPA**와 같이 `@Entity` 어노테이션을 이용한 Persistence Model 정의를 사용하지 않는다. 그래서 단순히 데이터 객체만 생성하면된다. 다만 Primary Key를 지정해주기 위해서 `@Id` 어노테이션은 사용한다.

```kotlin
data class Person(
    @Id val id: Long?,
    val firstname: String,
    val lastname: String
)
```

### Repository

다른 **Spring Data**와 비슷하게 **Spring Data R2DBC**도 인터페이스 정의만으로 레파지토리를 추상화 하고 있다. `ReactiveCrudRepository`인터페이스를 구현해서 새로운 인터페이스를 정의하면 된다.

```kotlin
interface PersonRepository : ReactiveCrudRepository<Person, Long>
```

### Testing

사실 **Spring Data R2DBC**의 레파지토리에서 제공하는 기본 함수에 대한 테스트는 검증이 이미 된 함수이기 때문에 테스트 코드가 필요하지 않다. 하지만 우리는 **Practice Testing**을 통해 **Spring Data R2DBC**에서 제공하는 기능을 익힐 수 있으므로 테스트 코드를 작성해 보면 좋을 것 같다. (**Practice Testing**에 대한 장점은 [여기 글](https://www.ernweb.com/educational-research-articles/learning-techniques-effective-study/){: target="\_blank" }을 참고하자.)

먼저 테스트 클래스를 생성하자. Spring Boot 2.2 버전에서는 테스트용 도구로 [junit5](https://junit.org/junit5/docs/current/user-guide/){: target="\_blank" }를 사용하고 있다.
테스트 클래스임을 설정하기 위해 `@ExtendWith(SpringExtension::class)` 주석을 추가한다. 다음으로 테스트할 레파지토리 빈을 생성하기 위해 `@ContextConfiguration(classes = [R2bcConfiguration::class])` 주석을 추가한다. 해당을 주석을 추가함으로써 `PersonRepository`와 `DatabaseClient`를 사용할 수 있게 되었다. `PersonRepository`는 실제로 테스트할 [SUT](https://en.wikipedia.org/wiki/System_under_test){: target="\_blank" } 객체 이고, `DatabaseClient`는 데이터를 검증하기 위해 사용될 객체이다.

```kotlin
@ExtendWith(SpringExtension::class)
@ContextConfiguration(classes = [R2bcConfiguration::class])
internal class PersonRepositoryTest(
    @Autowired private val personRepository: PersonRepository,
    @Autowired private val databaseClient: DatabaseClient
) {

}
```

<br/>

각 테스트 별로 데이터에 따른 영향을 제거하기 위해 매 테스트 함수 실행 전에 데이터를 모두 지우는 함수를 추가한다.

```kotlin
@BeforeEach
fun beforeEach() {
    databaseClient
        .execute("delete from person")
        .fetch()
        .rowsUpdated()
        .block()
}
```

<br/>

테스트를 도와주기 위한 함수를 먼저 추가한다. 해당 함수들은 Persistence Model를 생성하는 함수와 데이터 insert 쿼리를 실행하는 함수이다.

```kotlin
private fun insertPerson(id: Long? = null): Person {
    val givenPerson = generatePerson(id)
    databaseClient.insert().into(Person::class.java)
        .using(givenPerson)
        .fetch()
        .rowsUpdated()
        .block()
    return givenPerson
}

private fun generatePerson(id: Long? = null): Person {
    return Person(
        id = id,
        firstname = UUID.randomUUID().toString(),
        lastname = UUID.randomUUID().toString()
    )
}
```

<br/>

insert 테스트를 추가해보자. repository의 `save`를 실행 한 후 다음 엘리먼트가 실행되는지 검증한다. 실제로 데이터가 저장되었는지 검증하기 위해 count SQL 쿼리를 실행하여 확인하는 코드도 추가였다.

```kotlin
@Test
fun `test save`() {
    val person = generatePerson()
    personRepository.save(person)
        .`as` { StepVerifier.create(it) }
        .expectNextCount(1)
        .verifyComplete()

    val selectOne = databaseClient
        .execute(
            """
                select count(*) from person
                where firstname='${person.firstname}' and lastname='${person.lastname}'
            """.trimIndent()
        )
        .asType<Long>()
        .fetch()
        .one()

    StepVerifier
        .create(selectOne)
        .expectNext(1)
        .verifyComplete()
}
```

<br/>

update 테스트를 추가해보자. 데이터를 먼저 저장한 후 동일한 PK를 가진 다른 데이터를 저장하는 방식으로 진행하였다. 검증방식은 **insert 테스트**와 유사하다.

```kotlin
@Test
fun `test update`() {
    val givenPerson = insertPerson(Random.nextLong())

    val person = givenPerson.copy(
        firstname = UUID.randomUUID().toString(),
        lastname = UUID.randomUUID().toString()
    )
    personRepository.save(person)
        .`as` { StepVerifier.create(it) }
        .expectNextCount(1)
        .verifyComplete()

    val expected = databaseClient
        .execute("select * from person where id = ${person.id}")
        .asType<Person>()
        .fetch()
        .one()

    StepVerifier
        .create(expected)
        .expectNext(person)
        .verifyComplete()
}
```

<br/>

select 테스트를 추가해보자. 데이터를 먼저 저장한 후 저장한 데이터 개수를 검증하는 방식으로 진행하였다.

```kotlin
@Test
fun `test findAll`() {
    insertPerson()
    insertPerson()
    insertPerson()

    personRepository.findAll()
        .`as` { StepVerifier.create(it) }
        .expectNextCount(3)
        .verifyComplete()
}
```

<br/>

delete 테스트를 추가해보자. 데이터를 저장한 후 해당 PK를 이용하여 데이터를 삭제한 다음 검증하는 방식으로 진행하였다.

```kotlin
@Test
fun `test deleteById`() {
    val id = Random.nextLong()
    insertPerson(id)

    personRepository.deleteById(id)
        .`as` { StepVerifier.create(it) }
        .expectNextCount(0)
        .verifyComplete()

    val expected = databaseClient
        .execute("select count(*) from person where id = $id")
        .asType<Long>()
        .fetch()
        .all()

    StepVerifier
        .create(expected)
        .expectNext(0)
        .verifyComplete()
}
```

### Query Method

**Spring Data**에서 제공하는 주요 기능중 하나인 [Query Method](https://docs.spring.io/spring-data/data-commons/docs/current/reference/html/#repositories.query-methods){: target="\_blank" }는 **Spring Data R2DBC**에서는 아직 제대로 지원하지 않고 있다. 대신에 `@Query` 어노테이션을 추가하여 쿼리를 입력해준다면 사용가능하다. (해당 내용은 [문서](https://docs.spring.io/spring-data/r2dbc/docs/1.0.0.RELEASE/reference/html/#r2dbc.repositories.queries){: target="\_blank" } 참고)

```kotlin
@Query("select * from person where firstname = :firstname")
fun findAllByFirstname(firstname: String): Flux<Person>
```

## GIT

해당 튜토리얼에 대한 전체 코드는 Github을 참고하기 바란다. ([https://github.com/veluxer62/toy/tree/r2dbc-repository-with-h2/spring-data-r2bc-example](https://github.com/veluxer62/toy/tree/r2dbc-repository-with-h2/spring-data-r2bc-example){: target="\_blank" })

## Wrap up

**Spring WebFlux**는 2019년 1월 Spring Framework 5.0 버전때 출시되면서 소개되었다. 비차단, 반응형의 장점은 많은 개발자들이 관심을 가질만 하지만 아직까지 Spring WebFlux를 현업에서 사용하기 꺼져지는 가장 큰 이유는 JDBC가 비동기 관계형 데이터베이스 라이브러리가 아니기 때문일 것이다.
스프링 팀에서도 이러한 니즈를 인지하고 비동기 관계형 데이터베이스 라이브러리를 만들기 위한 노력을 이어가고 있으며 이러한 노력으로 2019년 12월 **Spring Data R2DBC** 1.0 버전이 정식으로 릴리스 되었다.

다만 아직 [R2DBC](https://r2dbc.io/){: target="\_blank" } 라이브러리들이 메이저 버전이 1인 릴리스 버전이 출시되어 있지 않은 상태(2020년 3월 기준)라는 점과 **Query Method**의 미 지원 등을 이유로 실무에 적용은 아직 시기 상조라 생각된다.
하지만 **R2DBC**의 메이저 1버전이 정식으로 출시되면 한번 써보는 것도 나쁘지 않아 보이긴 하다.

> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }
>
> [https://r2dbc.io/](https://r2dbc.io/){: target="\_blank" }
>
> [https://github.com/spring-projects/spring-data-r2dbc](https://github.com/spring-projects/spring-data-r2dbc){: target="\_blank" }
>
> [https://github.com/r2dbc/r2dbc-h2](https://github.com/r2dbc/r2dbc-h2){: target="\_blank" }
>
> [https://h2database.com/html/features.html](https://h2database.com/html/features.html){: target="\_blank" }
>
> [https://docs.spring.io/spring-data/data-commons/docs/current/reference/html/#repositories.query-methods](https://docs.spring.io/spring-data/data-commons/docs/current/reference/html/#repositories.query-methods){: target="\_blank" }
