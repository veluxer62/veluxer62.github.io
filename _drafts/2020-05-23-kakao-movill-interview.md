---
layout: single
title: "카카오모빌 1차 면접 준비"
categories:
  - INTERVIEW
tags:
  - 카카오모빌
---

## 자기소개

안녕하세요. 이번에 모빌에 개발자로 입사지원한 남경호 입니다. 저는 지난 개발경력 동안 API 서버, Back-Office 서버, Batch 서버등 백앤드 서버개발에 대한 개발 경험을 가지고 있습니다.

처음 재직한 닉스테크라는 회사는 보안관련 솔루션을 판매하는 회사였습니다. 그곳에서 보안관제용 Back-Office 기능개발과 대시보드, 배치서비스, 보고서 및 Splunk, Archsight, Qradar와 같은 SIEM 장비 연동과 관련한 기능 개발을 담당하였습니다. 주로 데이터 연동과 저장된 데이터를 시각화 하는 부분을 집중적으로 개발하였습니다.

현재 재직중인 엑스트라이버라는 회사는 트립스토어라는 여행패키지 상품 가격 비교 플랫폼 서비스를 운영하고 있는 회사입니다. 주로 클라이언트를 위한 API 개발과 Back-office 및 배치 서비스 개발을 담당하였고, 현재는 예약파트에서 B2B 사이트를 위한 예약 관련 기능 개발을 담당하고 있습니다.
엑스트라이버에서는 API의 속도 및 다량의 데이터를 어떻게 다룰지, 어떻게 API를 설계하면 좋은지를 고민하며 개발을 진행하고 있습니다.

저는 모빌에서 제가 가진 이러한 역량을 잘 발휘하여 서비스를 발전시킬 수 있는 개발자가 될 수 있다고 자신합니다.

## 지원동기

이곳에서의 젊은 조직 문화와 전망이 좋은 비지니스 모델, 그리고 서비스를 함께 발전시켜 나가면서 함께 성장해 갈 수 있을 것이라는 기대감을 가지고 지원하게 되었습니다.

## 이직사유

저의 백엔드 개발 기술에 대한 전문성을 더 발전 시키고 싶어서 이직을 결심하게 되었습니다. 개인적으로 부족한 기술에 대한 공부를 하면서 이런 부분을 채워가려고 노력하고 있지만 혼자서는 부족한 부분이 많았습니다. 모빌에는 유능하신 시니어 백엔드 개발자 분들이 계시고 서비스 발전에 기여하면서 저의 전문성을 발전 시키고 싶었습니다.

- 회사에는 유능한 시니어 개발자가 없나?

  물론 현재 트립스토어에도 유능한 시니어 개발자 분들이 계십니다. 하지만 백엔드 개발언어 및 프레임워크가 나눠지면서 제가 가진 궁금증에 대한 전문적인 조언 또는 해결책을 얻기가 쉽지 않았습니다.

### 사용해 본 디자인 패턴에 대해서 설명해 보라.

- Dependency Injection

  구성요소간의 의존 관계가 소스코드 내부가 아닌 외부의 설정파일 등을 통해 정의되게 하는 디자인 패턴 중의 하나입니다.
  의존성 주입과 항상 함께 나오는 단어가 inversion of control입니다. 의존 주입 시 구현체가 아닌 인터페이스를 주입한다면 종속성을 감소시키고 재사용성을 높일 수 있습니다. 테스트 코드를 작성하기에도 좋습니다.

  ![dependency-injection-pattern](/assets/images/draft/dependency-injection-pattern.jpg)

- Decorator

  주어진 상황 및 용도에 따라 어떤 객체에 책임을 덧붙이는 패턴입니다. 외부 인터페이스의 변경 없이 기능을 추가하는 경우에 유용하다.

  ![decorator-pattern](/assets/images/draft/decorator-pattern.png)

- Abstract Factory

  구체적인 클래스에 의존하지 않고 서로 연관되거나 의존적인 객체들의 조합을 만드는 인터페이스, 관련성 있는 여러 객체를 일관된 방식으로 생성하는 경우에 유용하다.

  ![abstract-factory-pattern](/assets/images/draft/abstract-factory-pattern.png)

- Factory Method

  Factory method는 부모(상위) 클래스에 알려지지 않은 구체 클래스를 생성하는 패턴이며. 자식(하위) 클래스가 어떤 객체를 생성할지를 결정하도록 하는 패턴이기도 하다. 부모(상위) 클래스 코드에 구체 클래스 이름을 감추기 위한 방법으로도 사용한다.

  ![factory-method-pattern](/assets/images/draft/factory-method-pattern.png)

- Command

  실행될 기능을 캡슐화함으로써 주어진 여러 기능을 실행할 수 있는 재사용성이 높은 클래스를 설계하는 패턴, 이벤트가 발생하였을 때 실행될 기능이 다양하면서도 변경이 필요한 경우 이벤트를 발생시키는 클래스를 변경하지 않고 재사용하고자 하는 경우 유용하다.

  ![command-pattern](/assets/images/draft/command-pattern.png)

- Observer

  한 객체의 상태 변화에 따라 다른 객체의 상태도 연동이 되도록 일대다 객체 의존관게를 구성하는 패턴, 데이터의 변경이 발생하는 경우 상대 객체에게 의존하지 않으면서 데이터 변경을 통보하고자 할때 유용하다.

  ![observer-pattern](/assets/images/draft/observer-pattern.png)

- Template Method

  어떤 작업을 처리하는 일부분을 서브 클래스로 캡슐화해서 전체 일을 수행하는 구조는 바꾸지 않으면서 특정 단계에서 수행하는 내역을 바꾸는 패턴, 코드의 중복을 최소화 할때 유용하다.

  ![template-method-pattern](/assets/images/draft/template-method-pattern.png)

- Composite

  객체들의 관계를 트리 구조로 구성하여 부분-전체 계층을 표현하는 패턴으로, 사용자가 단일 객체와 복합 객체 모두 동일하게 다루도록 한다.

  ![composite-pattern](/assets/images/draft/composite-pattern.png)

- Stratege

  행위를 클래스로 캡슐화해 동적으로 행위를 자유롭게 바꿀 수 있는 패턴, 같은 문제를 해결하는 여러 알고리즘이 클래스별로 캡슐화 되어 있고 이들이 필요할 때 교체할 수 있도록 함으로써 동일한 문제를 다른 알고리즘으로 해결할 수 있도록 하는 디자인 패턴

  ![strategy-pattern](/assets/images/draft/strategy-pattern.png)

- Singleton

  전역 변수를 사용하지 않고 객체를 하나만 생성 하도록 하며, 생성된 객체를 어디에서든지 참조할 수 있도록 하는 패턴

  ![singleton-pattern](/assets/images/draft/singleton-pattern.png)

### String, StringBuilder, StringBuffer

- String

  문자열을 다루는 참조타입인 클래스(final class)이며 불변 개체이다. 그래서 새롭게 문자열이 생성될 때마다 메모리 공간이 변경되는 것이 아니라 새롭게 생성되므로 문자열 연산시 성능이 좋지 않다.
  문자열 연산이 적고 멀티쓰레드 환경에 사용하기 적합

- StringBuffer

  문자열 변화를 다룰때 사용하는 클래스이다. 동기화를 지원하기 때문에 Thread-safe하다. 대표적으로 `append`, `insert`, `toString`, `capacity`, `reserse`, `substring`, `delete` 등이 있다.

- StringBuilder

  문자열 변화를 다룰때 사용하는 클래스이다. 동기화를 지원하지 않기 때문에 멀티쓰레드 환경에 사용하기에 적합하지 못하다. 연산처리가 StringBuffer보다 빠르다. 대표적으로 `append`, `insert`, `toString`, `capacity`, `reserse`, `substring`, `delete` 등이 있다.

## Filter, Interceptor, AOP

사용 사례 부분 추가

Servlet Request <-> filter <-> DispatcherServlet <-> Interceptor <-> AOP <-> Controller

- Filter

  Web Application에 등록을 하고 DispatcherServlet 이전에 실행된다.
  스프링에서는 `web.xml`을 통하여 Filter를 등록하였지만 스프링부트에서는 `FilterRegistrationBean`을 `@Configuration`을 통해 등록하거나 `@WebServlet`, `@WebFilter`, `@webListener`를 선언한후 `@ServletComponentScan`을 통해 사용한다.
  대표적인 함수로 `init`, `doFilter`, `destroy`가 있다.

  주로 인코딩 처리, XSS 방어등을 위해 사용한다.

  XSS 방어를 위해서 `lucy-xss-servlet-filter`할 수 있으나 form-data만 지원하고 JSON은 지원하지 않는다. 그래서 JSON을 XSS 방어를 하기 위해서 인터셉터나 Convertor에서 처리해서 사용하기도 한다.
  좀더 단순하게 `X-XSS-Protection`를 HTTP 헤더에 설정하여 브라우저에서 실행이 되지 않도록 막는 방법도 있다.

  스프링 부트에서는 인코딩 설정방법이 두가지가 있다. 하나는 `FilterRegistrationBean`을 이용하여(`CharacterEncodingFilter`)를 등록하여 인코딩을 설정하는 방법, 다른하나는 `application.property`를 설정하여 필터를 적용하는 방법이다. `spring.http.encoding.charset=UTF-8, spring.http.encoding.enabled=true, spring.http.encoding.force=true`

  예외 발생시 스프링에서는 `web.xml`에서 `error-page` 설정을 통해 처리할 수 있었으나 스프링부트에서는 try catch를 통한 예외 처리가 필요하다.

- Interceptor

  Spring의 Context에 등록을 하고 DispatcherServlet과 Controller 사이에 실행된다. 주로 로그인 체크, 권한체크, 프로그램 실행시간 계산 등을 위해 사용한다. 예외처리는 @ControllerAdvice, @ExceptionHandler등을 이용하여 처리할 수 있다. 대표적인 함수 `preHandle`, `postHandle`, `afterCompletion`

  Interceptor를 등록하기 위해서는 `HandlerInterceptor` 또는 `HandlerInterceptorAdapter`를 구현한 다음 `WebMvcConfigurer` 설정에서 등록하여 사용한다.

- AOP

  Interceptor와 Controller 사이에 실행된다. 주로 로깅, 트렌젝션, 에러 처리 등을 위해 사용한다. AOP와 Interceptor의 큰 차이는 파라미터 차이인데 AOP는 JoinPoint나 ProceedingJoinPoint를 사용하는 반면 Interceptor는 HttpServletRequest, HttpServletResponse를 사용한다.

  `@Aspect` 어노테이션을 클래스에 선언하고 `@Pointcut`, `@before`, `@Around`, `@After` 어노테이션을 함수에 선언하여 사용한다. `@EnableAspectAutoProxy`를 Application클래스에 적용해야 동작한다.

## SOLID

- SRP(단일책임 원칙)
  한 클래스는 단 한가지의 변경 책임을 가지는 것

- OCP(개방-폐쇄 원칙)
  소프트웨어 개체는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 함

- LSP(리스코프 치환 원칙)
  자료형 S가 자료형 T의 하위형이라면 필요한 프로그램의 속성의 변경 없이 자료형 T의 객체를 자료형 S의 객체로 교체할 수 있어야 한다는 원칙

- ISP(인터페이스 분리 원칙)
  클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다는 원칙

- DIP(의존성 역전 원칙)
  상위수준의 모듈은 하위수준의 모듈에 의존해서는 안된다는 원칙

## 자바 7과 자바 8의 차이

- 자바 7

  - 제네릭 지원
  - Switch문에서 String 클래스 지원
  - 자동 리소스 자원 관리 (try() {} catch {} 를 사용하여 finally에서 close 함수를 사용하지 않아도 됨)
  - Multi-Catch 지원

- 자바 8

  - 람다 표현식 추가
  - Stream API 추가
  - java.time 패키기 제공
  - Optional 클래스 제공

## 스프링의 장점

- POJO 기반 구성

  스프링은 내부적으로 객체 간의 관계를 구성할 때 별도의 API 등을 사용하지 않는 POJO(Plain Old Java Object) 만으로 구성이 가능하도록 되어있다. 따라서, 일반적인 Java 코드를 이용하여 객체를 구성하는 방식 그대로 스프링에서 사용 가능하다.

- DI (Dependency Injection)

  스프링에서는 구조적으로 DI를 사용하도록 유도하고 있어 개발자가 의도적으로 설계하지 않아도 DI의 장점을 누릴 수 있다. (IoC, 테스팅 등)

- AOP

  스프링은 손쉽게 AOP 적용을 통해 개발자가 불필요한 반복작업을 줄이는 등 비지니스 로직에 집중 할 수 있도록 해준다.

- Transaction

  스프링은 손쉬운 Transaction 지원으로 개발자가 신뢰성 있는 Transaction 기능을 사용하여 비지니스 로직에 집중하고 안정적인 어플리케이션을 개발 할 수 있도록 도와준다.

- 다양한 라이브러리

  일반적인 공통 요구사항을 다룰 수 있는 다양한 라이브러리를 지원함으로써 개발자는 비지니스 요구사항의 구현에 맞는 신뢰도 높은 어플리케이션을 생산성이 높게 개발할 수 있다.

## 스프링의 단점

- 복잡하고 다양한 추상기능

  스프링에서 다양한 추상화된 기능을 제공한다. 이러한 추상화된 기능은 구현 시 동작에 대한 이해와 부작용에 대한 주의를 기울여야 하는데 스프링에서 제공하는 다양하고 복잡한 추상화된 기능들은 개발자들의 학습비용 요한다.

- 다양한 대안과 넓은 유연성

  유연성은 스프링에게 큰 장점이지만 처음 스프링을 접하는 개발자에게는 어떻게 구현을 해야할지 막막한 넓은 벽이 될 수도 있다.

- 무겁고 복잡한 프레임워크

  다양한 기능을 제공하는 만큼 복잡하고 무거운 개발환경을 제공한다. 최근 출시된 여러 프레임워크들은 단순한 설정을 통해 간단히 어플리케이션을 구동할 수 있고 스프링도 이런한 유행을 따라 스프링부트를 출시하면서 단순함을 추구하고 있다. 하지만 기본으로 제공하는 여러가지 설정 및 빈들은 어플리케이션이 기본적으로 무게감을 가지고 있을 수 밖에 없고 이런 무거움을 줄이고자 하려면 스프링에 대한 전반적인 지식을 요한다.

## 서비스(도메인)이란 무엇인가?

도메인이란 소프트웨어로 해결하고자 하는 문제 및 관심사를 말한다. 그래서 도메인 모델이란 이러한 문제와 관심사의 솔루션을 추상화한 것이라고 보면 된다.

## 스프링의 라이프 사이클

![spring-mvc-lifecycle](/assets/images/draft/spring-mvc-lifecycle.jpg)

1. 사용자로부터 요청이 들어오면 `Filter`를 거치게 된다.
2. `Filter`를 거쳐 전달된 사용자 요청은 `DispatcherServlet`으로 전달되고 `HandlerMapping`을 통해서 적절한 `Controller`를 찾아 `DispatcherServlet`에 전달한다.
3. `DispatcherServlet`은 `HandlerMapping`이 전달해준 `Controller`를 `HandlerAdapter`에게 전달한다.
4. `HandlerAdapter`는 `Controller`를 호출한다.
5. `Controller`는 전달된 요청을 처리한 후 결과를 Model에 담고 `View의 이름`을 `HandlerAdapter`에게 전달한다.
6. `HandlerAdapter`는 `View의 이름`을 `DispatcherServlet`에게 전달한다.
7. `DispatcherServlet`은 `ViewResolver`에게 `View의 이름`을 전달한다.
8. `ViewResolver`는 `View의 이름`을 가지고 매핑된 `View`를 찾아서 `DispatcherServlet`에 반환한다.
9. `DispatcherServlet`은 `View`를 `Filter`에 반환한다.
10. `Filter`는 `View`를 사용자에게 반환한다.

## 자바의 특징

- 객체지향언어
- 메모리를 자동으로 관리한다. (GC)
- 멀티쓰레드 구현이 쉽다.
- 높은 이식성 (OS와는 무관하게 동일 한 기능 보장 - JVM)
- 다양한 어플리케이션 구현 가능 (Console, UI, Server, Mobile)
- 풍부한 오픈소스 라이브러리

## 스프링 DI

객체 자체가 아니라 Framework에 의해 객체 의존성이 주입되는 설계 패턴. 설정에 명시된 대로 Container가 Bean 객체를 생성하고 종속성을 주입힌다.

주입 방법

- 생성자 주입
- 함수 주입(Setter)
- Field 주입(Autowired)

장점

- 종속성이 감소
- 재사용성 증가
- 테스트 용이성

## IoC Container

IoC 컨테이너란 Bean 펙토리라고도 불리며 객체의 생성, 생명주기의 관리까지 모든 객체에 대한 제어권을 가지고 있는 스프링 컨테이너를 말한다. IoC 컨테이너가 관리하는 객체들을 Bean이라고 한다.

빈을 등록하는 방법

- `application.xml`을 통해 등록
- `@Configuration`을 적용한 클래스 내에서 `@Bean`을 통해 등록
- `@ComponentScan`과 `@Component`를 통해 등록

## SPA 정의

SPA란 Single Page Application의 약자로 서버에서 요청에 대한 페이지를 완성한 후 클라이언트에 전달해주는 서버 사이드 랜더링과는 다르게 브라우저에서 최초 한번 페이지를 로드한 후 특정 부분만 서버에서 제공한 데이터를 기반으로 페이지를 구성하는 방식을 말한다.

대표적인 라이브러리/프레임워크는 React.js, Vue, Angular가 있다.

장점

- 하나하나 화면 전체를 랜더링할 필요가 없기 때문에 화면이동이 빠르다.
- 화면에 필요한 부분만 랜더링하기 때문에 처리과정이 효율적이다.
- 화면의 깜빡임이 없으므로 사용자 경험이 좋다.
- 컴포넌트별 개발이 용이하며 재사용성이 좋다.

단점

- SEO 최적화하기가 쉽지 않다. (SSR로 해결이 가능하다.)
- 처음 화면 로딩지 전체 화면이 미리 준비되어야 하기 때문에 로딩 시간이 길다. (code spliting을 통해 해결이 가능하다.)
- 어플리케이션을 구성하는데 드는 비용이 높다.
- 클라이언트 PC의 성능에 크게 영향을 받는다.
- 보안이 상대적으로 취약하다.

## HTTP란?

HTTP는 Hyper Text Transfer Protocol의 준말로 링크를 기반으로 데이터를 주고 받는 것을 말한다. 클라이언트(요청)와 서버(응답)가 데이터를 주고받기 위한 프로토콜이다. HTML 문서 뿐만 아니라, 이미지, 동영상, 텍스트 문서등을 주고받을 수 있다.

특징

- TCP/IP를 이용하는 응용 프로토콜
- 연결 상태를 유지하지 않는 비연결성 프로토콜(Cookie와 Sesison으로 해결)
- 요청/응답 방식으로 동작

## REST란?

REST(Representational State Transfer)는 월드 와이드 웹과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식을 말하며 로이필딩이 2000년 박사학위 논문에서 소개하였다.

특징

- 클라이언트/서버 구조 : 일관적인 인터페이스로 분리되어야 한다.
- 무상태 : 각 요청에 대한 클라이언트 컨텍스트는 서버에 저장되어서는 안된다.
- 캐시 처리 가능 : 클라이언트는 캐싱이 가능해야 한다.
- 계층화 : 클라이언트는 서버와 직접 연결되었는지 중간 서버를 통해 연결되었는지 알 수 없다.
- 인터페이스 일관성 : 아키텍처를 단순화시키고 작은단위로 분리하여 클라이언트-서버의 각 파트가 독립적으로 개선될 수 있도록 해준다.
- 자체 표현 기능 : 메시지 만으로 그 요청이 어떤 행위를 하는지 알 수 있다.

가이드

- URI는 정보의 자원을 표현해야 한다.
- 자원에 대한 행위는 HTTP Method로 표현한다.

레벨

- Level 0 : 웹의 기본 메커니즘을 전혀 사용하지 않는 단계 (단일 URI, 단일 Method)
- Level 1(Resource) : 모든 요청은 개별 리소스와 통신, 단일 Method 사용
- Level 2(HTTP Method) : URI + Method 조합으로 리소스를 구분, 응답 상태를 Http Status code로 활용. 가장 많으 REST API가 채택
- Level 3(Hypermedia Control) : HATEOAS를 준수하는 방식으로 자원에 대한 모든 링크를 제공하면서 서버와 상호작을 가능하도록 하는 것이다. 복잡성이 아주 높으며 현실적으로 구현이 쉽지 않다.

## 본인 스스로 평가

어딜가든 나를 필요로 하는 개발자라고 생각합니다. 프로젝트 참여 시 언제나 적극적으로 해당 프로젝트에 임하며 더 나은 기능을 제공하기 위해 도메인 지식을 습득하고 기술고민을 합니다. 또한 기술적으로 구현이 어렵거나 상황상 요구사항을 충족하기 어렵다면 동료들에게 조언을 구하거나 현실적인 다른 대안을 모색하는 등 일이 되도록 하기 위한 노력을 아끼지 않습니다. 주니어 개발자로써 부족한 부분이 많지만 이러한 저의 부족함을 메우기 위해 항상 공부하고 끊이없이 질문하고 있습니다. 이런 저의 역량을 바탕으로 모빌에서 함께 성장하고 발전하는 개발자가 되고자 합니다.

## 마지막 할말?

저는 기능을 개발할 때 되도록 많은 사람들과 논의를 통해서 요구사항을 정의하고 제가 담당한 기능에 대해서는 최대한 깊이 이해하려고 노력하고 사용자가 편리하게 사용할 수 있도록 기능을 구현하는데 집중을 하여왔습니다. 이러한 저의 노력은 현재 재직 중인 회사에서 사용중인 서비스에 대한 높은 이해력을 가질 수 있도록 하였습니다. 또한 저는 끊임없이 자기 개발을 위한 노력을 이어가고 있습니다. 저의 블로그를 보시면 아실 수 있으시겠지만 저는 제가 가진 개발 기술을 향상 시키고 부족한 부분을 채우기 위한 노력을 항상 하고 있으며 이러한 노력을 잊지 않기 위해서 회고나 블로그 작성을 통해서 기록을 하고 있습니다.

저의 이런 노력과 역량은 제가 지원한 모빌에서 요구하는 백엔드 개발자 역량에 아주 잘 맞는다고 생각하고 모빌의 구성원으로 맡은 바 업무를 잘 진행할 수 있을 것이라 생각합니다. 부디 모빌의 일원이 되어 함께 긍정적으로 크게 발전 할 기회를 주셨으면 합니다. 감사합니다.

## 질문?

- 장비, 툴 지원
- 개발 프로세스
- 사내 복지
