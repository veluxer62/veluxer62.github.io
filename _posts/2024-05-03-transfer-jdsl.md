---
layout: single
title: "Querydsl에서 Kotlin JDSL 으로"
categories:
  - EXPLANATION
tags:
  - kotlin
  - querydsl
  - kotlin jdsl
toc: true
toc_sticky: true
toc_h_max: 3
excerpt: "우리가 이렇게 유명한 라이브러리인 Querydsl을 Kotlin JDSL로 변경하게 된 계기는 무엇이며 변경하면서 우리가 겪었던 이슈나 전환 시 우리가 사용했던 여러 가지 팁들을 공유하면서 전환 이야기를 나눠보도록 하겠다."
---

## Intro

이 글은 사내 블로그에 작성한 [Querydsl에서 Kotlin JDSL 으로](https://spoqa.github.io/2024/05/03/transfer-jdsl.html){: target="\_blank" } 내용을 그대로 가져오면서 나의 블로그의 언어톤에 맞게 변경한 글이다.

최근 우리 백엔드팀에서는 [Querydsl](https://github.com/querydsl/querydsl){: target="\_blank" }을 [Kotlin JDSL](https://github.com/line/kotlin-jdsl){: target="\_blank" }로 전환하는 작업을 진행하였다. Querydsl은 Spring Framework를 사용하고 있다면 누구나 알고 있는 유명한 쿼리 빌더 라이브러리이다. 한편 Kotlin JDSL은 2021년 라인에서 공개한 쿼리빌더 라이브러리이다.

우리가 이렇게 유명한 라이브러리인 Querydsl을 Kotlin JDSL로 변경하게 된 계기는 무엇이며 변경하면서 우리가 겪었던 이슈나 전환 시 우리가 사용했던 여러 가지 팁들을 공유하면서 전환 이야기를 나눠보도록 하겠다.

## 전환 배경

### kapt를 요하는 Querydsl

Kotlin에서 Querydsl을 사용하기 위해서는 kapt가 필요하다. kapt는 Kotlin의 Annotation Processor를 활용하기 위한 도구로, 주로 Kotlin에서 작성된 코드의 Annotation을 처리하는 데 사용된다. 그러나 공식 문서에 따르면 kapt의 새로운 기능 지원이 중단되고 유지보수만 진행될 예정이라고 한다. 앞으로는 Kotlin에 더 친화적이고 빌드 시간이 더 빠른 KSP를 권장할 것으로 예상된다. 이는 결국 kapt가 향후 Kotlin에서 지원되지 않을 가능성이 높다는 것을 의미한다.

![kapt-warning](/assets/images/posts/transfer-jdsl/kapt-warning.png)

Kotlin에서 Querydsl를 사용하려면 Gradle에서 의존성을 추가할 때 다음과 같이 kapt가 필요하다.

```kotlin
// build.gradle.kts

plugins {
    kotlin("kapt") version "1.9.21"
}

dependencies {
    implementation("com.Querydsl:Querydsl-jpa:5.1.0:jakarta")
    kapt("com.Querydsl:Querydsl-apt:5.1.0:jakarta")
}
```

kapt를 사용하지 않는다면 KSP를 대안으로 고려할 수 있다. 그러나 [[Kotlin] Migrate kapt to ksp](https://github.com/Querydsl/Querydsl/issues/3284){: target="\_blank" } 이슈에서 보면 Querydsl에서 KSP를 지원할지 여부는 아직 미지수인듯 하다.

### Querydsl 유지보수

Querydsl의 [릴리스 노트](https://github.com/Querydsl/Querydsl/releases){: target="\_blank" }를 살펴보면 배포 주기가 상당히 길다는 사실을 확인할 수 있다. 2018년에는 2번, 2019년에는 1번, 2020년에는 2번, 2021년에는 두 번, 그리고 2024년에는 한 번의 릴리스가 있었다. 이렇듯 이슈는 꾸준히 제기되고 있지만, 실제로 적극적인 패치나 새로운 기능의 개발은 이루어지고 있지 않은 실정이다.

오픈소스를 선택할 때 때 큰 이유 중 하나는 이슈에 대한 적극적인 패치와 업데이트로 인한 신뢰성과 안정성을 기대하기 때문이다. 그러나 이와 같은 상황은 Querydsl을 사용하는 경우 이슈에 대한 신속한 대응과 새로운 기능 개발을 기대하기 어렵다는 것을 보여준다. 따라서 우리는 Querydsl이 더 이상 오픈소스로서 안정성을 유지하는 데 어려움이 있다고 판단했다.

## 왜 Kotlin JDSL인가?

### 쿼리 빌더 선택 비교

Kotlin 생태계에서는 Querydsl 외에도 여러 선택지가 있다. 아래는 Kotlin에서 사용할 수 있는 다양한 쿼리 빌더들 중에서 우리가 검토해 본 것들이다.

- **Exposed**: Jetbrains사에서 만든 Kotlin 경량 ORM 이다. 직관적이고 간결한 API를 제공한다.
- **Ktorm**: Kotlin을 위한 경량 ORM 라이브러리로, DSL을 통한 효율적인 쿼리 작성이 특징이다.
- **JOOQ**: Java의 또 다른 유명한 쿼리 빌더이다. Kotlin에서도 사용할 수 있으며, 타입 안전성을 강조하고 SQL을 자동으로 생성한다.
- **JPA Criteria API**: 영속화를 위한 쿼리언어인 JPQL을 자바 코드로 작성할 수 있도록 지원하는 쿼리 빌더이다. JPA를 사용한다면 추가로 의존성을 추가해 줄 필요 없이 사용할 수 있다.
- **Query Method**: Spring Data JPA의 Query Methods는 함수명을 분석하여 자동으로 SQL 쿼리를 생성해 주는 기능을 제공한다.
- **Kotlin JDSL**: Kotlin으로 메타모델 없이 작성할 수 있는 쿼리 빌더이다. JPQL을 기반으로 쿼리가 생성된다.

#### Exposed, Ktorm

[Exposed](https://github.com/JetBrains/Exposed){: target="\_blank" }와 [Ktorm](https://github.com/kotlin-orm/ktorm){: target="\_blank" }은 둘 다 ORM 라이브러리이다. 우리는 이미 JPA를 ORM으로 사용하고 있으며, JPA의 엔티티로 반환되는 쿼리 결과 및 이를 활용한 Dirty Checking, Lazy Loading, Cascade 등의 비즈니스 로직이 많이 존재한다. 따라서 Exposed와 Ktorm과 같은 ORM을 대체하는 옵션은 고려되지 않았다.

#### JOOQ

[JOOQ](https://github.com/jOOQ/jOOQ){: target="\_blank" }는 Java 생태계에서 새롭게 주목받는 유명한 쿼리 빌더이다. 타입 안정성을 제공하여 MyBatis와는 달리 뛰어난 유지보수성을 가지고 있으며, ORM인 Hibernate와 비교하여 SQL 친화적이어서 최적화된 쿼리와 세밀한 쿼리 작성이 가능하다.

우리는 [JOOQ가 JPA와 함께 사용할 수 있다는 점](https://www.jooq.org/doc/latest/manual/sql-execution/alternative-execution-models/using-jooq-with-jpa/){: target="\_blank" }에 주목하여 POC를 진행했다. 그러나 멀티 모듈 설정이나 메타모델 생성을 위한 복잡한 메커니즘 등으로 인해 JPA와 함께 사용하는 것에 대한 거부감을 느꼈다. 또한 JOOQ에서 권장하는 방식과 다르기 때문에 JPA와 함께 사용할 때 발생할 수 있는 이슈에 대한 공식적인 지원을 받기 어려울 것이라는 우려도 있었다.

이러한 이유로 JOOQ도 전환 대상에서 제외하기로 했다.

![jooq-with-jpa](/assets/images/posts/transfer-jdsl/jooq-with-jpa.png)

#### Spring Data의 Query Method

Spring Data에서 제공해 주는 [Query Method](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html){: target="\_blank" }는 아래와 같이 쿼리빌더를 통해 쿼리를 세세하게 작성하지 않아도 원하는 쿼리를 실행시켜 준다는 장점이 있다.

```kotlin
interface AdminRepository : JpaRepository<Admin, UUID> {
    fun findByEmail(email: String): Optional<Admin>
}
```

만약 복잡한 쿼리를 생성해야 하는 경우에는 아래와 같이 `@Query` 어노테이션을 활용하여 작성하기도 한다.

```kotlin
interface AdminRepository : JpaRepository<Admin, UUID> {
    @Query(value = "SELECT a FROM Admin a JOIN AdminMeta b on a.id = b.adminId WHERE b.isActive is true ")
    fun findActive(): List<Admin>
}
```

Query Method를 활용하면 추가된 라이브러리 없이 간편하게 쿼리를 작성할 수 있지만, 작성된 쿼리가 컴파일 시점에서 정상 작동하는지 확인하기 어렵다는 점이 있다. 이는 테스트 코드를 통해 보완할 수 있지만, 추가로 매개변수에 따라 동적으로 쿼리를 작성하는 기능을 활용하기 어려운 단점도 있기에 Spring Data의 Query Method사용은 Querydsl의 대체제로는 제외되었다.

#### JPA의 Criteria API

JPA의 [Criteria API](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#criteria){: target="\_blank" }를 사용하면 Querydsl과 같이 쿼리를 코드로 생성할 수 있다.

```kotlin
fun findPrivateStores(): List<Store> {
    val cb = entityManager.criteriaBuilder
    return cb
        .createQuery(Store::class.java)
        .apply {
            val store = from(Store::class.java)

            select(store)
                .where(cb.equal(store.get<StoreType>("storeType"), StoreType.private))
        }
        .let { entityManager.createQuery(it).resultList }
}
```

또한, Spring Data에서 지원하는 [Specifications](https://docs.spring.io/spring-data/jpa/reference/jpa/specifications.html){: target="\_blank" }를 통해서 동적 쿼리도 작성할 수 있다.

```kotlin
interface StoreRepository : JpaRepository<Store, UUID>, JpaSpecificationExecutor<Store>

data class StoreSpecification(
    val createdAtGte: OffsetDateTime?,
    val createdAtLte: OffsetDateTime?,
) : Specification<Store> {
    override fun toPredicate(root: Root<Store>, query: CriteriaQuery<*>, builder: CriteriaBuilder): Predicate {
        val predicates = mutableListOf<Predicate>()

        createdAtGte?.let {
            predicates.add(builder.greaterThanOrEqualTo(root.get("createdAt"), it))
        }

        createdAtLte?.let {
            predicates.add(builder.lessThanOrEqualTo(root.get("createdAt"), it))
        }

        return builder.and(*predicates.toTypedArray())
    }

}

test("test specification") {
    storeRepository.findAll(
        StoreSpecification(
            createdAtGte = null,
            createdAtLte = null,
        )
    )
    // select * from store s1_0 where 1=1;

    storeRepository.findAll(
        StoreSpecification(
            createdAtGte = OffsetDateTime.now(),
            createdAtLte = null,
        )
    )
    // select * from store s1_0 where s1_0.created_at>='2021-01-01T01:00:00.000+0900';

    storeRepository.findAll(
        StoreSpecification(
            createdAtGte = null,
            createdAtLte = OffsetDateTime.now(),
        )
    )
    // select * from store s1_0 where s1_0.created_at<='2024-04-26T23:04:53.156+0900';

    storeRepository.findAll(
        StoreSpecification(
            createdAtGte = OffsetDateTime.now(),
            createdAtLte = OffsetDateTime.now(),
        )
    )
    // select * from store s1_0 where s1_0.created_at>='2024-04-26T23:04:53.162+0900' and s1_0.created_at<='2024-04-26T23:04:53.162+0900';
}
```

코드를 살펴보면 조건문에서 `s.get<StoreType>("storeType")`과 같이 문자열 매개변수를 사용하는 것을 알 수 있다. 이는 타입 안정성이 다소 떨어지기 때문에 유지보수에 대한 고민이 있었다. 이러한 문제를 해결하기 위해 `s.get<StoreType>(Store::storeType.name)`과 같이 Entity의 Property로 정의하거나 Hibernate의 [메타모델](https://docs.jboss.org/hibernate/jpamodelgen/1.0/reference/en-US/html_single/#whatisit){: target="\_blank" }을 활용하여 타입 안정성을 확보할 수 있다. 그러나 여전히 매개변수로 문자열을 사용할 수 있기 때문에 팀 내에서 엄격한 코딩 규칙을 정하고 사용을 제한하는 방법으로 타입 안정성을 보장할 수 있을 것으로 생각된다.

이렇듯 JPA의 Criteria API는 타입 안정성을 유지하면서 쿼리를 작성하고, Specifications를 통해 동적 쿼리를 지원하기 때문에 Kotlin JDSL을 발견하기 전까지 가장 유력한 전환 후보였다.

#### Kotlin JDSL

라인의 개발팀에서 개발한 [Kotlin JDSL](https://github.com/line/kotlin-jdsl){: target="\_blank" }은 Querydsl이나 JOOQ와는 다르게 메타모델을 필요로 하지 않고 쿼리를 쉽게 작성할 수 있는 라이브러리이다. DSL을 이용하여 타입 안정성 있는 쿼리를 작성할 수 있으며, 동적 쿼리를 간편하게 생성할 수 있는 장점을 갖고 있다. 또한 JPQL을 기반으로 동작하기 때문에 JPA와의 호환성도 우수하다.

```kotlin
interface StoreRepository : JpaRepository<Store, UUID>, KotlinJdslJpqlExecutor

val namePredicate = "JDSL 매장"

storeRepository.findAll {
    select(entity(Store::class))
        .from(entity(Store::class))
        .where(
            path(Store::name).like("%$namePredicate%")
        )
}
```

비록 추가적인 의존성을 필요로 하기는 하지만 DSL을 활용하여 Criteria API에 비해 간결하고 직관적인 쿼리를 작성할 수 있고 타입 안정성을 기본적으로 지원하기에 우리는 최종적으로 Kotlin JDSL으로 전환하기로 하였다.

### 비교

다시 한번 위에서 말한 쿼리빌더들을 표로 정리하면 아래와 같다.

|           이름           | JPA 연동여부 | 타입 안정성 | 동적 쿼리 | 추가 의존성 |
| :----------------------: | :----------: | :---------: | :-------: | :---------: |
|         Exposed          |      X       |      O      |     O     |      O      |
|          Ktorm           |      X       |      O      |     O     |      O      |
|           JOOQ           |      △       |      O      |     O     |      O      |
| Spring Data Query Method |      O       |      X      |     X     |      X      |
|     JPA Criteria API     |      O       |      △      |     O     |      △      |
|        Kotlin JDSL       |      △       |      O      |     O     |      O      |

## 전환 방법

Querydsl에서 Kotlin JDSL로의 전환을 본격적으로 이야기해 보겠다.

여러 제품 개발 프로젝트가 동시에 진행되는 상황에서 Querydsl을 Kotlin JDSL로 전환해야 했다. 이를 위해 효율적이고 리스크를 최소화할 방법을 고민한 결과, 아래와 같은 순서로 작업을 진행했다.

1. **전환 대상 목록화**: 먼저 전환할 대상을 명확히 정리하고 우선순위를 결정했다.
2. **베이스 코드 작성**: 전환할 대상을 위한 환경 설정 및 베이스 코드를 작성하고 팀원들이 손쉽게 적응할 수 있도록 예시 코드를 작성했다.
3. **작업 방식 전파**: 전환된 코드와 작업 방식에 대한 교육과 트레이닝을 진행하여 팀 전체에 새로운 작업 방식을 전파했다.
4. **병렬 작업**: 여러 프로젝트를 수행하면서 동시에 전환 작업을 병렬적으로 진행하여 효율성을 극대화했다.

### 전환 대상 목록화

우리 팀은 Querydsl을 Kotlin JDSL로 전환하는 프로젝트와 같이 규모가 있는 리펙터링 작업인 경우 아래와 같이 작업 목록을 생성하여 진행하였다.

![transfer-epic](/assets/images/posts/transfer-jdsl/transfer-epic.png)

이처럼 작업 목록을 생성하고 리팩토링을 진행하면 다음과 같은 장점을 얻을 수 있다.

- **규모 파악**: 리팩토링을 위한 작업을 미리 Task로 등록하면 해당 프로젝트의 규모와 작업에 들 시간을 예상할 수 있다.
- **영향 범위 파악**: 생성된 작업 목록을 통해 프로젝트의 영향 범위를 손쉽게 파악할 수 있다. 물론 작업 명을 명확하게 작성하는 것이 중요하다.
- **진행률 파악**: 에픽을 통해 프로젝트 진행률을 손쉽게 파악할 수 있다. 소스 코드를 탐색하지 않고도 개발 상황을 파악할 수 있다.
- **병렬 작업**: 필요한 작업을 미리 생성하고 목록화하면 여유가 생길 때 해당 작업을 누구나 진행할 수 있다.

이러한 장점을 활용하기 위해 Kotlin JDSL 전환 작업을 위한 에픽을 생성하고, Querydsl을 사용한 코드를 모두 검토하여 작업 목록을 미리 생성하는 단계부터 시작했다.

### 베이스 코드 작성

베이스 코드는 여러 명의 개발자가 가능한 한 일관된 방식으로 전환할 수 있도록 가이드라인을 제공하는 역할을 한다. 개발자들이 공식 문서를 참고하여 각자가 Querydsl에서 Kotlin JDSL로의 전환을 수행하는 것은 어렵지 않겠지만, 코드의 일관성이 부족하면 전체적인 생산성에 영향을 미칠 수 있다.

따라서 우리는 먼저 아래와 같이 베이스 코드를 작성하여 팀원들이 일관된 방식으로 코드 작업을 수행할 수 있도록 하였다. 이를 통해 병렬로 코드 작업을 진행할 수 있도록 지원하였다.

#### 의존성 추가

전환 작업을 모두가 병렬적으로 수행하기 위해서는 우선 의존성 설정과 환경 설정 등을 위한 기반 코드를 작성해야 한다. 따라서 Kotlin JDSL을 사용하기 위한 의존성을 먼저 추가했다. 또한, 우리는 Spring을 사용하고 있기 때문에 [`spring-data-jpa-support`](https://kotlin-jdsl.gitbook.io/docs/v/ko-1/jpql-with-kotlin-jdsl/spring-supports){: target="\_blank" }도 함께 추가했다.

```kotlin
implementation("com.linecorp.kotlin-jdsl:jpql-dsl:$jdslVersion")
implementation("com.linecorp.kotlin-jdsl:jpql-render:$jdslVersion")
implementation("com.linecorp.kotlin-jdsl:spring-data-jpa-support:$jdslVersion")
```

#### 테스트 코드 설정

다음으로 Repository 테스트에서 Kotlin JDSL을 이용한 쿼리 테스트를 수행하기 위해서 아래와 같이 `KotlinJdslAutoConfiguration`도 추가해 준다.

```kotlin
@DataJpaTest
@Import(
    KotlinJdslAutoConfiguration::class,
)
abstract class RepositoryTestBase
```

#### 예시 전환 코드 작성

우리는 프로젝트 초기부터 Kotlin JDSL을 적용하는 것이 아니라 Querydsl에서 Kotlin JDSL로의 전환을 진행했기 때문에 공식 문서에서 안내하는 방식대로 코드를 작성하는 것이 어려웠다. 이는 해당 방식으로는 Service 코드를 변경해야 했기 때문이다.

```kotlin
interface AdminRepository : JpaRepository<Admin, UUID>, KotlinJdslJpqlExecutor

class AdminService(
    private val bookRepository: BookRepository,
) {
    fun getByFilter(
        filter: SearchFilter<Admin>,
    ): List<Admin> {
        return bookRepository.findAll {
            select(entity(Admin::class))
                .from(entity(Admin::class))
                .whereAnd(
                    path(Admin::email).like("%${filter.email}%"),
                )
        }
        .filterNotNull()
    }
}
```

그래서 우리는 Querydsl에서 활용한 Spring Data가 제공하는 [Custom Repository Implementations](https://docs.spring.io/spring-data/jpa/reference/repositories/custom-implementations.html){: target="\_blank" }을 그대로 활용하기로 했다.

```kotlin
interface AdminRepository : JpaRepository<Admin, UUID>, CustomAdminRepository {
    fun findByEmail(email: String): Optional<Admin>
}

interface CustomAdminRepository {
    fun findByFilter(
        filter: SearchFilter<Admin>
    ): List<Admin>
}

class CustomAdminRepositoryImpl(
    private val kotlinJdslJpqlExecutor: KotlinJdslJpqlExecutor,
) : CustomAdminRepository {
    override fun findByFilter(
        filter: SearchFilter<Admin>
    ): List<Admin> {
        return kotlinJdslJpqlExecutor
            .findAll {
                select(entity(Admin::class))
                    .from(entity(Admin::class))
                    .whereAnd(
                        path(Admin::email).like("%${filter.email}%"),
                    )
            }
            .filterNotNull()
    }
}

class AdminService(
    private val adminRepository: AdminRepository,
) {
    fun getByFilter(
        filter: SearchFilter<Admin>,
    ): List<Admin> {
        return adminRepository.findByFilter(filter)
    }
}
```

해당 방식은 앞서 언급한 방식보다 코드양이 많아 보일 수 있지만, 우리가 개발할 때 추상화된 인터페이스를 사용하는 가장 큰 이유라고 생각한다.

이 방식의 가장 큰 장점은 구현 세부 사항의 변경(쿼리빌더가 Querydsl에서 Kotlin JDSL로 변경)이 비즈니스 계층인 Service의 변경을 일으키지 않는다는 것이다. 따라서 쿼리빌더의 변경으로 인한 비즈니스 로직의 변경에 대해 걱정할 필요가 없다. 이는 코드를 유연하게 변경할 수 있음을 보여준다.

실제로 저희가 Querydsl에서 Kotlin JDSL로 전환하는 동안 CustomRepository 외에 Service 코드를 변경한 커밋은 하나도 없었다. 이에 따라 Repository에서 사용된 쿼리 빌더의 변경에만 집중할 수 있었고, 전환된 쿼리 빌더에서 실행된 쿼리가 기존과 동일하게 동작하는지 확인하는 데 집중할 수 있었다.

### 작업 방식 전파

베이스 코드를 작성한 후, 팀원들에게 전환 스타일에 대한 리뷰를 받고 작업 방식을 전파하면서 적용한 코드를 함께 리뷰하는 시간을 가졌다. 각자가 이해하고 있는 라이브러리에 대한 지식과 코딩 스타일이 다를 수 있기 때문에, 가능한 한 맞추되 어느 정도 이견에 대한 합의가 필요했다.

### 병렬 작업

작업 방식에 대한 기본적인 합의가 이루어지면 각자가 본격적으로 전환 작업을 수행할 수 있다. 이 작업은 제품 개발 프로젝트보다는 우선순위가 낮았기 때문에, 특별한 일정 없이 작업시간이 여유로운 개발자가 자율적으로 작업을 할당받아 진행하는 칸반 방식을 채택했다.

## 이슈

아래는 우리가 Kotlin JDSL로 전환하면서 직면한 몇 가지 이슈를 소개해 보려고 한다.

현재는 이미 해결된 이슈들도 있고 Kotlin JDSL의 이슈가 아니라 Spring Data JPA의 이슈인 것들도 있다. Querydsl과 같이 수많은 사용자층이 있는 라이브러리가 아니기 때문에 구글링이나 GPT를 통한 이슈 해결은 어려워 주로 Github Issue를 통해 이슈를 문의하고 답변을 받아 이슈를 해결했다. 다행인 점은 Kotlin JDSL을 주로 관리하는 라인 측 개발자분께서 이슈를 남기면 대부분 하루 안에 친절한 설명과 함께 답변을 주셔서 큰 문제 없이 전환 작업을 완료할 수 있었다.

Kotlin JDSL을 적용 검토 중인 분들이라면 아래 소개하는 이슈들을 참고하면 좋을 것 같다.

#### Spring Data Custom Repository Implementations 이슈

이슈 링크: [Custom Repository Implementations is not available in spring-data-jpa-support](https://github.com/line/kotlin-jdsl/issues/668){: target="\_blank" }

최초 도입 당시 버전인 `3.3.1` 버전에서 spring-data-jpa-support의존성을 추가한 후 Spring Data JPA의 Custom Repository Implementations를 사용하면 아래와 같이 오류가 발생하는 이슈가 있었다. 해당 이슈는 이슈를 제기한 당일 `3.3.2` 버전에서 핫픽스로 빠르게 조치되어 해결되었다.

```
java.lang.IllegalStateException: Failed to load ApplicationContext for [WebMergedContextConfiguration@662061e3 testClass = com.example.demo.ApplicationTests, locations = [], classes = [com.example.demo.Application], contextInitializerClasses = [], activeProfiles = [], propertySourceDescriptors = [], propertySourceProperties = ["org.springframework.boot.test.context.SpringBootTestContextBootstrapper=true"], contextCustomizers = [org.springframework.boot.test.autoconfigure.actuate.observability.ObservabilityContextCustomizerFactory$DisableObservabilityContextCustomizer@1f, org.springframework.boot.test.autoconfigure.properties.PropertyMappingContextCustomizer@0, org.springframework.boot.test.autoconfigure.web.servlet.WebDriverContextCustomizer@18a136ac, org.springframework.boot.test.context.filter.ExcludeFilterContextCustomizer@421bba99, org.springframework.boot.test.json.DuplicateJsonObjectContextCustomizerFactory$DuplicateJsonObjectContextCustomizer@31bcf236, org.springframework.boot.test.mock.mockito.MockitoContextCustomizer@0, org.springframework.boot.test.web.client.TestRestTemplateContextCustomizer@253d9f73, org.springframework.boot.test.context.SpringBootTestAnnotation@3a387497], resourceBasePath = "src/main/webapp", contextLoader = org.springframework.boot.test.context.SpringBootContextLoader, parent = null]
	at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContext(DefaultCacheAwareContextLoaderDelegate.java:180)
	at org.springframework.test.context.support.DefaultTestContext.getApplicationContext(DefaultTestContext.java:130)
...
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'boardRepository' defined in com.example.demo.BoardRepository defined in @EnableJpaRepositories declared on JpaConfig: Could not create query for public abstract java.util.List com.example.demo.CustomBoardRepository.findAllByFilter(com.example.demo.BoardFilter); Reason: Failed to create query for method public abstract java.util.List com.example.demo.CustomBoardRepository.findAllByFilter(com.example.demo.BoardFilter); No property 'filter' found for type 'Board'
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1786)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:600)
...
Caused by: org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract java.util.List com.example.demo.CustomBoardRepository.findAllByFilter(com.example.demo.BoardFilter); Reason: Failed to create query for method public abstract java.util.List com.example.demo.CustomBoardRepository.findAllByFilter(com.example.demo.BoardFilter); No property 'filter' found for type 'Board'
	at org.springframework.data.repository.query.QueryCreationException.create(QueryCreationException.java:101)
	at org.springframework.data.repository.core.support.QueryExecutorMethodInterceptor.lookupQuery(QueryExecutorMethodInterceptor.java:115)
...
Caused by: java.lang.IllegalArgumentException: Failed to create query for method public abstract java.util.List com.example.demo.CustomBoardRepository.findAllByFilter(com.example.demo.BoardFilter); No property 'filter' found for type 'Board'
	at org.springframework.data.jpa.repository.query.PartTreeJpaQuery.<init>(PartTreeJpaQuery.java:106)
	at org.springframework.data.jpa.repository.query.JpaQueryLookupStrategy$CreateQueryLookupStrategy.resolveQuery(JpaQueryLookupStrategy.java:124)
...
Caused by: org.springframework.data.mapping.PropertyReferenceException: No property 'filter' found for type 'Board'
	at org.springframework.data.mapping.PropertyPath.<init>(PropertyPath.java:90)
	at org.springframework.data.mapping.PropertyPath.create(PropertyPath.java:443)
...
```

#### 데이터베이스 잠금 설정 문의

이슈 링크: [Inquire how to use CRUD method metadata](https://github.com/line/kotlin-jdsl/issues/677){: target="\_blank" }

아래와 같이 Querydsl로 작성된 기존의 코드 중에 동시성 이슈를 방지하고자 데이터베이스의 잠금 설정을 적용한 함수가 있었다.

```kotlin
override fun findAllActive(
    orderableVendorId: UUID,
    storeIds: List<UUID>,
): List<Bill> {
    return jpaQueryFactory
        .selectFrom(bill)
        .where(
            bill.orderableVendor.id.eq(orderableVendorId)
                .and(bill.store.id.`in`(storeIds))
                .and(bill.state.eq(BillState.ACTIVE)),
        )
        .setLockMode(LockModeType.PESSIMISTIC_WRITE)
        .setHint("org.hibernate.cacheable", false)
        .setHint("jakarta.persistence.lock.timeout", 10000)
        .fetch()
}
```

Kotlin JDSL의 공식 문서에는 잠금과 관련한 내용이 언급되어 있지 않아 어떻게 사용하면 좋을지 문의를 남겼었다. 그결과 아래와 같이 EntityManager를 이용하여 적용하는 방법을 제안받았다.

```kotlin
class CustomBillRepositoryImpl(
    private val kotlinJdslJpqlExecutor: KotlinJdslJpqlExecutor,
    private val entityManager: EntityManager,
    private val jpqlRenderContext: RenderContext,
) : CustomBillRepository {
    override fun findAllActive(
        orderableVendorId: UUID,
        storeIds: List<UUID>,
    ): List<Bill> {
        val query = jpql {
            select(entity(Bill::class))
                .from(entity(Bill:class))
                .whereAnd(
                    path(Bill::orderableVendor)(OrderableVendor::getId).eq(orderableVendorId),
                    path(Bill::store)(Store::getId).customIn(storeIds),
                    path(Bill::state).eq(BillState.ACTIVE),
                )
        }
        
        return entityManager
            .createQuery(query, jpqlRenderContext)
            .apply { 
                lockMode = LockModeType.PESSIMISTIC_WRITE
                setHint(HINT_CACHEABLE, false)
                setHint(HINT_SPEC_LOCK_TIMEOUT, 10000)
            }
            .resultList
    }
}
```

#### ManyToOne fetch join 이슈

이슈 링크: [Fetch join issue in pagination](https://github.com/line/kotlin-jdsl/issues/682){: target="\_blank" }

`3.3.2` 버전에서 아래와 같이 1:N 관계의 Entity가 존재할 때 페이징 쿼리에서 `ManyToOne` 연관관계를 조회할 때 오류가 발생하는 이슈가 있었다.

```kotlin
@Entity
class Actor(
    @Column(nullable = false)
    val name: String,
) {
    @Id
    val id: UUID = UUID.randomUUID()
}

@Entity
class Board(
    @Column(nullable = false)
    val title: String,

    @Column(nullable = false)
    val content: String,

    @ManyToOne(fetch = FetchType.LAZY)
    val actor: Actor,
) {
    @Id
    val id: UUID = UUID.randomUUID()
}

interface BoardRepository : JpaRepository<Board, UUID>, CustomBoardRepository

interface CustomBoardRepository {
    fun findByActorName(actorName: String, pageable: Pageable): Page<Board?>
}

class CustomBoardRepositoryImpl(
    private val kotlinJdslJpqlExecutor: KotlinJdslJpqlExecutor,
) : CustomBoardRepository {
    override fun findByActorName(actorName: String, pageable: Pageable): Page<Board?> {
        return kotlinJdslJpqlExecutor.findPage(pageable) {
            select(entity(Board::class))
                .from(
                    entity(Board::class),
                    fetchJoin(Board::actor),
                )
                .where(path(Board::actor)(Actor::name).eq(actorName))
        }
    }
}
```

```
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.query.SemanticException: Query specified join fetching, but the owner of the fetched association was not present in the select list [SqmSingularJoin(com.example.demo.Board(Board).actor(Actor) : actor)]
	at org.springframework.orm.jpa.EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible(EntityManagerFactoryUtils.java:371)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:246)
...
Caused by: java.lang.IllegalArgumentException: org.hibernate.query.SemanticException: Query specified join fetching, but the owner of the fetched association was not present in the select list [SqmSingularJoin(com.example.demo.Board(Board).actor(Actor) : actor)]
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:143)
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:167)
...
Caused by: org.hibernate.query.SemanticException: Query specified join fetching, but the owner of the fetched association was not present in the select list [SqmSingularJoin(com.example.demo.Board(Board).actor(Actor) : actor)]
	at org.hibernate.query.sqm.tree.select.SqmQuerySpec.assertFetchOwner(SqmQuerySpec.java:573)
	at org.hibernate.query.sqm.tree.select.SqmQuerySpec.validateFetchOwners(SqmQuerySpec.java:548)
...
```

해당 이슈의 원인은 Kotlin JDSL에서 사용되는 Legacy Query Parser의 페이지네이션과 Fetch Join 처리 메커니즘으로 인한 것으로 확인되었다. 이 이슈는 `3.4.0` 버전에서 해결되었다.

#### selectDistinctNew 이슈

이슈 링크: [When use selectDistinctNew function in findPage, there are some paging error](https://github.com/line/kotlin-jdsl/issues/694){: target="\_blank" }

해당 이슈는 `selectDistinctNew`를 사용하여 페이징 쿼리를 호출하면 일부 페이지에서 오류가 발생하는 이슈이다.

```kotlin
@Entity
class Board(
    @Column(nullable = false)
    val title: String,

    @Column(nullable = false)
    val content: String,
) {
    @Id
    val id: UUID = UUID.randomUUID()
}

@Entity
class BoardStats(
    @Column(nullable = false)
    val boardId: UUID,

    @Column
    val viewCount: Int,
) {
    @Id
    val id: UUID = UUID.randomUUID()
}

interface BoardRepository : JpaRepository<Board, UUID>, KotlinJdslJpqlExecutor

@Test
fun test_findPage() {
    val pageable = PageRequest.of(5, 10)

    val actual = boardRepository.findPage(pageable) {
        selectDistinctNew<Pair<Board, Int?>>(
            entity(Board::class),
            path(BoardStats::viewCount),
        )
            .from(
                entity(Board::class),
                leftJoin(BoardStats::class)
                    .on(path(Board::id).eq(path(BoardStats::boardId)),)
            )
    }
    
    assertEquals(5, actual.totalElements)
}
```

```
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.query.SyntaxException: At 1:26 and token 'kotlin', no viable alternative at input 'SELECT count(DISTINCT NEW *kotlin.Pair(Board, BoardStats.viewCount)) FROM Board AS Board LEFT JOIN BoardStats AS BoardStats ON Board.id = BoardStats.boardId' [SELECT count(DISTINCT NEW kotlin.Pair(Board, BoardStats.viewCount)) FROM Board AS Board LEFT JOIN BoardStats AS BoardStats ON Board.id = BoardStats.boardId]
	at org.springframework.orm.jpa.EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible(EntityManagerFactoryUtils.java:371)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:246)
...
Caused by: java.lang.IllegalArgumentException: org.hibernate.query.SyntaxException: At 1:26 and token 'kotlin', no viable alternative at input 'SELECT count(DISTINCT NEW *kotlin.Pair(Board, BoardStats.viewCount)) FROM Board AS Board LEFT JOIN BoardStats AS BoardStats ON Board.id = BoardStats.boardId' [SELECT count(DISTINCT NEW kotlin.Pair(Board, BoardStats.viewCount)) FROM Board AS Board LEFT JOIN BoardStats AS BoardStats ON Board.id = BoardStats.boardId]
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:143)
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:167)
...
Caused by: org.hibernate.query.SyntaxException: At 1:26 and token 'kotlin', no viable alternative at input 'SELECT count(DISTINCT NEW *kotlin.Pair(Board, BoardStats.viewCount)) FROM Board AS Board LEFT JOIN BoardStats AS BoardStats ON Board.id = BoardStats.boardId' [SELECT count(DISTINCT NEW kotlin.Pair(Board, BoardStats.viewCount)) FROM Board AS Board LEFT JOIN BoardStats AS BoardStats ON Board.id = BoardStats.boardId]
	at org.hibernate.query.hql.internal.StandardHqlTranslator$1.syntaxError(StandardHqlTranslator.java:108)
	at org.antlr.v4.runtime.ProxyErrorListener.syntaxError(ProxyErrorListener.java:41)
...
```

해당 이슈에 대해 Kotlin JDSL 측에 문의해 보았으나, JDSL은 해당 기능을 사용할 때 Spring Data JPA API를 활용하기 때문에 Spring Data JPA 측에 문의하는 것이 좋을 것으로 답변을 받았다.

따라서 Spring Data JPA 측에 이슈를 문의하기 전에, 혹시나 이전에 비슷한 이슈가 있었는지 확인하기 위해 검색을 진행했다. 그 결과 아래 링크에서 관련된 이슈가 이미 제기되어 있는 것을 발견하였다.

- [Cannot compile SELECT with DISTINCT with pagination](https://github.com/spring-projects/spring-data-jpa/issues/3098){: target="\_blank" }

이슈를 요약하면, 기존에 동작하던 Count Query가 잘못된 방식으로 동작하고 있었으며, Distinct와 Count 쿼리를 함께 사용하면 의도치 않게 동작할 수 있으니 다른 방법으로 사용할 것을 권장한다는 내용이다.

따라서 우리도 쿼리를 다르게 작성하여 `selectDistinctNew`를 사용하여 페이지네이션을 하지 않는 방향으로 이 문제를 해결했다.

#### limit 이슈

이슈 링크: [An issue where an error occurs when using limit with a custom serializer in JDSL](https://github.com/line/kotlin-jdsl/issues/695){: target="\_blank" }

기존에 Querydsl을 사용하면서 특정 개수의 데이터만 조회하고자 할 때 주로 `limit` 함수를 활용했다.

```kotlin
override fun findRecentOrderSheetsByStoreId(
    storeId: UUID,
    count: Int,
): List<OrderSheet> {
    return from(orderSheet)
        .where(
            orderSheet.storeId.eq(storeId),
        )
        .orderBy(orderSheet.updatedAt.desc())
        .limit(count.toLong())
        .fetch()
}
```

그래서 Kotlin JDSL로 전환하는 과정에서도 `limit` 함수를 사용해 보려 했으나, 아쉽게도 해당 함수가 존재하지 않았다. 따라서 아래와 같이 Custom DSL을 만들어 사용하게 되었다.

```kotlin
data class JpqlLimit<T : Any>(
    val selectQuery: QueryPart,
    val limit: Long,
    override val returnType: KClass<T>,
) : SelectQuery<T>

object JpqlLimitSerializer : JpqlSerializer<JpqlLimit<*>> {
    override fun handledType(): KClass<JpqlLimit<*>> {
        return JpqlLimit::class
    }

    override fun serialize(
        part: JpqlLimit<*>,
        writer: JpqlWriter,
        context: RenderContext,
    ) {
        val delegate = context.getValue(JpqlRenderSerializer)

        delegate.serialize(part.selectQuery, writer, context)

        writer.write(" LIMIT ${part.limit}")
    }
}

class CustomJpql : Jpql() {
    companion object Constructor : JpqlDsl.Constructor<CustomJpql> {
        override fun newInstance(): CustomJpql = CustomJpql()
    }

    inline fun <reified T : Any> JpqlQueryable<SelectQuery<T>>.limit(limit: Int): JpqlLimit<T> {
        return JpqlLimit(
            this.toQuery(),
            limit.toLong(),
            T::class,
        )
    }
}
```

```kotlin
override fun findRecentOrderSheetsByStoreId(
    storeId: UUID,
    count: Int,
): List<OrderSheet> {
    return kotlinJdslJpqlExecutor
        .findAll {
            select(entity(OrderSheet::class))
                .from(entity(OrderSheet::class))
                .where(path(OrderSheet::storeId).eq(storeId))
                .orderBy(path(OrderSheet::updatedAt).desc())
                .limit(count)
        }
        .filterNotNull()
}
```

하지만 위와 같이 `orderBy` 절을 함께 사용하는 경우 문제가 되지 않았지만, `orderBy` 절 없이 `limit` 함수를 사용하면 아래와 같이 오류가 발생하는 이슈가 있었다.

```kotlin
override fun findRecentOrderSheetsByStoreId(
    storeId: UUID,
    count: Int,
): List<OrderSheet> {
    return kotlinJdslJpqlExecutor
        .findAll {
            select(entity(OrderSheet::class))
                .from(entity(OrderSheet::class))
                .where(path(OrderSheet::storeId).eq(storeId))
                .limit(count)
        }
        .filterNotNull()
}
```

```
org.hibernate.query.SyntaxException: At 1:33 and token 'LIMIT', mismatched input 'LIMIT', expecting one of the following tokens: <EOF>, ',', CROSS, FULL, GROUP, INNER, JOIN, LEFT, ORDER, OUTER, RIGHT, WHERE [SELECT Actor FROM Actor AS Actor LIMIT 1]
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.query.SyntaxException: At 1:33 and token 'LIMIT', mismatched input 'LIMIT', expecting one of the following tokens: <EOF>, ',', CROSS, FULL, GROUP, INNER, JOIN, LEFT, ORDER, OUTER, RIGHT, WHERE [SELECT Actor FROM Actor AS Actor LIMIT 1]
	at org.springframework.orm.jpa.EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible(EntityManagerFactoryUtils.java:371)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:246)
...
Caused by: java.lang.IllegalArgumentException: org.hibernate.query.SyntaxException: At 1:33 and token 'LIMIT', mismatched input 'LIMIT', expecting one of the following tokens: <EOF>, ',', CROSS, FULL, GROUP, INNER, JOIN, LEFT, ORDER, OUTER, RIGHT, WHERE [SELECT Actor FROM Actor AS Actor LIMIT 1]
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:143)
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:167)
...
Caused by: org.hibernate.query.SyntaxException: At 1:33 and token 'LIMIT', mismatched input 'LIMIT', expecting one of the following tokens: <EOF>, ',', CROSS, FULL, GROUP, INNER, JOIN, LEFT, ORDER, OUTER, RIGHT, WHERE [SELECT Actor FROM Actor AS Actor LIMIT 1]
	at org.hibernate.query.hql.internal.StandardHqlTranslator$1.syntaxError(StandardHqlTranslator.java:108)
	at org.antlr.v4.runtime.ProxyErrorListener.syntaxError(ProxyErrorListener.java:41)
...
org.hibernate.query.SyntaxException: At 1:33 and token 'LIMIT', mismatched input 'LIMIT', expecting one of the following tokens: <EOF>, ',', CROSS, FULL, GROUP, INNER, JOIN, LEFT, ORDER, OUTER, RIGHT, WHERE [SELECT Actor FROM Actor AS Actor LIMIT 1]
java.lang.IllegalArgumentException: org.hibernate.query.SyntaxException: At 1:33 and token 'LIMIT', mismatched input 'LIMIT', expecting one of the following tokens: <EOF>, ',', CROSS, FULL, GROUP, INNER, JOIN, LEFT, ORDER, OUTER, RIGHT, WHERE [SELECT Actor FROM Actor AS Actor LIMIT 1]
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:143)
	at org.hibernate.internal.ExceptionConverterImpl.convert(ExceptionConverterImpl.java:167)
...
Caused by: org.hibernate.query.SyntaxException: At 1:33 and token 'LIMIT', mismatched input 'LIMIT', expecting one of the following tokens: <EOF>, ',', CROSS, FULL, GROUP, INNER, JOIN, LEFT, ORDER, OUTER, RIGHT, WHERE [SELECT Actor FROM Actor AS Actor LIMIT 1]
	at app//org.hibernate.query.hql.internal.StandardHqlTranslator$1.syntaxError(StandardHqlTranslator.java:108)
	at app//org.antlr.v4.runtime.ProxyErrorListener.syntaxError(ProxyErrorListener.java:41)
...
```

원인은 Hibernate의 `limit` 쿼리가 필수적으로 orderBy 절을 요구하기 때문에 발생한 것이다. 따라서 `limit` 을 사용하려면 `orderBy` 절을 함께 사용하거나, `limit` 을 사용하지 않고 페이지네이션을 하는 방법, 또는 `EntityManager`의 `setMaxResults`를 사용하는 방법 중 하나를 선택하여 이 문제를 해결해야 했다.

현재는 우리가 `limit`을 적용한 함수에 `orderBy` 절이 함께 있어서 문제가 되지 않았지만, 무조건 `limit`을 사용할 때 `orderBy` 절을 함께 사용하는 것은 DB 성능을 불필요하게 낭비하는 것이라 판단되어, `EntityManager`의 `setMaxResults`를 사용하는 방법을 고려해 보고 있다.

```kotlin
override fun findRecentOrderSheetsByStoreId(
    storeId: UUID,
    count: Int,
): List<OrderSheet> {
    val query = jpql {
        select(entity(OrderSheet::class))
              .from(entity(OrderSheet::class))
              .where(path(OrderSheet::storeId).eq(storeId))
    }
    
    return entityManager
            .createQuery(query, jpqlRenderContext)
            .apply { setMaxResults(count) }
            .resultList
}
```

## 활용팁

여기서는 Kotlin JDSL에서 공식적으로 지원하지 않지만, 우리 팀에서 좀 더 효율적으로 Kotlin JDSL을 사용하기 위한 여러 가지 팁들을 소개하고자 한다.

### Helper 클래스

우선 우리는 여러 상황에서 발생하는 코드 중복을 줄이고 Repository의 의존성을 줄이고자 Helper 클래스를 두고 아래와 같이 Repository가 `KotlinJdslJpqlExecutor`가 아닌 Helper 클래스인 `JdslExecutorSupport`를 의존하도록 기존 코드를 리펙토링하였다.

```kotlin
class CustomOrderSheetRepositoryImpl(
    private val jdslExecutorSupport: JdslExecutorSupport,
) : CustomOrderSheetRepository
```

그럼 어떤 상황들로 인해 Helper 클래스가 필요하게 되었는지 알아보자.

#### 데이터베이스 잠금

`KotlinJdslJpqlExecutor`를 사용하면 우선 대부분의 쿼리를 작성하는 데에는 큰 문제가 없었다. 하지만 위에서 말씀드린 이슈 중에 데이터베이스의 잠금 기능을 사용해야 하는 경우 `EntityManager`를 사용할 수밖에 없다는 것을 보았을 것이다.

이러면 아래와 같이 Custom Repository에서 `EntityManager`와 `RenderContext`를 추가적으로 의존성을 추가해 주어 사용해야 할 불편함이 있다.

```kotlin
class CustomBillRepositoryImpl(
    private val kotlinJdslJpqlExecutor: KotlinJdslJpqlExecutor,
    private val entityManager: EntityManager,
    private val jpqlRenderContext: RenderContext,
) : CustomBillRepository {
    override fun findByIdUsingPessimisticLock(id: UUID): Optional<OrderSheet> {
        val query = jpql {
            select(entity(OrderSheet::class))
                .from(eneity(OrderSheet::class))
                .where(path(OrderSheet::getId).eq(id))
        }
            
        return entityManager
            .createQuery(query, jpqlRenderContext)
            .apply {
                lockMode = LockModeType.PESSIMISTIC_WRITE
                setHint(HINT_CACHEABLE, false)
                setHint(HINT_SPEC_LOCK_TIMEOUT, 10000)
            }
            .resultList
            .firstOrNull()
            .let { Optional.ofNullable(it) }
    }
}
```

Repository 입장에서는 쿼리를 실행하기 위한 `KotlinJdslJpqlExecutor`만 의존하는 것이 이상적일 것으로 보인다. 반면 보다 세부적인 `EntityManager`와 `RenderContext`를 알아야 한다는 것은 의존성 구조를 복잡하게 만든다고 생각된다. 또한, 잠금을 사용하는 함수의 존재 여부에 따라 개발자가 의존성을 추가해야 하는 부분도 좋지 않은 영향을 미친다고 생각된다.

LockMode를 설정하는 코드도 살펴보면, 이를 공통 함수로 추출한다면 중복된 LockMode와 Hint 설정을 줄일 수 있다. 이로써 개발자가 실수로 `setHint(HINT_SPEC_LOCK_TIMEOUT, 10000)`를 빼먹는 등의 오류를 줄일 수 있습니다.

따라서 우리는 Helper 클래스에 `findWithPessimisticLock`를 만들어서 위와 같은 문제를 해결하려고 하였다.

```kotlin
@Component
class JdslExecutorSupport(
    private val kotlinJdslJpqlExecutor: KotlinJdslJpqlExecutor,
    private val entityManager: EntityManager,
    private val jpqlRenderContext: RenderContext,
) {
    fun <T : Any> findWithPessimisticLock(init: JpqlDsl.() -> JpqlQueryable<SelectQuery<T>>): Optional<T> {
        return entityManager
            .createQuery(jpql(CustomJpql, init), jpqlRenderContext)
            .apply {
                lockMode = LockModeType.PESSIMISTIC_WRITE
                setHint(HINT_CACHEABLE, false)
                setHint(HINT_SPEC_LOCK_TIMEOUT, 10000)
            }
            .resultList
            .firstOrNull()
            .let { Optional.ofNullable(it) }
    }
}

class CustomOrderSheetRepositoryImpl(
    private val jdslExecutorSupport: JdslExecutorSupport,
) : CustomOrderSheetRepository {
    override fun findByIdUsingPessimisticLock(id: UUID): Optional<OrderSheet> {
        return jdslExecutorSupport
            .findWithPessimisticLock {
                selectFrom(OrderSheet::class)
                    .where(path(OrderSheet::getId).eq(id))
            }
    }
}
```

`CustomOrderSheetRepositoryImpl`를 보면 확실히 의존하는 클래스도 줄어들었고 함수도 단순하게 변경된 것을 볼 수 있다. 조금 더 응용하면 `findAllWithSkipLock`, `findAllWithOptimisticLock` 등과 같은 함수들도 만들 수 있을 것이다.

#### Nullable Collection 처리

기본적으로 `KotlinJdslJpqlExecutor`의 `findAll`, `findPage`, `findSlice`는 nullable 한 타입을 가진 Collection을 반환한다. 이에 따라 매번 nullable 한 요소들을 non-nullable하도록 필터링해 주거나 nullable 한 Collection을 반환해야 하는 불편함이 있었다.

```kotlin
override fun findByFilter(
    filter: SearchFilter<Admin>
): List<Admin> {
    return kotlinJdslJpqlExecutor
        .findAll {
            select(entity(Admin::class))
                .from(entity(Admin::class))
                .whereAnd(
                    path(Admin::email).like("%${filter.email}%"),
                )
        }
        .filterNotNull()
}
```

Kotlin JDSL의 과거 [이슈](https://github.com/line/kotlin-jdsl/issues/608){: target="\_blank" }를 보면 nullable 한 Collection을 반환하는 이유를 설명하고 있다.

![kotlin-jdsl-null-type-issue](/assets/images/posts/transfer-jdsl/kotlin-jdsl-null-type-issue.png)

우리는 nullable 한 항목을 반환하는 경우가 거의 없으므로 Helper 클래스에 nullable 한 항목을 필터링하여 non-nullable 한 Collection을 기본적으로 반환하는 함수를 새롭게 만들어 사용했다.

```kotlin
@Component
class JdslExecutorSupport(
    private val kotlinJdslJpqlExecutor: KotlinJdslJpqlExecutor,
    private val entityManager: EntityManager,
    private val jpqlRenderContext: RenderContext,
) {
    fun <T : Any> findAll(init: JpqlDsl.() -> JpqlQueryable<SelectQuery<T>>): List<T> {
        return kotlinJdslJpqlExecutor
            .findAll(CustomJpql, init)
            .filterNotNull()
    }
}
```

위 함수를 통해 이제 더 이상 아래와 같이 매번 `filterNotNull`함수를 적어주지 않아도 되게 개선되었다.

```kotlin
override fun findByFilter(
    filter: SearchFilter<Admin>
): List<Admin> {
    return jdslExecutorSupport
        .findAll {
            select(entity(Admin::class))
                .from(entity(Admin::class))
                .whereAnd(
                    path(Admin::email).like("%${filter.email}%"),
                )
        }
}
```

#### 단일조회

Kotlin JDSL에서 제공하는 `KotlinJdslJpqlExecutor`에서는 단일 조회함수가 별도로 존재하지 않는다. 그래서 만약 단일 데이터를 조회하려면 아래와 같이 Kotlin에서 제공하는 `List#firstOrNull`, `List#first`, `List#find` 등의 함수를 사용해야 한다.

```kotlin
override fun findActive(
    orderableVendorId: UUID,
    storeId: UUID,
): Optional<Bill> {
    return kotlinJdslJpqlExecutor
        .findAll {
            select(entity(Bill::class))
                .from(entity(Bill::class))
                .whereAnd(
                    path(Bill::orderableVendor)(OrderableVendor::getId).eq(orderableVendorId),
                    path(Bill::store)(Store::getId).eq(storeId),
                    path(Bill::state).eq(BillState.ACTIVE),
                )
        }
        .firstOrNull()
        .let { Optional.ofNullable(it) }
}
```

매번 위와 같은 코드를 작성하는 것은 번거롭기 때문에 우리는 Helper 클래스를 통해 단일 조회 함수들을 만들어서 사용하였다.

```kotlin
@Component
class JdslExecutorSupport(
    private val kotlinJdslJpqlExecutor: KotlinJdslJpqlExecutor,
    private val entityManager: EntityManager,
    private val jpqlRenderContext: RenderContext,
) {
    fun <T : Any> find(init: Jpql.() -> JpqlQueryable<SelectQuery<T>>): Optional<T> {
        return getOrNull(init).let { Optional.ofNullable(it) }
    }

    fun <T : Any> getOrNull(init: Jpql.() -> JpqlQueryable<SelectQuery<T>>): T? {
        return kotlinJdslJpqlExecutor
            .findAll(init)
            .firstOrNull()
    }
}

override fun findActive(
    orderableVendorId: UUID,
    storeId: UUID,
): Optional<Bill> {
    return jdslExecutorSupport
        .find {
            selectFrom(Bill::class)
                .whereAnd(
                    path(Bill::orderableVendor)(OrderableVendor::getId).eq(orderableVendorId),
                    path(Bill::store)(Store::getId).eq(storeId),
                    path(Bill::state).eq(BillState.ACTIVE),
                )
        }
}
```

추가로, `JdslExecutorSupport#find`나 `JdslExecutorSupport#getOrNull` 함수를 사용하여 단일 데이터를 조회할 때 의도치 않게 1개 이상의 쿼리를 작성하여 원치 않는 조회 비용이 발생할 수 있는 우려가 있었다.

이 문제를 회피하기 위해 쿼리에 `limit` 구문을 추가하는 방법이 있지만, 앞서 언급한 `limit` 쿼리 이슈로 인해 불필요하게 `orderBy` 절을 사용해야 하거나, 매번 `limit` 구문을 추가하는 번거로움과 코드 중복을 유발한다는 문제가 있다. 이에 따라 우리는 단일 조회 시 1개의 row만 반환하도록 `setMaxResults`를 활용하여 최적화를 시도했다.

```kotlin
fun <T : Any> getOrNull(init: Jpql.() -> JpqlQueryable<SelectQuery<T>>): T? {
    return entityManager
            .createQuery(init, jpqlRenderContext)
            .apply { setMaxResults(1) }
            .resultList
            .firstOrNull()
}
```

### Custom DSL 활용

Kotlin JDSL은 공식적으로 지원하는 DSL 외에도 개발자가 자신만의 DSL을 만들어 사용할 수 있도록 Custom DSL을 제공한다. 우리도 아래와 같이 Custom DSL을 적용하여 사용하고 있는데, 적용한 함수들이 어떤 목적으로 사용되었는지 소개해 보고자 한다.

```kotlin
class CustomJpql : Jpql() {
    private val falsePredicate get() = booleanLiteral(false).eq(booleanLiteral(true))
    val truePredicate get() = booleanLiteral(true).eq(booleanLiteral(true))

    companion object Constructor : JpqlDsl.Constructor<CustomJpql> {
        override fun newInstance(): CustomJpql = CustomJpql()
    }

    fun <T : Enum<*>> Expressionable<Array<T>>.arrayContains(value: T): Predicate {
        return JpqlArrayContain(this.toExpression(), value)
    }

    fun <T> Expressionable<List<T>>.jsonbExistsAny(value: List<T>): Predicate {
        return JpqlJsonbExistsAny(this.toExpression(), value(value.map { it.toString() }.toTypedArray()))
    }

    fun <T : Any, S : T?> Expressionable<T>.`in`(compareValues: Collection<S>): Predicate {
        if (compareValues.isEmpty()) {
            return falsePredicate
        }

        return Predicates.`in`(this.toExpression(), compareValues.map { Expressions.value(it) })
    }

    inline fun <reified T : Any> selectFrom(type: KClass<T>): SelectQueryWhereStep<T> {
        val entity = entity(type)
        return select(entity).from(entity)
    }
}
```

#### 데이터베이스 함수 표현

우리는 PostgreSQL을 사용하고 있는데, PostgreSQL의 함수들(`ARRAY_POSITION({0}, {1}) is not null`, `FUNCTION('jsonb_exists_any', {0}, {1}) = true`)을 DSL로 표현하여 사용하기 위해서 아래와 같이 `arrayContains`, `jsonbExistsAny` 함수를 만들어 활용하고 있다.

```kotlin
data class JpqlArrayContain<T : Enum<*>>(
    val value: Expression<Array<T>>,
    val compareValues: T,
) : Predicate

object JpqlArrayContainsSerializer : JpqlSerializer<JpqlArrayContain<*>> {
    override fun handledType(): KClass<JpqlArrayContain<*>> {
        return JpqlArrayContain::class
    }

    override fun serialize(
        part: JpqlArrayContain<*>,
        writer: JpqlWriter,
        context: RenderContext,
    ) {
        val delegate = context.getValue(JpqlRenderSerializer)

        // ARRAY_POSITION({0}, {1}) is not null
        writer.write("ARRAY_POSITION(")
        delegate.serialize(part.value, writer, context)
        writer.write(", ")
        writer.write("'${part.compareValues.name}'")
        writer.write(") IS NOT NULL")
    }
}

data class JpqlJsonbExistsAny<T>(
    val value: Expression<List<T>>,
    val compareValues: Expression<Array<*>>,
) : Predicate

object JpqlJsonbExistsAnySerializer : JpqlSerializer<JpqlJsonbExistsAny<*>> {
    override fun handledType(): KClass<JpqlJsonbExistsAny<*>> {
        return JpqlJsonbExistsAny::class
    }

    override fun serialize(
        part: JpqlJsonbExistsAny<*>,
        writer: JpqlWriter,
        context: RenderContext,
    ) {
        val delegate = context.getValue(JpqlRenderSerializer)

        // FUNCTION('jsonb_exists_any', {0}, {1}) = true
        writer.write("FUNCTION('jsonb_exists_any', ")
        delegate.serialize(part.value, writer, context)
        writer.write(", ")
        delegate.serialize(part.compareValues, writer, context)
        writer.write(") = true")
    }
}

fun <T : Enum<*>> Expressionable<Array<T>>.arrayContains(value: T): Predicate {
    return JpqlArrayContain(this.toExpression(), value)
}

fun <T> Expressionable<List<T>>.jsonbExistsAny(value: List<T>): Predicate {
    return JpqlJsonbExistsAny(this.toExpression(), value(value.map { it.toString() }.toTypedArray()))
}
```

이렇게 Custom DSL을 만들면 아래처럼 단순하게 복잡한 쿼리를 실행시킬 수 있다.

```kotlin
override fun findRecommendations(
    category: MajorTradeStoreCategory,
    towns: List<Town>,
): List<OrderableVendor> {
    return jdslExecutorSupport
        .findAll {
            selectFrom(OrderableVendor::class)
                .whereAnd(
                    path(OrderableVendor::metaInfo)
                        .path(MetaOrderableVendor::advertisementEnabled)
                        .eq(true),
                    path(OrderableVendor::metaInfo)
                        .path(MetaOrderableVendor::productInfo)
                        .path(OrderableVendorProductInfo::majorTradeStoreCategories)
                        .arrayContains(category),
                    path(OrderableVendor::metaInfo)
                        .path(MetaOrderableVendor::deliveryInfo)
                        .path(OrderableVendorDeliveryInfo::deliveryTowns)
                        .jsonbExistsAny(towns),
                )
        }
}
```

```sql
select
  *
from
  orderable_vendor o1_0
  join meta_orderable_vendor m1_0 on m1_0.id = o1_0.meta_orderable_vendor_id
where
  (m1_0.advertisement_enabled = true)
  and (
    array_position(m1_0.major_trade_store_categories, 'KOREAN') is not null
  )
  and (
    jsonb_exists_any(m1_0.delivery_towns, '{"GANGNAM_GU"}') = true
  );
```

#### Custom In

`in` 절을 사용할 때, Querydsl은 매개변수가 빈 배열이어도 false로 처리하고 오류를 반환하지 않지만, Kotlin JDSL의 경우 오류를 반환하는 이슈가 있었다. 어느 구현이 맞다고 말하기 어렵다고 판단하여 별도로 이슈를 제기하지는 않았는데, 문제는 클라이언트에서 빈 배열을 보내는 형태로 로직이 구현되어 있다는 것이 이슈였다. 그래서 우리는 아래와 같이 3가지 방법을 고민하였다.

- **빈 배열이 들어올 때 예외를 발생시킨다.** 명시적이라 잘못된 매개변수를 넘기는 경우에 대한 인지가 가능하다는 장점이 있지만 프론트에서 매번 예외 처리해야 하므로 번거로움이 있다.
- **빈 배열이 들어오면 매개변수를 넘기지 않은 것과 같이 모든 데이터를 반환한다.** 원치 않는 모든 데이터를 반환하므로 성능적인 이슈가 있을 수 있다. 빈 배열을 넘겼는데 모든 값이 넘어온다는 가정이 일반적인 동작 방식인가 고민해 보았을 때 일반적인 방식은 아니라고 판단했다.
- **빈 배열이 들어오면 false로 처리한다.** 즉, 일치하는 조건이 없으므로 데이터를 반환하지 않는다. Querydsl은 이렇게 동작하도록 처리되어 있다. 일리가 있는 동작 방식이나 SQL 문법상으로는 알맞지 않은 것 같다.

결국 우리는 기존 동작 방식을 변경하지 않는다는 기본 전제조건을 지키기 위해서 3번으로 구현하기로 하였다.

한편, `in` 함수명에 대한 고민도 있었는데, 왜냐하면 `CustomJpql`의 부모 클래스인 `Jpql`에서 이미 인스턴스 확장함수로 `in`을 선언해 두었기 때문이다. 그래서 처음에는 `customIn`이라는 함수명을 사용했으나 `in` 절을 사용할 때 자칫 `customIn`을 사용하는 것을 잊어버리고 `in`함수를 사용할까 우려되었다. 거기다 확장함수라 `Jpql`의 함수를 오버라이딩할 수 없었는데, 다행히 [Kotlin 확장함수 우선순위 정책](https://kotlinlang.org/docs/extensions.html#declaring-extensions-as-members){: target="\_blank" }에 따르면 동일한 함수명이 동일한 경우 맴버 함수가 우선된다는 점을 이용하여 아래와 같이 `CustomJpql`의 인스턴스 확장함수로 정의하였다.

이로써 빈 배열이 들어오면 false로 처리되도록 조치되었다.


```kotlin
class CustomJpql : Jpql() {
    private val falsePredicate get() = booleanLiteral(false).eq(booleanLiteral(true))

    fun <T : Any, S : T?> Expressionable<T>.`in`(compareValues: Collection<S>): Predicate {
        if (compareValues.isEmpty()) {
            return falsePredicate
        }

        return Predicates.`in`(this.toExpression(), compareValues.map { Expressions.value(it) })
    }
}

override fun findAllActive(
    orderableVendorId: UUID,
    storeIds: List<UUID>,
): List<Bill> {
    return jdslExecutorSupport
        .findAllWithPessimisticLock {
            selectFrom(Bill::class)
                .whereAnd(
                    path(Bill::orderableVendor)(OrderableVendor::getId).eq(orderableVendorId),
                    path(Bill::store)(Store::getId).`in`(storeIds),
                    path(Bill::state).eq(BillState.ACTIVE),
                )
        }
}

test("빈 매장 ID 목록이 주어지면 빈 목록을 반환한다.") {
    // Given
    // ...생략

    // When
    val actual = billRepository.findAllActive(
        orderableVendorId = orderableVendor.id,
        storeIds = listOf(),
    )

    // Then
    actual.shouldBeEmpty()
}
```

```sql
select
  *
from
  bill b1_0
where
  (
    b1_0.orderable_vendor_id = '018f39dd-e8f2-ee4f-7041-1d55cdea8fd6'
  )
  and (false = true)
  and (b1_0.state = 'ACTIVE') for no key
update
;
```

#### selectFrom

솔직히 해당 팁은 너무 단순해서 소개할지 말지 고민하다가, 그래도 기왕에 구현한 것을 소개해 보자고 생각해서 소개하고자 한다.

Kotlin JDSL 쿼리를 작성하다보면 상당수의 쿼리가 아래와 같이 Entity를 반환하는 경우가 많다.

```kotlin
override fun findAllActive(
    storeId: UUID,
    pageRequest: Pageable,
): Page<Bill> {
    return jdslExecutorSupport
        .findPage(pageRequest) {
            select(entity(Bill::class))
                .from(entity(Bill::class))
                .whereAnd(
                    path(Bill::store)(Store::getId).eq(storeId),
                    path(Bill::state).eq(BillState.ACTIVE),
                )
                .orderBy(path(Bill::orderableVendor)(OrderableVendor::name).asc())
        }
}
```

그러다 보니 `select(entity(Entity::Class)).from(entity(Entity::class))` 코드가 반복되는 쿼리들이 보였다. 그래서 해당 코드의 중복도 줄이고 단순하게 작성할 수 있도록 아래와 같이 `selectFrom`이라는 함수를 추가하여 해당 코드들에 적용하였다. (해당 함수는 Querydsl의 `JPAQueryFactory#selectFrom`함수를 참고하였다)

```kotlin
inline fun <reified T : Any> selectFrom(type: KClass<T>): SelectQueryWhereStep<T> {
    val entity = entity(type)
    return select(entity).from(entity)
}

override fun findAllActive(
    storeId: UUID,
    pageRequest: Pageable,
): Page<Bill> {
    return jdslExecutorSupport
        .findPage(pageRequest) {
            selectFrom(Bill::class)
                .whereAnd(
                    path(Bill::store)(Store::getId).eq(storeId),
                    path(Bill::state).eq(BillState.ACTIVE),
                )
                .orderBy(path(Bill::orderableVendor)(OrderableVendor::name).asc())
        }
}
```

### ManyToMany 이슈

저희가 개발 중인 서비스의 수많은 Entity 중 N:M 관계를 맺은 Entity가 몇 개 있다. N:M 관계를 맺은 Entity들 중 일부는 아래와 같이 ManyToMany로 표현해서 사용하고 있다.

```kotlin
@Entity
class Product {
    // ...생략
    
    @ManyToMany(fetch = FetchType.LAZY, mappedBy = "mutableProducts")
    protected val mutableTags: MutableList<OrderableVendorProductTag> = mutableListOf()
    val tags: List<Tag> get() = mutableTags.toList()
}

@Entity
class Tag {
    // ...생략
    
    @ManyToMany(fetch = FetchType.LAZY, cascade = [CascadeType.PERSIST, CascadeType.MERGE])
    @JoinTable(
        name = "product_tag_assoc",
        joinColumns = [JoinColumn(name = "tag_id")],
        inverseJoinColumns = [JoinColumn(name = "product_id")],
    )
    protected var mutableProducts: MutableList<Product> = mutableListOf()
    val products: List<Product> get() = mutableProducts.toList()
}
```

문제는 아래와 같이 `tag_id`를 가진 `Product`를 조회하는 쿼리를 작성할 때 Kotlin JDSL로 위에 정의된 Entity로는 표현할 방법을 찾을 수 없었다는 것이었다.

```sql
select
  *
from
  product p
where
  exists(
    select
      1
    from
      product_tag_assoc a
    where
      a.tag_id = '018efa72-0b67-e9a3-29e1-a44576a66bba'
      and a.product_id = p.id
  )
```

그래서 어쩔 수 없이 Kotlin JDSL 쿼리를 위한 관계 Entity를 아래와 같이 추가한 후 쿼리를 만들어서 해결하였다.

```kotlin
@Entity
@IdClass(ProductTagAssocId::class)
class ProductTagAssoc(
    id: ProductTagAssocId,
) {
    @Id
    @Column(name = "product_id", nullable = false)
    val productId: UUID = id.productId

    @Id
    @Column(name = "tag_id", nullable = false)
    val tagId: UUID = id.tagId
}

data class ProductTagAssocId(
    val productId: UUID,
    val tagId: UUID,
) : Serializable


override fun findByTagId(
    tagId: UUID,
): List<Product> {
    return jdslExecutorSupport
        .findAll {
            selectFrom(Product::class)
                .where(
                    exists(
                        selectFrom(ProductTagAssoc::class)
                            .whereAnd(
                                path(ProductTagAssoc::tagId).eq(tagId),
                                path(ProductTagAssoc::productId).eq(path(Product::getId)),
                            )
                            .asSubquery()
                    )
                )
        }
}
```

위와 같은 방식이 가장 좋은 방식인지는 잘 모르겠다. Kotlin JDSL의 문서나 예시 코드에도 위와 같은 사례를 찾아볼 순 없어서 일단 문제 해결에 초점을 맞추어 조치하였는데, 해당 부분은 한번 Kotlin JDSL측에 문의해 보아야겠다.

### Literals

아래와 같이 특정 컬럼의 존재 여부에 따라서 정렬하고 싶은 경우 CASE 문과 함께 상숫값을 넣어줘야 하는 경우가 있었다.

```sql
select
  *
from
  product p
where
  1 = 1
order by
  case
    when p.price is null then 2
    else 1
  end asc,
  p.name asc offset 2 rows
fetch first
  2 rows only;
```

처음에 아래와 같이 쿼리를 작성해 보았다.

```kotlin
override fun findByFilter(
    pageable: Pageable,
): Page<OrderableVendorCatalogProduct> {
    return jdslExecutorSupport
        .findPage(pageable) {
            selectFrom(Product::class)
                .orderBy(
                    caseWhen(path(Product::price).isNull())
                        .then(2)
                        .`else`(1)
                        .asc(),
                    path(Product::name).asc(),
                )
        }
}
```

해당 함수를 실행해 보면 아래와 같이 오류가 발생함을 볼 수 있다.

```
Could not locate named parameter [param1], expecting one of []
org.springframework.dao.InvalidDataAccessApiUsageException: Could not locate named parameter [param1], expecting one of []
	at org.springframework.orm.jpa.EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible(EntityManagerFactoryUtils.java:371)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:246)
...
Caused by: java.lang.IllegalArgumentException: Could not locate named parameter [param1], expecting one of []
	at org.hibernate.query.internal.ParameterMetadataImpl.getQueryParameter(ParameterMetadataImpl.java:263)
	at org.hibernate.query.spi.AbstractCommonQueryContract.setParameter(AbstractCommonQueryContract.java:844)
...
Caused by: org.hibernate.query.UnknownParameterException: Could not locate named parameter [param1], expecting one of []
	... 519 more
```

원인은 `then(2)`과 같이 매개변수를 전달하면 내부적으로는 `then(value(2))`으로 동작하게 되고 `value`는 JPQL로 변환할 때 쿼리 매개변수(`param1`)로 변경되는데 실제로 페이징 쿼리를 수행할 때 `param1`을 치환할 값이 없으니, 오류가 발생하는 것으로 추측된다.

```kotlin
fun writeParam(value: Any?) {
    val name = "param${incrementer.getNext()}"

    writeParam(name, value)
}
```

그래서 전달된 값 그대로 쿼리로 치환할 수 있도록 intLiteral을 사용하여 위와 같은 오류를 해결하였다.

```kotlin
override fun findByFilter(
    pageable: Pageable,
): Page<OrderableVendorCatalogProduct> {
    return jdslExecutorSupport
        .findPage(pageable) {
            selectFrom(Product::class)
                .orderBy(
                    caseWhen(path(Product::price).isNull())
                        .then(intLiteral(2))
                        .`else`(intLiteral(1))
                        .asc(),
                    path(Product::name).asc(),
                )
        }
}
```

## 마치며

지금까지 Querydsl에서 Kotlin JDSL으로 어떻게 전환하였고 전환하면서 겪었던 이슈들과 팁들을 소개해 보았다. Kotlin JDSL에 대한 레퍼런스가 많지 않아 우리가 겪었던 경험을 최대한 소개해 보려고 하다 보니 다소 내용이 길어진 감이 있는 점 양해바란다.

개인적으로 Querydsl에서 Kotlin JDSL으로 전환은 만족스러운 결과를 얻었다고 생각한다. 메타모델을 별도로 필요로 하지 않고 Entity의 Property를 통해 쿼리를 작성하기 때문에 쿼리를 위해 Entity에서 외부로 노출하지 않아도 될 Property를 어쩔 수 없이 노출해야 할 단점은 있었지만, 메타모델 생성을 위한 복잡한 설정을 하지 않아도 되고 DSL로 직관적이고 단순한 쿼리를 통해 손쉽게 쿼리를 생성할 수 있다는 부분에서 큰 매력으로 다가왔다.

그리고 무엇보다 더 이상 kapt를 사용하지 않고 메타모델을 위한 Code Generation을 실행하지 않다 보니 컴파일 시간을 상당히 줄일 수 있다는 점에서도 큰 장점이라고 생각된다.

#### 전환 전 - 1분 22초

![as-is-transferr](/assets/images/posts/transfer-jdsl/as-is-transferr.png)

#### 전환 후 - 34초

![to-be-transfer](/assets/images/posts/transfer-jdsl/to-be-transfer.png)

모쪼록 이 글을 통해 Querydsl을 다른 쿼리 빌더로 전환할 계획을 세우거나 새로운 프로젝트에 도입할 만한 쿼리 빌더를 고민하고 있는 분들께 작은 도움이 되었길 바라면서 글을 마무리 짓도록 하겠다.
