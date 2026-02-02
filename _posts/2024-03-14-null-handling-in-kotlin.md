---
layout: single
title: "Null을 피하기만 하는게 마냥 좋은 것일까(feat. Kotlin)"
categories:
  - EXPLANATION
tags:
  - kotlin
  - null
toc: true
toc_sticky: true
excerpt: "블로그를 보다 보면 Null 반환에 대한 글들을 많이 볼 수 있다. 대부분 클린코드에서 말하는 Null 코드는 나쁜 것이니 지양해서 사용해야 한다고 이야기하고 있다. 하지만 과연 나쁘다고 덮어놓고 사용하지 말아야 할까? 특정 언어에서는 Null safety를 지원하지 않기 때문에 장황한 방어코드를 양산해 코드의 가독성을 떨어뜨린다거나 NullPointerException이 발생할 가능성이 크다는 단점은 있지만 적어도 Null safety를 지원하는 언어(Kotlin, Swift, TypeScript, Dart, Rust 등)에서는 이러한 단점을 상당수 보완하였기 때문에 오히려 덮어놓고 지양해야 하는 것은 아니라고 생각한다. 이번 글에서는 현재 내가 사용하고 있는 Kotlin을 통해서 단순히 Null을 피하기보다 어쩔 수 없이 사용해야 할 때 선택할 수 있는 활용 방법에 대해 이야기를 해보고자고자한다."
---

## Intro

블로그를 보다 보면 Null 반환에 대한 글들을 많이 볼 수 있다. 대부분 클린코드에서 말하는 Null 코드는 나쁜 것이니 지양해서 사용해야 한다고 이야기하고 있다. 하지만 과연 나쁘다고 덮어놓고 사용하지 말아야 할까?

특정 언어에서는 Null safety를 지원하지 않기 때문에 장황한 방어코드를 양산해 코드의 가독성을 떨어뜨린다거나 NullPointerException이 발생할 가능성이 크다는 단점은 있지만 적어도 Null safety를 지원하는 언어(Kotlin, Swift, TypeScript, Dart, Rust 등)에서는 이러한 단점을 상당수 보완하였기 때문에 오히려 덮어놓고 지양해야 하는 것은 아니라고 생각한다.

이번 글에서는 현재 내가 사용하고 있는 Kotlin을 통해서 단순히 Null을 피하기보다 어쩔 수 없이 사용해야 할 때 선택할 수 있는 활용 방법에 대해 이야기를 해보고자고자한다.

## Optional vs Nullable

Optional은 객체가 비어있을 수 있는지를 나타내는 래퍼 클래스이다. Null 반환 가능성이 있는 반환 값에 주로 사용하는데 나의 경우 아래와 같이 Repository에서 단일 Entity를 조회하는 경우 주로 활용하고 있었다.

```kotlin
class FooRpository {
    fun findById(id: UUID): Optional<Foo> {
        // 구현 코드...
    }
}
```

그리고 존재하지 않는 Entity를 반환하지 않는 곳에서는 아래와 같이 예외를 발생시켜 의도하지 않게 빈 데이터로 인한 예외가 발생하지 않도록 하였다.

```kotlin
class FooService(
    private val fooRepository: FooRepository,
) {
    fun getById(id: UUID): Foo {
        return adminRepository.findById(id).orElseThrow()
    }
}
```

자바를 사용할 때부터 사용했었고 Null 객체를 다루지 않아서 마냥 좋은 사용 패턴이라 생각하고 있었다. 하지만 어느날 함께 개발하고 있던 동료는 아래와 같이 자신은 Optional을 사용하는 것보다 null을 반환해서 처리하는 게 좀 더 좋다는 의견을 주었다.

```kotlin
class FooRpository {
    fun findById(id: UUID): Foo? {
        // 구현 코드...
    }
}

class FooService(
    private val fooRepository: FooRpository,
) {
    fun getById(id: UUID): Foo {
        return fooRepository.findById(id) ?: throw Exception()
    }
}
```

Null Safety 하므로 NullPointerException 발생 가능성을 상당히 줄일 수 있고, Kotlin에서 지원하는 문법 설탕이 많아 null 객체에 대한 처리가 손쉽다는 것이다. 그리고 Optional 객체를 사용하면 `Optional<Foo>`과 같이 객체를 감싸야 하는 등 코드가 장황해지는데 Nullable하게 반환하도록 하면 `Foo?`로 단순하게 표현할 수 있으니 가독성 측면에서 더 좋아 보인다는 의견도 주었다.

이전까지는 마냥 Null을 반환하는 게 좋지 않다고만 생각해서 Optional을 당연하게 사용해야 한다고 생각했었는데, 위와 같은 의견을 들으면서 생각이 조금 바뀌게 되었다. Kotlin에서는 nullable 객체를 좀 더 손쉽게 다루게 문법 설탕을 제공하는데 이러한 점으로 봐서 오히려 Kotlin은 nullable 객체를 명시적으로 드러내서 좀 더 우아하게 다루도록 권장한다는 느낌을 받을 수 있었다.

또한 Optional 객체 자체는 매우 가벼우나 Optional에 포함된 객체가 무거운 경우에는 Optional을 생성하고 처리하는 데 많은 자원을 소비할 수 있다는 점에서 Optional보다는 nullable 객체 자체를 사용하는 게 좀 더 매력적이다고 생각이 들었다.

## Null Object vs Nullable

Null Object 패턴은 [클린 코드](https://product.kyobobook.co.kr/detail/S000001032980){: target="\_blank" }, [클린 아키텍처](https://product.kyobobook.co.kr/detail/S000001033082){: target="\_blank" }, [리펙터링](https://product.kyobobook.co.kr/detail/S000001810241){: target="\_blank" }, [테스트 주도 개발](https://product.kyobobook.co.kr/detail/S000001032985){: target="\_blank" } 등 저명한 책들에서 소개하거나 인용될 정도로 Null 처리에 대한 기법이다. 앞서 소개한 Optional도 그중 하나이다.

Null Object를 통해 예외 발생 가능성을 줄이고 Null 처리를 보다 유용하게 할 수 있다는 점에서 장점이나 잘못 사용하는 경우 예외 탐지가 어렵다거나 무분별한 Null Object가 만들어질 경우 관리의 어려움 등이 있을 수 있다. 개인적으로 Null Object가 남용되는 것이 두드러지는 문제라고 생각하는데 그래서 되도록 언어 레벨에서 지원하는 객체나 문법을 사용하는 게 좋다고 생각한다.

Kotlin의 경우 아래와 같이 nullable 타입에 대한 처리를 위한 연산자를 제공하거나 확장함수를 지원함으로써 Null Object를 직접 생성하지 않고 손쉽게 nullable 타입을 다룰 수 있도록 지원한다.

```kotlin
val command = readlnOrNull()

println(command.orEmpty())
println(command ?: "입력되지 않았습니다.")
```

## Default/Constants vs Nullable

null을 반환하는 게 싫어서 기본값을 반환하는 코드를 작성하는 경우가 있다. 혹은 `-1`과 같이 정상적으로 처리되지 않았음을 나타내는 상수를 정의하고 null 값을 대신해서 표현하는 경우도 있다. 나는 이와 같은 방식을 단순히 null 값을 반환하는 게 나쁘다는 이유로 무조건 따르는 것은 좋지 않다고 생각한다. null과 같이 원치 않는 값을 반환하는 경우 기본값이나 `-1`과 같은 상수를 반환한다고 해도 예외 처리를 해주어야 하는 것은 매한가지인데 기본값이나 상수를 정의하는 것은 자칫 불필요하거나 과도한 객체를 양산할 수도 있고 코드상 드러나지 않는 규칙을 남용할 수 있기 때문이다.

물론 Java와 같이 Null safety를 지원하지 않는 언어의 경우 대체 값이 유용할 수 있다. 개발자가 null 처리를 하지 않아 NullPointerException이 발생하는 것을 최소한 방지할 수 있기 때문이다. 하지만 Java도 `@Nullable`을 이용해 IDEA의 지원을 받아 개발하거나 정적분석 도구 등을 이용하여 이러한 리스크를 감소시킬 수 있는 다양한 방법이 존재한다.

그러므로 어중간한 기본값이나 상수를 다룸으로 인해 에러 탐지를 어렵게 하기보다 null에 대한 예외 처리를 잘할 방법을  고민해 보는 것도 어떨지 생각한다.

## NullPointerException 발생 사례

아무리 Kotlin이 Null safety를 지원한다고 하더라도 NullPointerException이 발생하지 않는 것은 아니다. 특히 Java 라이브러리를 사용할 때 무심코 nullable 타입을 non-nullable 타입으로 선언하고 사용할 때 많이 발생한다.

아래 Database Table을 Kotlin을 이용하여 JPA의 Entity로 정의한 경우를 살펴보자.

```sql
CREATE TABLE board
(
    id           uuid                     NOT NULL,
    created_at   timestamp with time zone NOT NULL,
    title        varchar                  NOT NULL,
    content      varchar                  NOT NULL,
    label        varchar                  NULL,
    CONSTRAINT board_pk PRIMARY KEY (id)
);
```

```kotlin
@Entity
class Board(
    @Id
    val id: UUID,

    @Column(nullable=false)
    val createdAt: LocalDateTime,

    @Column(nullable=false)
    val title: String,

    @Column(nullable=false)
    val content: String,

    @Column(nullable=false)
    val label: String,
)
```

Board Entity를 보면 `label`은 nullable 칼럼임에도 불구하고 non-nullable 타입으로 정의된 것을 볼 수 있다. 이런 경우 만약 board 테이블의 특정 row에 label 값이 null이 저장되어 있고 이를 Entity로 불러오는 경우 label을 호출하면 NullPointerException이 발생하게 된다. 상당히 단순한 실수이지만 은근히 자주 발생하는 실수이기도 하다.

이러한 사례는 단순히 개발자의 부족한 꼼꼼함으로 인해 발생한 사례라고만 생각하지 않는다. Null 타입에 대한 부정적인 인식이 자신도 모르게 nullable 타입을 정의하지 않는 실수를 유발한게 아니냐는 생각이다. 물론 Kotlin도 의도적으로 `?`를 추가함으로써 Nullable 타입을 정의하는 것을 암묵적으로 지양하도록 유도하고 있다는 생각이 든다. 하지만 Nullable 타입을 효율적으로 처리할 수 있도록 문법적인 지원이 많은 것을 보면 Null 타입이 단순히 나쁘기 때문에 지양하려고 하기보다 반드시 써야 하는 곳에서는 명확하게 정의하여 실수로 인한 오류가 발생하지 않도록 하는 게 아닌가 싶다.

## Migration vs Nullable

일전에 동료 개발자가 품목 Entity에 새로운 속성을 추가하는 개발 건을 진행한 적이 있다. 우리는 주문 Entity에서 과거에 주문했던 주문 품목이 원본 품목의 변경으로 인해 변경되는 것을 방지하고자 품목의 복제본을 만들어 저장하도록 구현하였다.

```kotlin
@Entity
class Product(
    @Id
    val id: UUID,

    @Column(nullable=false)
    val name: String,
)

@Entity
class Order(
    @Id
    val id: UUID,

    @Column(nullable=false)
    val createdAt: LocalDatetime,
) {
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "order", cascade = [CascadeType.ALL], orphanRemoval = true)
    val products: List<OrderProduct> = listOf()
}

@Entity
class OrderProduct(
    @Id
    val id: UUID,

    @Column(nullable=false)
    val productId: UUID,

    @Column(nullable=false)
    val name: String,

    @Column(nullable=false)
    val quantity: Int,
)
```

위 코드에서 동료 개발자는 품목 Entity에 단위 속성을 추가하는 작업을 진행하였고 주문 품목 Entity에도 해당 속성을 추가하려고 하였다.

```kotlin
@Entity
class Product(
    @Id
    val id: UUID,

    @Column(nullable=false)
    val name: String,

    // 추가됨
    @Column(nullable=false)
    val unit: String,
)

@Entity
class OrderProduct(
    @Id
    val id: UUID,

    @Column(nullable=false)
    val productId: UUID,

    @Column(nullable=false)
    val name: String,

    @Column(nullable=false)
    val quantity: Int,

    // 추가됨
    @Column(nullable=false)
    val unit: String,
)
```

여기서 동료는 주문 품목 Entity에 `unit` 속성을 non-nullable 타입으로 정의하고 기존에 주문서 품목 정보들을 마이그레이션을 통해 값을 넣어주고자 계획하였다. (만약 마이그레이션을 하지 않는다면 앞서 말한 예제처럼 NullPointerException이 발생할 수 있다)

하지만 나는 주문 품목을 마이그레이션을 하는 것보다는 `unit` 속성은 nullable 타입으로 하는 게 더 나아 보인다는 의견을 드렸다. 과거의 이력은 과거대로 두어야 한다고 생각했고 과거의 `unit` 속성값이 반드시 현재의 `unit` 속성값이라고 단정지을 수 없기 때문이다. 그래서 차라리 nullable 타입으로 정의하고 아래와 같이 사용자에게 노출할 때 어떻게 표현하면 좋을지 선택해서 표현하도록 하는 게 더 나아 보인다고 생각한다.

```kotlin
@Entity
class OrderProduct(
    @Id
    val id: UUID,

    @Column(nullable=false)
    val productId: UUID,

    @Column(nullable=false)
    val name: String,

    @Column(nullable=false)
    val quantity: Int,

    // 추가됨
    @Column
    val unit: String?,
)

// 선택1
orderProduct.unit.orEmpty()

// 선택2
orderProduct.unit ?: "미지원 품목 단위"
```

이러한 사례도 결국 우리가 Nullable 타입을 지양하는 인식으로 인해 정작 도메인의 중요한 포인트를 놓치게 되는 경우를 말해주고 있다.

## Null Handling 사례

나는 코드를 작성하면서 가끔 Kotlin이 제공하는 Nullable 타입에 대한 연산자를 활용하기 위해 의도적으로 Nullable 타입을 유도하기도 한다. 먼저 아래 코드를 보자.

주문한 품목이 1개 이상인 경우 `~외 N개의 품목이 주문되었습니다.`라는 문구를 만들어보고자 한다.

```kotlin
fun makeOrderMessage(products: List<Product>): String {
    if (products.isEmpty()) {
        return "주문할 품목이 존재하지 않습니다."
    }

    val name = products.first().name
    val extra = takeIf { products.products > 1 }
        ?.let { " 외 ${products.size - 1}개의" }
        .orEmpty()

    return "$name$extra 품목이 주문되었습니다."
}
```

추가 문구를 생성하기 위한 코드를 보면 품목이 1개 이상 존재할 때 의도적으로 nullable 타입을 만들어준 후 코틀린에서 지원하는 [Safe call operator](https://kotlinlang.org/docs/null-safety.html#safe-calls){: target="\_blank" }를 사용하여 추가 문구를 만들어주는 것을 볼 수 있다.

물론 사람마다 취향이 다르므로 아래 코드가 더 좋을 수 있다고 생각할 수 있겠지만 개인적으로는 위에 소개한 코드가 좀 더 Kotlin 스럽다(?)는 느낌이 든다.

```kotlin
fun makeOrderMessage(products: List<Product>): String {
    if (products.isEmpty()) {
        return "주문할 품목이 존재하지 않습니다."
    }

    val name = products.first().name
    val extra = if (bills.size > 1) {
        " 외 ${bills.size - 1}개의"
    } else {
        ""
    }

    return "$name$extra 품목이 주문되었습니다."
}
```

결국 여기서 하고자 하는 말은 Nullable 타입을 잘 활용하면 (사람에 따라 다르겠지만) 더 가독성 있는 코드를 작성할 수 있으므로 Nullable 타입에 대한 무조건적인 배척보다는 활용 방법도 함께 모색해 볼 수 있다는 말이다.

## 마무리

지금까지 "Null을 피하기만 하는 게 마냥 좋은 것일까"라는 다소 자극적일 수 있는 주제로 내 생각을 이야기해 보았다. 다년간 Java를 사용하면서 수없이 많은 NullPointerException을 겪어본 1인으로써 지금도 Nullable 타입을 보면 먼저 거부감부터 드는 것은 사실이다.

피할 수 없으면 즐기라고 했던가. 우리가 애플리케이션을 개발하면서 적어도 내가 살아있는 동안은 Nullable 타입은 계속해서 존재할 것으로 생각한다. 솔직히 Null safety를 지원하지 않는 언어는 힘들 수 있겠지만 Null safety를 지원하는 언어를 사용한다면 Nullable 타입을 좀 더 적극적으로 수용하여 좀 더 우아하고(?) 효율적으로 처리하는 노력을 기울여보는 것은 어떨까.

## 참고

> [https://en.wikipedia.org/wiki/Null_object_pattern](https://en.wikipedia.org/wiki/Null_object_pattern){: target="\_blank" }

> [https://johngrib.github.io/wiki/pattern/null-object/](https://johngrib.github.io/wiki/pattern/null-object/){: target="\_blank" }
