---
layout: single
title: "2020년 11월 1주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

지금까지 작업해오던 Inventory 작업이 중단될 뻔했다. 현재 급하게 처리해야 될일들이 많이 있고 현재 인력이 부족한 상황에서 언제 마무리 될 지 모를 작업을 중단할지 고민하는 부분에 대해서는 아쉬운 마음이 들지만 PO의 입장에서 그럴 수 있다고 생각했다. 결론적으로 다행히 운영 부서에서 Inventory의 중요성이 높다고 얘기해 주었고 진행하던 작업을 멈추지 않고 계속 할 수 있게 되었다. 작업 중단을 고민하는 하루동안 이런저런 생각이 들었는데 한편으로는 프로젝트를 작업하던 압박감에서 해방된다는 생각과 섭섭한 감정이 복합적으로 들었던 것 같다. 이번 경험을 계기로 프로젝트의 퀄리티도 중요하지만 작업에 대한 속도도 어느정도 받쳐줘야 우선순위에 밀리지 않고 계속해서 작업할 수 있을 것 같다 라는 생각을 하게 되었다. 코드와 타협하지 않으면서 최대한 아웃풋을 빠르게 낼 수 있도록 경험을 쌓아야 겠다.

아래는 내가 전주동안 배우거나 겪었던 이슈를 정리한 글이다.

## Mac에서 `idea` 커멘드를 실행하는 방법

Intellij를 설치하면 기본적으로 idea 명령어를 사용할 수 있다고 생각했었는데, 따로 설정을 해주지 않으면 사용할 수 없었다. 설정하는 방법은 아래와 같다.

`tools` -> `Create command line launcher`를 선택한다. 그런다음 아래 사진과 같이 원하는 디렉토리를 지정한 후 `OK`버튼을 클릭하여 저장하면 `idea`명령어를 사용할 수 있다.

![create-launcher-script](/assets/images/retrospective/create-launcher-script.png)

## JPA `MappedSuperclass`

JPA에서 공통 속성을 정의하기 위해서 부모 클래스를 생성한 후 Entity들이 해당 클래스를 상속 받도록 할 수 있다. 대부분 감사정보를 저장하기 위해 등록일시와 수정일시 등등을 저장하는 용도로 사용하는데, 이때 부모 클래스가 Entity로 인식되지 않도록 하기 위해서는 `@MappedSuperclass` 어노테이션을 사용해 주어야 한다.

사람마다 스타일이 있겟지만 개인적으로는 상속을 통해 공통 속성을 정의하는 것보다는 감사로그용 Embedded 클래스를 만들어서 감사로그를 사용하고자 하는 Entity의 속성에 정의하는 것이 더 나아 보인다. 원할 경우 속성값을 변경할 수도 있고 필요하지 않은 경우 빼 버릴 수도 있기 때문이다. 하지만 개별 클래스마다 `@EntityListeners(AuditingEntityListener::class)`를 넣어줘야 하는 불편함이 있겠지만 사용하고 싶지 않은 Entity가 존재하는 경우에는 오히려 부모 클래스의 상속을 받지 않아야 하기 때문에 또다른 문제를 가지고 있다고 생각한다.

## kotlin 1.4.10

얼마전에 코틀린 버전이 1.4.10 버전이 나와 있었다는것을 알게 되었다. (~~9월에 나왓지만 이제서야 알았다 ㅠㅠ~~) 버전 업데이트 구독을 해야 겠다.
변경 사항은 아래 링크를 참고하자.

[https://github.com/JetBrains/kotlin/releases/tag/v1.4.10](https://github.com/JetBrains/kotlin/releases/tag/v1.4.10){: target="\_blank" }

## Azure Pipelines

회사에서 Azure Devops를 도입하면서 PR시 build check를 Azure Pipelines로 변경하였다. Github에서는 Git Action으로만 가능할 줄 알았는데 Azure Pipelines를 이용할 수 있다는 것을 알게 되었다. 추후에 repository를 Azure git으로 옮길 예정이지만 한동안 이렇게 사용해봐야 겠다.

추후에 EKS에 배포하는 걸 적용 해보고 글을 적어봐야 겠다.

## Transaction Propagation

예전에 공부하면서 봤던 내용인데 최근에 과제검사하면서 사용하신 분이 계셔서 다시한번 정리해 본다.

- REQUIRED

부모 트랜잭션 내에서 실행하며 부모 트랜잭션이 없을 경우 새로운 트랜잭션을 생성

- REQUIRES_NEW

부모 트랜잭션을 무시하고 무조건 새로운 트랜잭션이 생성

- SUPPORT

부모 트랜잭션 내에서 실행하며 부모 트랜잭션이 없을 경우 nontransactionally로 실행

- MANDATORY

부모 트랜잭션 내에서 실행되며 부모 트랜잭션이 없을 경우 예외가 발생

- NOT_SUPPORT

nontransactionally로 실행되며 부모 트랜잭션이 존재한다면 예외가 발생

- NEVER

nontrasactionally로 실행되며 부모 트랜잭션이 존재한다면 예외가 발생

- NESTED

해당 메서드가 부모 트랜잭션에서 진행될 경우 별개로 커밋되거나 롤백될 수 있음. 둘러싼 트랜잭션이 없을 경우 REQUIRED와 동일하게 작동

## Kotlin `padStart`

`00001`와 같이 숫자 앞에 특정 문자열 패드를 붙이고자 하는경우 사용하기 좋은 함수가 `padStart`이다. 첫번째 매개변수는 길이이고 두번째 매개변수는 패드할 문자열이다.
[https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/pad-start.html](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/pad-start.html){: target="\_blank" }

## sealed class

특정 클래스를 열거형처럼 사용하고자 하는 경우 sealed class를 사용할 수 있다. sealed class는 클래스를 추상화 하고 타입별로 Factory를 만드는 경우에 유용하게 사용할 수 있다. 타입에 따라 구체 클래스를 생성해야 하는 경우 if문이나 when절을 사용하게 되는데 특히 when절을 사용하는 경우 sealed class의 타입이 추가될 때마다 when절에 조건을 추가하지 않으면 컴파일 시점에서 오류가 발생하도록 강제할 수 있기 때문에 interface나 일반적인 부모 클래스를 사용하는 것보다 유용할 수 있다.

[https://kotlinlang.org/docs/reference/sealed-classes.html](https://kotlinlang.org/docs/reference/sealed-classes.html){: target="\_blank" }

## Spring에서 비동기 Event Listener 사용 시 주의점

비동기로 Event Listener를 정의해서 사용하는 경우 Transaction을 분리할 수 있다. 그런 경우 아래와 같이 사용할 수 있는데

```
@Async
@TransactionalEventListener
```

이럴 경우 Trasaction이 Listener에도 적용이 되어서 데이터 일관성을 보장해 줄거라 생각했는데 그렇지 않았다... 그래서 아래와 같이 `@Transactional` 어노테이션을 추가해주어야 한다.

```
@Async
@TransactionalEventListener
@Transactional
```

비동기로 동작하는 경우 Transaction 전파가 되지 않는가 보다. 방법이 있는지는 좀 찾아봐야겠다!

## Test Containers

로컬환경에서 개발할 때에나 테스트 환경을 만들 때 PC에 필요한 리소스(DB, Queue 등등)를 설치하여 사용하기도 하는데 Docker를 사용하면 손쉽게 서비스 환경을 조성할 수 있으면서 PC에서 소비되는 리소스를 줄일 수 있다. 다만 직접 Docker를 실행해 주거나 Docker Compose를 만들어서 해주는 등 부가적인 작업을 해주어야 하는데 이렇게 할 필요 없이 Spring에서 Docker를 실행 시켜서 개발환경과 테스트환경에서 서비스를 실행해 볼 수 있도록 도와 주는 도구가 [Test Containers](https://www.testcontainers.org/){: target="\_blank" } 이다.

전주에 Test용 DB(H2)와 개발환경에서의 DB(MariaDB)의 차이로 인해 Native 쿼리를 실행하면 문법 차이로 테스트가 실패하는 이슈가 있었는데, Test Containers를 사용하면 해결 할 수 있을 것 같아 처음에 도입을 고려했었다. 결론적으로는 실행 시간이 길어진다는 문제점과 다른 방식으로 해결할 수 있어서 도입하지 않았지만 한번 튜토리얼을 해보면 나중에 도움이 많이 될 것 같다.

참고 블로그 : [https://velog.io/@lsb156/Spring-Boot-TestContainers](https://velog.io/@lsb156/Spring-Boot-TestContainers){: target="\_blank" }

## LocalStack

이것도 Test Containers와 비슷한 개념인데, 이 라이브러리는 AWS에서 사용할 수 있는 리소스를 로컬환경에서 실행 및 테스트를 할 수 있게 도와준다는 큰 이점을 가져다 준다. 현재는 AWS 리소스를 직접 사용할 일이 없어 도입하지 않았는데, 회사 동료분게서 도입해보고 크게 만족스러워 하셔서 추후 필요하면 바로 도입해야 겠다. 그전에 튜토리얼도 한번 해봐야지!

참고 블로그 : [https://woowabros.github.io/tools/2019/07/18/localstack-integration.html](https://woowabros.github.io/tools/2019/07/18/localstack-integration.html){: target="\_blank" }

현재 버전과 조금 달라 코드는 본인의 버전에 맞게 바꿔야 한다.

