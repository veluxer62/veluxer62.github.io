---
layout: single
title: "Spring Bean Scope"
categories:
  - REFERENCE
tags:
  - scope
  - singleton
  - prototype
  - request
  - session
  - application
  - websocket
toc: true
toc_sticky: true
excerpt: 빈을 정의할때, 스프링에서는 사용자가 생성한 빈을 어떠한 종속성을 가지도록 하거나 생성되는 객체의 범위를 지정하는 등을 제어할 수 있는 기능을 제공해준다. 이 글에서는 스프링에서 설정할 수 있는 Scope가 어떤 것들이 있으며 각 Scope 별 수행하는 역할에 대해서 적어보고자 한다.
---

## Intro

빈을 정의할때, 스프링에서는 사용자가 생성한 빈을 어떠한 종속성을 가지도록 하거나 생성되는 객체의 범위를 지정하는 등을 제어할 수 있는 기능을 제공해준다. 이 글에서는 스프링에서 설정할 수 있는 Scope가 어떤 것들이 있으며 각 Scope 별 수행하는 역할에 대해서 적어보고자 한다.

## Singleton

별도의 설정이 없으면 기본으로 채택되는 Scope로 각 Spring IoC Container 별로 단일 객체를 생성하고 유지한다.

[![singleton](/assets/images/posts/spring-bean-scope/singleton.png)](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/singleton.png){: target="\_blank" }

Spring Bean Scope에서 말하는 Singleton은 GoF의 디자인패턴에서 말하는 Singleton과 비슷해 보이지만 엄연히 따지면 다르다. GoF의 디자인페턴에서의 Singleton은 각 ClassLoader마다 단일 클래스 인스턴스가 생성되는 것이지만, Spring Bean Socpe에서의 Singleton은 컨테이너와 빈 마다 단일 인스턴스를 생성하는 것을 의미한다.

코드로 표현하면 아래와 같다.

```kotlin
// Scope를 별도로 지정하지 않아도 기본적으로 Singleton으로 생성된다.
@Bean
fun accountService() = AccountService()

@Bean
@Scope(BeanDefinition.SCOPE_SINGLETON)
fun accountService() = AccountService()
```

## Prototype

빈의 요청이 있을 때마다 새로운 인스턴스를 생성한다. 스프링에서는 상태를 가지는 빈의 경우 Prototype을, 상태가 없는 빈의 경우 Singletone을 사용하는 것을 원칙으로 한다.

[![prototype](/assets/images/posts/spring-bean-scope/prototype.png)](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/prototype.png){: target="\_blank" }

다른 scope들과 다르게 Spring은 Prototype Bean에 대한 라이프사이클을 완전히 관리하지 않는다. 그래서 Prototype Bean을 사용하는 코드는 반드시 해당 빈을 사용한 후 해체 또는 정리하는 작업을 해야 한다.

코드로 표현하면 아래와 같다.

```kotlin
@Bean
@Scope(BeanDefinition.SCOPE_PROTOTYPE)
fun accountService() = AccountService()
```

## Request

Spring Container는 각 HTTP request에 대해 새로운 인스턴스를 생성하고 유지한다. 해당 인스턴스는 request에 대한 처리가 완료되면 폐기된다. 웹과 관련한 Spring ApplicationContext 구현에서만 사용할 수 있다.

코드로 표현하면 아래와 같다.

```kotlin
@Bean
@Scope(WebApplicationContext.SCOPE_REQUEST)
fun loginAction() = LoginAction()

@Component
@RequestScope
class LoginAction {
    //...
}
```

## Session

Spring Container는 각 HTTP session의 수명기간 동안 새로운 인스턴스를 생성하고 유지한다. 해당 인스턴스는 Session이 폐기되면 Bean도 함께 폐기된다. 웹과 관련한 Spring ApplicationContext 구현에서만 사용할 수 있다.

코드로 표현하면 아래와 같다.

```kotlin
@Bean
@Scope(WebApplicationContext.SCOPE_SESSION)
fun userPreference() = UserPreference()

@Component
@SessionScope
class UserPreference {
    //...
}
```

## Application

Spring Container는 전체 Web Application에 대해 한번 새로운 인스턴스를 생성하고 유지한다. 이는 Singletone과 매우 유사한 형태를 보이나 아래와 같은 차이를 가진다.

1. Singleton은 각 `ServletContext`별로 Bean을 생성하지만, Application Scope는 `ApplicationContext`별로 Bean을 생성한다.
2. Application Scope는 실제로 노출되기 때문에 `ServletContext`의 속성에서 볼 수 있다.

코드로 표현하면 아래와 같다.

```kotlin
@Bean
@Scope(WebApplicationContext.SCOPE_APPLICATION)
fun appPreference() = AppPreference()

@Component
@ApplicationScope
class AppPreference {
    //...
}
```

## Wrap Up

지금까지 Spring에서 다루는 Bean에 대한 Scope에 대해 알아보았다. Scope를 어떻게 정의하느냐에 따라 우리가 처리해야 할 비지니스 로직이 달라 질 수 있다. 기본 Scope가 Singleton이기 때문에 아무생각 없이 State를 클래스 내에 정의하여 사용하는 경우 동시성 문제가 발생하거나, Prototype으로 Bean을 설정하여 불필요한 자원이 사용되어 서버의 성능에 영향을 미치는 등 많은 문제점을 발생시킬 수 있기 때문에 Scope를 정확히 알고 활용할 수 있어야 좋은 어플리케이션을 만들 수 있을 것이다. 위 내용 외에 좀 더 자세한 내용을 알고 싶다면 아래에 참고 링크를 보길 바란다.

> [https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-scopes](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-scopes){: target="\_blank" }
