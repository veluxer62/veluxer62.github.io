---
layout: single
title: "Entity와 Value Object"
categories:
  - EXPLANATION
tags:
  - entity
  - value object
  - domain driven development

toc: true
toc_sticky: true
excerpt: 객체지향언어로 개발을 하다보면 우리는 수많은 클래스를 생성하고 그 클래스를 사용한다. 도메인 주도 설계 책을 읽으면서 우리 만드는 클래스들이 어떤 책임을 가지냐에 따라서 역할을 나눌 수 있다는 것을 배우게 되었고 그중에서 이 글에서는 Entity와 Value Object에 대해서 다루어 보고자 한다.
---

## Intro

객체지향언어로 개발을 하다보면 우리는 수많은 클래스를 생성하고 그 클래스를 사용한다. [도메인 주도 설계](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788992939850){: target="\_blank" } 책을 읽으면서 우리 만드는 클래스들이 어떤 책임을 가지냐에 따라서 역할을 나눌 수 있다는 것을 배우게 되었고 그중에서 이 글에서는 Entity와 Value Object에 대해서 다루어 보고자 한다.

## Entity

우리말로 참조 객체라고 해석되기도 하며 식별성과 연속성을 가진 객체를 말한다.

### 식별성

식별성을 가진 객체란 식별자를 가진 객체라고 할 수 있는데 예를 들면 아래와 같다.

- 사람 : 주민등록번호가 대표적인 식별자이다.
- 통장 : 통장번호가 대표적인 식별자이다.
- 택배 : 송장번호가 대표적인 식별자이다.

### 연속성

Entity는 자신의 생명주기 동안에 형태와 내용이 변경될 수 있다. 사람을 예로들면 이름이나 성별(?), 출생지 등은 생애주기 동안에 언제든지 바뀔 수 있다. 하지만 주민등록번호는 한번 생성되고 나면 변경되지 않는다. 여기에서 이름이나 성별, 출생지 등이 변경되어도 동일한 사람임을 추적할 수 있는 성질이 연속성이다.

> 물론 주민등록번호도 바꿀 수 있는 걸로 알고 있다. 하지만 우리가 사용하는 많은 전산 프로그램에서는 주민등록번호를 사람의 식별자 처럼 사용하고 있다. 그래서 실제로 주민등록번호를 바꾸게 되면 전산상으로 바꿔야 할게 어마어마하게 많다.

**그럼 Entity의 동일성을 어떻게 확인할까?**

객체지향언어에서는 모든 객체에 `equals`가 내장되어 있다. 기본적으로 두 객체의 메모리 주소가 같은지를 확인하는데, 객체 인스턴스 관점에서는 이 메모리 주소가 식별자 역할을 한다고 보면 된다. 하지만 웹 프로그래밍을 할때 우리는 하나의 서버만 사용하지 않고 Entity 객체는 데이터베이스에 영속화 되었다가 필요 시 다시 메모리로 가져오는 행위를 한다. 그렇기 때문에 메모리 주소를 식별자로 사용할 수는 없다. 그래서 주로 사용하는 식별자가 데이터베이스의 Primary Key이다. RDBMS나 MongoDB와 같은 데이터베이스는 기본적으로 Primary Key를 제공해주는데 성능적인 이유로도 식별자로 많이들 사용하고 있다. 하지만 Entity의 식별자를 단순히 DB의 PK로만 생각하지 않았으면 좋겠다. 위에서 말한 바와 같이 주민등록번호나 통장번호, 송장번호등이 식별자가 될 수 있음을 꼭 인지하고 선택의 폭을 넓혀두었으면 좋겠다.

### 모델링

커머스 도메인을 하나의 예로 들어서 얘기해 보겠다. 키트는 상품번호를 식별자로 가지고 있고 스토어의 식별자를 가짐으로써 연관관계를 가지고 있다.

![entity-model](/assets/images/posts/about-entity-and-value-object/entity-model.png)

그림에서 보면 키트와 스토어는 각자 Entity이며 각자의 생명주기를 가진다. 그렇기 때문에 키트는 스토어의 식별자 정보만 가지고 있고 해당 정보를 이용하여 스토어 정보를 얻을 수 있다. RDBMS에서는 키트 테이블이 스토어 식별자를 가지고 있으면서 Join문을 통해 정보를 조회하고 MongoDB에서는 Aggrigate를 통해 정보를 조회하는 방식을 예로 들 수 있을 것 같다.

## Value Object

우리말로 번역하면 값 객체라고 부를 수 있을 것 같다. 개념적인 식별성이 없이 도메인의 서술적 특면만을 나타내는 객체를 얘기한다. 이렇게 정의하면 너무 어려운 말 같아서 책에 적힌 인용문을 적어보겠다.

> 한 아이가 그림을 그릴 때 그 아이는 자기가 고른 펜의 색깔과 펜촉의 두께에 관심이 있을지도 모른다. 그러나 색과 모양이 같은 펜이 두 자루 있다면 아이는 아마 둘 중 어느 것을 사용하고 있는지 신경쓰지 않을 것이다. 펜을 잃어버려서 펜 꾸러미에서 색깔이 같은 펜을 꺼내 바꿔놓더라도 아이는 펜이 바뀌었는지는 개의치 않고 계속해서 그림을 그릴 것이다.

인용문에서 볼 수 있듯이 Value Object는 객체의 식별성은 중요하지 않고 속성의 값이 중요하다. 속성 값이 동일하다면 동일한 객체로 인지하는 것이다. 그래서 대부분의 경우 Value Object는 식별자를 필요로 하지 않는다. 어찌보면 Value Object가 Data Transfer Object처럼 보일 수도 있다. 실제로 비슷하게 사용되기도 하다. 하지만 엄밀히 따지면 두 객체가 지향하는 바는 다르다. 이 부분에 대해서는 논란의 여지가 많고 현재 얘기하고자 하는 논지에서 벗어나므로 넘어가자. 관련 내용이 궁금하다면 [VO와 DTO를 비교해 놓은 좋은 글](https://github.com/benelog/blog/issues/27){: target="\_blank" }이 있어 해당 글을 읽어보면 좋을 것 같다.

Value Object는 `equals`를 메모리의 주소나 식별자로 하는 것이 아니라 객체가 가진 모든 속성값으로 정의한다.

### 모델링

위에서 든 예시를 바꿔서 다시 들어보겠다. 키트는 가격이란느 속성을 가지고 있다. 여기에서 가격은 돈(Money)를 얘기하며 돈은 금액, 통화, 과세정보등의 속성값을 가지고 있다.

![value-object-model](/assets/images/posts/about-entity-and-value-object/value-object-model.png)

그림에서 키트는 Entity이며 가격은 키트가 가진 하나의 Value Object이다. 그래서 키트는 상품정보라는 식별자를 가지고 있지만 가격은 식별자를 가지고 있지 않다. RDBMS는 구조상 Value Object를 표현하는게 쉽지 않다. 그래서 Flat하게 나열해서 표현하거나 식별자가 없는 별도의 테이블로 표현하기도 한다. MongoDB는 Value Object를 표현하기 아주 쉬운데 그냥 Collection 내에 하나의 속성값으로 표현하면 된다.

## Comparison

Entity와 Value Object의 차이점에 대해서 좀더 자세히 얘기해 보자. 이 차이점을 알면 우리가 클래스를 정의할 때 Entity로 역할을 부여해야 할지 Value Object로 역할을 부여해야 할지 결정하는데 많은 도움이 될것이라 생각한다.

### 식별자

Entity는 반드시 식별자를 가지고 있으며 이 식별자를 통해서 동일한 객체임을 알 수 있다. 반면 Value Object는 대부분의 경우에 식별자를 가지고 있지 않으며 객체가 가진 속성을 통해서 동일한 객체임을 알 수 있다.

### 불변성

Entity는 연속성을 가지고 있다. 그래서 식별자는 불편하게 하고 나머지 속성에 대해서는 변경 가능하도록 설계한다. 반면 Value Object는 특별한 경우가 아니라면 모든 속성이 불변성을 가지도록 하는 것이 좋다. 만약 Value Object를 변경하고자 한다면 새로운 객체를 생성한 후에 교체하는 방식으로 사용한다.

Value Object의 속성이 변경가능하도록 하는 경우는 아래와 같은 상황에서 고려할 수 있다.

- 값이 자주 변경되는 경우
- 객체 생성이나 삭제에 비용이 많이 드는 경우
- 교체로 인해 클러스터링이 제한되는 경우
- value를 공유할 일이 많지 않은 경우

Value Object는 언제든지 공유될 수 있다. 이때 Value Object의 값이 변경 가능하도록 설계되어 있다면 의도하지 않는 오류가 발생할 가능성이 높고 기능 구현의 복잡도가 증가한다. 특히 웹 어플리케이션을 개발하는 경우 멀티 쓰레드로 인한 부작용을 고려해야 하는데 이런경우 불변 속성으로 객체를 유지한다면 코드의 단순함을 유지할 수 있을 것이다.

### 함수

Entity는 속성이 변경가능하기 때문에 명령형의 단어를 선택하면 좋다. 예를 들어 변경자 함수는 `changeTitle`, `increasePrice` 등등 명령형으로 사용하고 반환타입은 없도록 하고 조회 함수는 `getTitle`, `getTaxablePrice`등등의 함수명으로 사용한다. (조회 함수는 되도록 null safe하게 구현해야 클라이언트 코드가 복잡해 지지 않는다.)

Value Object는 불변 속성을 가지기 때문에 서술형 단어를 선택하면 좋다. 예를 들어 위치를 나타내는 객체의 X 좌표를 왼족으로 옮기는 변경자 함수라면 `toTheLeft`라고 적을 수 있다. 서술형이라는 표현이 좀 어렵긴하다. [오브젝트 디자인 스타일 가이드](http://www.yes24.com/Product/Goods/91167539){: target="\_blank" }책에서 말하는 서술형 함수 이름을 짓기 위해서는 `이 ~을 원한다. 하지만 ...`이라는 템플릿을 채워보면 함수명을 정할 때 도움이 된다고 말하고 있다. (이걸 봐도 솔직히 어렵다.) 

Value Object의 변경자 함수의 특징은 명령형 함수이지만 반환타입이 있다는 것이다. 왜냐면 Value Object는 불변 속성을 가지고 있으므로 보통 값을 변경할 때 새로운 객체를 생성하여 값을 변경한다. 그래서 변경된 자기 자신을 반환해야 클라이언트 코드가 변경된 객체를 사용할 수 있기 때문이다.

```kotlin
class Position {
	// ...
	fun toTheLeft(x: Long): Position {
		// ...
	}
}
```

### ORM

스프링 프레임워크를 사용한다면 ORM으로 많이들 JPA를 사용할 것이다. 특정 라이브러리로 예를 드는게 좋진 않아 보이지만 내가 잘 설명할 수 있는 것이 JPA라 이걸로 설명해 보겠다. 다만 JPA를 얘기하는 글이 아니므로 자세한 설명은 생략하겠다.

JPA에서 Entity를 다루는 것은 아주 쉽다. 아래와 같이 단순히 Model에 `@Entity` 어노테이션만 추가하면 된다.

```kotlin
@Entity
class Kit {
	@Id
	val id: Long = 0

	// ...
}
```

Entity 클래스는 반드시 `@Id` 어노테이션이 선언된 속성이 존재해야 한다. 위에서 설명한 Entity에 식별자가 반드시 존재해야 하는 이유를 생각해보면 이해가 될 것이다.

Entity간에 연관관계를 맺어주는 방법은 `OneToOne`, `OneToMany`, `ManyToMany`와 같은 어노테이션을 선언하여 관계를 맺어주면 된다.

```kotlin
@Entity
class Kit {
	@Id
	val id: Long = 0

	@OneToOne
	var store: Store? = null

	// ...
}
```

```kotlin
@Entity
class Store {
	@Id
	val id: Long = 0

	// ...
}
```

<br/>

Value Objec를 표현하기 위해서는 `Embedded`나 `ElementCollection`을 사용하면 된다. `@Embedable`이 선언된 클래스를 보면 ID가 없는 것을 볼 수 있다.

```kotlin
@Entity
class Kit {
	@Id
	val id: Long = 0

	@OneToOne
	var store: Store? = null

	@Embedded
	var price: Money = Money()

	@ElementCollection
	var history: MutableList<ChangeHistory> = mutableListOf()
}
```

```kotlin
@Embedable
class Money {
	var quantity: Long
	var currency: Currency
	var taxType: TaxType
}
```

```kotlin
@Embedable
class ChangeHistory {
	var timestamp: ZonedDateTime,
	// ...
}
```

JPA를 사용할때 많은 사람들이 Value Object를 Entity로 정의해서 사용하는 것을 보았었다. 그래서 불필요한 식별자 값이 존재하게 되고 불필요한 코드들이 도메인 모델의 복잡도를 증가시키는 것을 보았었다. 나는 Embedable 클래스로 Value Object를 표현해 주는게 좀 더 좋다고 생각한다. 이 글에서 JPA를 얘기한 이유이기도 하다.

## Wrap up

이미 다들 잘 알고 있겠지만 도메인 모델의 정보는 풍부해야 하며 모호하지 않아야 한다. 객체의 정보가 풍부해질수록 복잡도는 그만큼 증가하기 때문이다. 그래서 Entity와 Value Object를 잘 구분해서 역할만 잘 나누어놔도 우리가 도메인을 이해하고 표현하는데 아주 큰 이점을 누릴 수 있을 것이라 생각한다. Entity와 Value Object를 구분짓는게 쉽지 않을 수도 있다. 꾸준히 나누어보는 연습을 하고 실수를 하면서 경험치를 쌓아가보자.
