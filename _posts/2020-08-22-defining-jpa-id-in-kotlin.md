---
layout: single
title: "Defining JPA's ID in Kotlin"
categories:
  - EXPLANATION
tags:
  - kotlin
  - jpa
  - id

toc: true
toc_sticky: true
excerpt: Kotlin으로 Spring Data JPA를 사용할 때 많은 고민을 하였다. Entity Class를 정의할 때 특히 그런데, `Data Class`를 사용할지 말지, `val`를 사용할지 `var`를 사용할지 말지 등등 항상 어떻게 하는게 좀더 의미있는지 고민이 된다. 이 글은 그중에서 ID를 정의할 때 어떻게 정의하면 좋을지에 대한 나의 생각을 정리한 글이다.
---

## Intro

Kotlin으로 Spring Data JPA를 사용할 때 많은 고민을 하였다. Entity Class를 정의할 때 특히 그런데, `Data Class`를 사용할지 말지, `val`를 사용할지 `var`를 사용할지 말지 등등 항상 어떻게 하는게 좀더 의미있는지 고민이 된다. 이 글은 그중에서 ID를 정의할 때 어떻게 정의하면 좋을지에 대한 나의 생각을 정리한 글이다.

최근 Kotlin 사용자가 많아지면서 Kotlin을 이용한 JPA Tutorial 글들을 많이 볼 수 있다. 그 중에 많은 글들에서 ID property 즉 primary key가 아래와 같이 nullable로 정의되어 있는 것을 볼 수 있었다.

```kotlin
@Entity
class Person(
    @Id
    var id: Long?,

    var name: String,

    var age: Int
)
```

아마 Spring Data에서 `save`함수를 사용할 시 `insert`를 하려면 ID가 null이어야 하기 때문이라고 생각해서 이렇게 작성되었을 것이라 예상해 본다.

하지만 나는 nullable이 되어서는 안된다고 생각한다. 특히 Long타입의 ID인 경우 nullable로 정의되지 않아도 되기 때문에 더더욱 nullable로 정의하면 안된다. 아래에 좀더 자세하게 그 이유에 대해서 적어보자

### RDBMS에서 Primary Key는 null값을 가지지 않는다.

Wiki의 [Primary key](https://en.wikipedia.org/wiki/Primary_key){: target="\_blank" }에 대한 정의를 보면 Primary key는 `NOT NULL`인것으로 정의된다고 적혀 있다. JPA의 Entity는 Database를 모델링한 것이므로 Database의 속성들과 다르게 정의되어서는 안된다.

### Long 타입의 ID는 null로 저장하지 않아도 insert가 가능하다.

Spring Data에서 제공하는 Repository의 save 함수를 살펴 보면 [`EntityInformation.isNew`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/core/EntityInformation.html){: target="\_blank" }함수를 찾아볼 수 있다. 이 함수는 Entity를 영속화 할때 insert인지 update인지 판단해주는 함수이다. Spring Data JPA를 사용한다면 `isNew`함수의 구현체는 [`AbstractEntityInformation`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/core/support/AbstractEntityInformation.html){: target="\_blank" }클래스인데, 해당 클래스에서 `isNew`함수를 구현한 내용을 적어보면 아래와 같다.

```java
public boolean isNew(T entity) {

    ID id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
        return id == null;
    }

    if (id instanceof Number) {
        return ((Number) id).longValue() == 0L;
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s!", idType));
}
```

코드에서 보는바와 같이 ID가 Number 타입인 경우 값이 0이면 `isNew`의 결과가 `true`임을 알 수 있다. 결국 ID의 값이 0인 Entity를 영속화 하면 insert가 실행되는 것이다.

### Kotlin에서 nullable 변수는 사용성이 떨어진다.

Kotlin은 언어레벨에서 null safe하게 작성할 수 있다. 그래서 만약 변수가 nullable 하다면 `!!`, `?:` 키워드를 사용하거나 별도로 null 타입 체크를 해주지 않으면 코드 작성에 불편함이 있다. 물론 가독성에도 영향을 준다고 생각한다.

```kotlin

val nullableString: String? = "sample"
            
val length = nullableString?.length ?: 0

val string = nullableString.orEmpty()

val string2 = nullableString!!

```

## Conclusion

정리하자면 위에서 말한 이유로 Long 타입의 ID를 정의하는 경우에는 아래와 같이 사용하면 좋을 것 같다.

```kotlin
@Entity
class Person(
    @Id
    var id: Long,

    var name: String,

    var age: Int
)
```

> [https://en.wikipedia.org/wiki/Primary_key](https://en.wikipedia.org/wiki/Primary_key){: target="\_blank" }
