---
layout: single
title: "2022년 4월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2022-april-dev-log/
---

언어전환 작업이 끝나고 QA에 입고한 후 하루 연차를 내어 친구와 속초 라이딩을 하였다. 처음 계획은 일주일 정도 부산 라이딩을 가려고 하였지만 QA 입고 후 이슈 대응 시간도 필요했기에 긴 연차를 쓰기에 부담이 되기도 하였고 친구와 라이딩 합을 처음 맞춰 보는 건데 처음부터 너무 빡센 라이딩을 하는것은 리스크가 있다고 생각해서 속초로 목적지를 변경하였다. (결론부터 말하자면 부산부터 가지 않고 속초부터 간 선택은 잘한 선택이었다 ^^;) 총 2박 3일 일정으로 계획을 잡았고 첫날은 서울에서 홍천까지, 둘째날은 홍천에서 속초까지, 마지막날은 속초에서 버스를 타고 서울로 돌아오는 일정이었다. 하루에 약 100km씩 달리는 일정으로 계속 약 오르막이긴 하지만 첫 라이딩 여행코스로 괜찮았다. (미시령 고개는 너무 힘들긴 했다...) 다음에는 춘천 라이딩 여행을 당일 코드로 가봐야겠다!

약 3주간의 QA를 마치고 드디어 약 5개월간의 언어전환을 마무리 지었다. 예상보다 일정이 밀리긴 했지만 이 글을 쓰는 지금도 새롭게 배포된 서버가 잘 돌아가는게 신기할 정도로 전환작업이 잘 마무리 되었다. 전환 작업에 대한 자세한 내용은 [서버 언어 전환 이야기](https://veluxer62.github.io/reference/all-new-server/){: target="\_blank" }에 적어 두었다. 이 글을 적는날만 기다렸었는데 이 글을 적게 되다니 감회가 새로웠다. 계획과 개발 배포까지 프로젝트를 이끌면서 많은 것들을 배우고 느낄 수 있었고 앞으로 이런 경험을 다시 하기도 쉽기 않기 때문에 내 개발 커리어상 큰 획을 긋는 프로젝트로 기억될 것 같다.

바쁘게 달렸던 언어전환 프로젝트를 마치고 쉴틈없이 바로 다음 프로젝트가 시작되었다. 프로젝트 계획과 일정산정, 그리고 배포후 발생하는 추가적인 이슈 대응까지 바쁜나날이 지속되고 있다. 거기에 풀 재택근무가 공식적으로 종료되고 3일 출근 2일 선택 재택 근무체계로 바뀌게 되면서 팀원들의 업무 집중도도 많은 영향을 미치고 있는 듯 하다.

![control chart](/assets/images/retrospective/control-chart-between-02-and-04.png)

위 차트에서 볼 수 있듯이 언어전환 작업이 끝나는 4월부터 작업 생산성이 많이 떨어진 것을 볼 수 있다. 지금 진행하고 있는 프로젝트를 마무리하고 나면 회고를 통해서 원인분석과 개선방법을 모색해 보아야 겠다.

언어전환 작업이 끝나면서 다시 제품개발을 위한 프로젝트가 시작되었다. 언어전환 작업 이전에 프로젝트를 진행할 때와 달리 구성인원이 어느정도 바뀌었기 때문에 어쩌면 지금의 프로젝트가 팀원간에 합을 맞추는 첫번째 작업이기도 하다. 그러면서 팀내에서 Product Manager의 역할에 대한 이야기가 나오게 되었다. 사실 내가 겪었던 Product Manager는 회사마다 그 역할이 달랐다. 어떤 회사는 기획자와 같은 역할을 수행하기도 하였고, 어떤 회사는 Product Manager가 아닌 Product Owner라는 이름으로 불리기도 하였다. 아마 다른 회사들도 각자가 가진 환경에 맞게 그 역할을 가지고 있을 것이다. 아무래도 PM이 가지고 있는 역할이 워낙 광범위하다보니 회사마다 어느 역할에 집중하냐에 따라 수행하는 업무가 변하는것이지 않을까 싶다. 우리 팀은 Product Manager는 기획자가 아니기에 기획자가 산출해 내는 디테일한 기획서를 만들어내지 않으며 Project Manager가 아니기 때문에 프로젝트 관리에 역할을 집중하지 않는다고 한다. 대신에 지표를 이용하여 서비스의 개선점을 모색하고 의사결정에 도움을 주는데 좀 더 집중을 한다고 들었다. 사실 Product Manager가 Proejct Manager와 기획자의 역할을 겸한다고 생각을 했었다. 그러다보니 역할에 대한 오해가 좀 있었던것 같고 바라는 바가 실제 역할과 차이가 있었던것 같다. 지금와서 느끼는거지만 역할은 크게 중요하지 않을 수 있다라고 생각이 들기도 한다. 서비스가 잘 성장할 수 있도록 각자 유기적으로 잘 협력하면 되지 않을까 라는 생각이 든다. 이제 첫단추를 끼우기 시작했다. 다음 단추를 끼워가며 잘 조정해 나가봐야 겠다.

아래는 4월동안 정리한 이슈 내용들이다.

## Git short hash issue

우리팀은 코드 배포 시 배포 이미지 태그를 Git hash를 이용하고 있다. 그런데 언제부터 갑자기 kubernetes에 deploy가 계속 실패하는 현상이 생겨서 확인해보니 이미지 태그를 찾을 수 없어서 배포가 되지 않는다는것을 알게되었다. 분명 CI에서 이미지 push까지 되는것이 확인되었고 ECR Repository에 잘 업로드된 것을 확인했는데 배포가 되지 않는게 이상했다.

한참 찾다가 태그를 보니 hash값의 길이가 다른것을 발견하게 되었다. (매의 눈으로 봐야한다..) CI 명령어를 바꾼적이 없는데 갑자기 왜 안되는지 원인을 찾아보았지만 관련해서 이슈가 제기된 글을 찾을 수 없어서 그냥 아래와 같이 명령어를 바꿔주었다.

As-Is

```
git rev-parse --short HEAD
```

To-Be

```
git rev-parse --short=7 HEAD
```

위와 같이 바꿔주니 제대로 동작하였다. 언제부터 명령어의 동작방식이 바뀐걸까....

## detekt

[detekt](https://github.com/detekt/detekt){: target="\_blank" }은 코틀린 코드 정적 분석기 도구이다. 자바에는 정적분석기로 [sonarqube](https://www.sonarqube.org/){: target="\_blank" }가 있는데 코틀린용으로 정적 분석기가 있다는 것을 이번에 알게 되었다.

정적 분석기를 적용하면 코드리뷰에서 잡아주지 못한 코드 스멜을 발견해 줄 것 같아서 도입해봄직 하지만 현재에도 CI가 실행되는 시간이 상당히 소요되고 있어서 분석기까지 도입하면 CI 시간이 늘어나 작업 생산성에 영향을 미칠까 걱정이 되는 부분이 있다. 여유가 될때 설정 테스트를 해보아야 겟다.

## QueryDSL Projections.constructor

QueryDSL 사용 시 만약 kotlin의 Data Class를 반환 타입으로 지정하고 싶으면 어떻게하면 될까? 바로 `Projections.constructor`를 사용하면 된다. 아래 예시 코드를 참고하자.

```kotlin
jpaQueryFactory
    .select(
        Projections.constructor(
            Test::class.java,
            product.id,
            product._category.`as`("category"),
            product._name.`as`("name"),
            product._standard.`as`("standard"),
            product._unit.`as`("unit"),
            receipt.storeId.`as`("storeId"),
            productPurchase.id.count().`as`(orderFrequency),
        ),
    )
    .from(productPurchase)
    .join(product).on(productPurchase._product.id.eq(product.id))
    .join(receipt).on(productPurchase._receipt.id.eq(receipt.id))
    .where(filter.getExpression())
    .groupBy(
        product.id,
        product._category,
        product._name,
        product._standard,
        product._unit,
        receipt.storeId,
    )
    .orderBy(orderFrequency.desc())
    .limit(3)
    .fetch()
```

참고 글: [https://cheese10yun.github.io/querydsl-projections/](https://cheese10yun.github.io/querydsl-projections/){: target="\_blank" }

## Spring Transactional이 선언된 함수에 `get or create` 를 구현할때 주의할점

[Object Design Style Guide](https://veluxer62.github.io/book-review/object-design-style-guide/#%ED%95%A8%EC%88%98-%EB%AA%85%EB%AA%85){: target="\_blank" }에 따라 조회 함수 구현 시 특별한 이유가 없다면 서비스 레이어에서 특정 리소스를 조회할 때 `get`함수를 사용하고 조회하려는 값이 존재하지 않으면 예외를 던지도록 구현하였다. 그러다보니 Facade 레이어에서 조회 후 값이 존재하지않으면 새로운 리소스를 생성하도록 함수를 구현할때 try catch를 이용하여 예외를 핸들링하도록 구현하였다.

Spring에서 `@Transactionl`이 선언된 함수에서는 함수내에 Uncheked Exception이 발생하면 롤백을 하도록 설정되어 있는데 try catch를 통해서 예외를 핸들링해도 무조건 롤백을 한다는 것이다. 기능 테스트가 아닌 통합테스트로 테스트를 수행하다보니 이러한 이슈를 테스트 코드에서 발견하지 못했고, QA도중 이슈가 발생해서 대응할 수 있었다.

대응방법은 간단하다. 조회한 값이 존재하지 않으면 예외를 던지는 `get`함수를 사용하는 것이 아닌 `Optional`을 반환하는 `find`함수로 대체하는 것으로 대응하였다.

참고 글: [https://techblog.woowahan.com/2606/](https://techblog.woowahan.com/2606/){: target="\_blank" }

## TDD 안티패턴

코드리뷰를 하다보면 종종 테스트 코드에 대한 안티 패턴들을 소개해주곤 한다. 그럴때마다 [http://blog.codepipes.com/testing/software-testing-antipatterns.html](http://blog.codepipes.com/testing/software-testing-antipatterns.html){: target="\_blank" }글을 함께 첨부하곤 하는데 기억해두려고 적어둔다.

## Gradle credential

Gradle 설정 시 만약 credential 관련 값을 다루어야 한다면 property를 활용하자. (물론 보안적으로 좀더 안전하게 하려면 secret 설정을 따로 해주어야 한다.)

참고: [https://docs.gradle.org/current/samples/sample_publishing_credentials.html](https://docs.gradle.org/current/samples/sample_publishing_credentials.html){: target="\_blank" }

## Terraform AWS provider S3 Domain Name issue

테라폼의 AWS providor에 정적사이트 웹 호스팅 시 Domain Name에 region이 들어가지 않는 이슈가 있다.

이슈: [https://github.com/hashicorp/terraform-provider-aws/issues/15102](https://github.com/hashicorp/terraform-provider-aws/issues/15102){: target="\_blank" }

이 문제를 해결하려면 S3의 버킷 도메인 명을 `bucket_domain_name`이 아닌 `bucket_regional_domain_name`로 사용하면 해결할 수 있다.

## MockBean memory leak issue

test container 에 DB connection leak이 있는거 같애서 확인하다가 MockBean을 사용하면서 context가 캐싱되면서 발생하는 문제임을 알게 되엇다.

이슈: [https://stackoverflow.com/questions/68468851/testcontainers-sql-too-many-connections-exception](https://stackoverflow.com/questions/68468851/testcontainers-sql-too-many-connections-exception){: target="\_blank" }

이는 Spring에서 제공하는 [Context Caching 메커니즘](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-caching){: target="\_blank" } 때문인데, MockBean을 사용하면 매번 사용할때마다 새로운 Cache key를 생성하기 때문에 메모리 누수가 발생하게 된다.

이 문제를 해결하기 위해 우선 유닛테스트에서 MockBean을 사용하는 코드를 모두 Mock으로 생성해서 직접 주입시켜주는 방식으로 변경해 주었다. (변경해주니 유닛 테스트 수행속도가 드라마틱하게 올라갔다.) 그리고 통합테스트에서는 위와같은 방식으로 간단하게 해결하기가 쉽지 않아서 통합테스트를 수행하는 부모 클래스를 만들어서 부모클래스에 MockBean을 정의해 주도록 변경하였다. Context Caching은 개별 클래스에서 수행되기 때문에 부모클래스를 두게 되면 MockBean이 한번만 수행되므로 어느정도 보완을 할 수 있다. 하지만 이와 같은 방법은 임시책이므로 MockBean을 완전하게 사용하지 않는 방법으로 찾아보아야 겠다.

참고 글: [https://taes-k.github.io/2022/02/02/spring-mockbean/](https://taes-k.github.io/2022/02/02/spring-mockbean/){: target="\_blank" }
