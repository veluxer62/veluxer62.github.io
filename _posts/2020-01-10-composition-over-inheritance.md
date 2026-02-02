---
layout: single
title: "Composition over inheritance"
categories:
  - EXPLANATION
tags:
  - inheritance
  - composition
  - composition over inheritance
toc: true
toc_sticky: true
excerpt: 상속과 구성의 특징 및 장단점에 대해 알아본다.
---

> 상속보다는 구성을 활용한다.

[**Head First Design Patterns**](http://www.yes24.com/Product/Goods/1778966){: target="\_blank" }을 읽으면서 이 구문이 의미하는 바가 정확하게 와 닿지 않아서 상속과 구성이 어떤 것이며, 왜 상속보다는 구성을 활용하는것을 권장하는지 이해하고자 이 글을 작성한다.

## Inheritance(상속)

[Inheritance(이하 상속)](<https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)>){: target="\_blank" }은 **_Super Class(이하 부모 클래스)_**의 특징을 계승하여 **_Sub Class(이하 자식 클래스)_**를 도출해 Class 계층으로 형성하는 [개체 지향 프로그래밍](https://en.wikipedia.org/wiki/Object-oriented_programming){: target="\_blank" }의 한 방법이다.
`is-a`관계로 표현되기도 하며, **_자식 클래스_**는 **_부모 클래스_**가 가진 특징 이외에 새로운 추가 특징을 가질 수 있다.

### 예시 코드

<script src="https://gist.github.com/veluxer62/05b524c5a3cd6a09cd3e85b980d86a24.js"></script>

## Composition

[Object composition(이하 구성)](https://en.wikipedia.org/wiki/Object_composition){: target="\_blank" }은 개체나 데이터 타입을 더 복잡(complex)한 것으로 결합하는 방법을 말한다. `has-a`관계로 표현되기도 하며 **_A Class_**가 **_B Class_**포함하는 경우 **_B Class_**는 **_A Class_** 없이 존재할 수 없는 경우를 말한다.

UML로 나타내면 아래 그림과 같다.

[![Composition](/assets/images/composition.png)](http://www.nextree.co.kr/content/images/2016/09/--19-Composition1.png){: target="\_blank" }

### 예시코드

<script src="https://gist.github.com/veluxer62/62e8140ac39a4531cea9631ac97f5a71.js"></script>

## Composition over inheritance

상속보다 구성을 선호하는 것은 설계의 유연성을 높이는 설계 원칙이다. **_A Class_**와 **_B Class_**의 공통점을 찾아서 상속관계를 만드려고 하는 것보다 다양한 구성으로 개체를 구축하면 사소한 변경에 덜 취약해 질 수 있다. 이는 미래의 요구사항을 좀더 쉽게 수용할 수있다는 점에서 큰 장점을 가진다.

상속이 가진 단점은 구성으로 설계하도록 권장하는 또 다른 이유를 가진다.

1. 런타임에서 부모 클래스로부터 구현된 자식 클래스를 변경할 수 없다. (이 문제는 [Decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern){: target="\_blank" }을 통해 해결할 수 있다.)
2. 부모 클래스 세부 사항에 자식 클래스를 노출하므로 캡슐화를 깨뜨리는 경우가 많이 발생한다. (많은 개발 언어에서 `protected` 접근자를 제공함으로써 데이터 접근을 제어할 수 있다.)
3. 무분별한 상속의 재 사용은 코드를 이해하는데 어려움을 줄 수 있다. (이 문제는 개발자의 역량에 맡겨야 한다고 생각한다.)

물론 상속이 무조건 나쁘다는 것은 아니다. 구성 또한 아래와 같이 단점을 가지고 있다.

1. 상속을 건너뛰고 구성에만 중점을 두면 상속을 사용하는 경우 상속을 사용할때 필요하지 않았던 코드들을 작성해야 하는 경우가 종종 있다.
2. 가끔 자신을 반복하도록 강요되기도 하며 이는 [DRY 원칙](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself){: target="\_blank" }에 위배된다.
3. 구성에서는 위임이 자주 필요하며, 함수는 다른 코드 없이 위임된 개체의 다른 함수를 호출한다. 상속된 함수를 호출하는 것은 상속되지 않은 함수를 호출하는 것 만큼 빠르거나 약간 느릴 수 있지만 일반적으로 두번의 연속 함수 호출보다는 빠르다.

실제 시스템의 동작방식, 상황 및 구조적인 의미에 따라 상속으로 구현하고 구성으로 구현하는 방법을 선택해야 한다. 즉, **차이점을 이해하고 알맞게 사용**해야 한다.

`상속보다는 구성을 활용한다.`라는 말은 `상속으로 구현하기 전에 구성으로 구현한는 것을 먼저 고려해보자`라고 생각하면 더 좋을 것 같다.

> [https://stackoverflow.com/questions/2399544/difference-between-inheritance-and-composition](https://stackoverflow.com/questions/2399544/difference-between-inheritance-and-composition){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Inheritance\_(object-oriented_programming)](<https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)>){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Object_composition](https://en.wikipedia.org/wiki/Object_composition){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Composition_over_inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance){: target="\_blank" }
>
> [http://www.trytoprogram.com/cplusplus-programming/inheritance/](http://www.trytoprogram.com/cplusplus-programming/inheritance/){: target="\_blank" }
