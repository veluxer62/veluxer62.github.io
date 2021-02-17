---
layout: single
title: "Spring retry"
categories:
  - POST
---

# 도입 배경

대사 작업을 위해 나이스페이 API에 데이터 조회 요청을 보내는 기능을 구현 했었다.  대사 테스트 중 나이스페이 API에서 간헐 적으로 이유 없이 응답 실패로 인해 `RestTemplate` 객체가 예외를 반환 하였고, 그로인해 대사 결과를 제대로 얻을 수 없는 상황을 겪었다. 특별한 문제는 없었고 다시 요청하면 해당 문제는 해결 되었는데, 나이스페이 API에 데이터 조회 요청을 보내는 기능에서 재시도 기능을 추가하면 간헐 적으로 발생하는 이 문제의 빈도를 많이 줄일 수 있을 것이라 판단했다.

# 소개

Spring Retry는 Spring Projects의 서브 프로젝트로 오픈소스 라이브러리이다. 깃헙 경로를 참고하자.

[spring-projects/spring-retry](https://github.com/spring-projects/spring-retry)

스프링 빈 객체에 어노테이션 선언만으로 기존에 동작하는 함수에 재시도 기능을 추가하는 방식이다.

# 코드예제

## Dependency

```groovy
// https://mvnrepository.com/artifact/org.springframework.retry/spring-retry
compile group: 'org.springframework.retry', name: 'spring-retry', version: '1.3.0'
```

## Configuration

단순히 `@EnableRetry` 를 `@Configuration` 이 선언된 클래스에 추가해 주면 된다.

```java
@Configuration
@EnableRetry
public class Application {
}
```

## Usage

단순히 Bean 객체에 `@Retryable` 어노테이션을 선언해서 사용하면 된다.

```java
@Service
class Service {
    @Retryable(RemoteAccessException.class)
    public void service() {
        // ... do something
    }
    @Recover
    public void recover(RemoteAccessException e) {
       // ... panic
    }
}
```

`@Recover` 어노테이션은 필수는 아니고 선택 사항인데 만약 재시도 횟수를 초과해도 예외가 발생하는 경우 예외처리를 수행할 함수를 정의하는 것이다.

## Option

대표적인 옵션은 아래와 같다. 더 많은 옵션이 있지만 사용해 보지 않았으므로....레퍼런스를 참고하자.

[Retryable (Spring Retry 1.2.2.RELEASE API)](https://docs.spring.io/spring-retry/docs/api/current/org/springframework/retry/annotation/Retryable.html)

- maxAttempts

    최초 실패 횟수를 포함하는 최대 재시도 횟수를 정의한다. 기본 값은 3 이다.

- value

    재시도 기능이 동작하는 예외 클래스 타입 들을 정의한다. 기본 값은 비어 있다.

    만약 예외 클래스 타입을 `RuntimeException` 으로 정의 했다면 `RuntimeException` 이 발생한 케이스에서만 재시도 기능이 동작한다.

    아무 값도 지정하지 않고 (excludes 또한 지정하지 않았다면) 모든 예외에 대해 재시도 기능을 수행한다.

- backoff

    backoff 기능을 정의한다. 기본 값은 `@Backoff` 의 기본 값을 따른다. 자세한 내용은 링크 참고

    [Backoff (Spring Retry 1.2.2.RELEASE API)](https://docs.spring.io/spring-retry/docs/api/current/org/springframework/retry/annotation/Backoff.html)

# 장단점

## 장점

- 간단한 어노테이션 설정만으로 번거로운 재시도 기능을 구현할 수 있다.
- 다양한 부가 옵션 기능을 제공해 주므로 직접 구현할 때의 리소스를 많이 줄일 수 있다.
- 오픈소스로 높은 안정성을 보장한다.

## 단점

- Spring Bean에서 동작한다. Spring을 사용하지 않는 순수 자바 코드에서는 사용하기 어렵다.
- AOP 기반으로 동작하기 때문에 Bean의 public 함수에서 만 사용이 가능하다.(대안으로 [RetryTemplate](https://docs.spring.io/spring-batch/docs/current/reference/html/retry.html#retryTemplate) 사용을 검토해 볼 수 있다.)
