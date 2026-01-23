---
layout: single
title: "서버 언어 전환 이야기"
categories:
  - REFERENCE
tags:
  - Spring
  - 언어전환
  - JPA
  - JWT
  - 회고
  - Graphql
toc: true
toc_sticky: true
excerpt: 이 글은 사내 블로그에 작성한 언어 전환 이야기 내용을 그대로 가져오면서 나의 블로그의 언어톤에 맞게 변경한 글이다. 최근 스포카는 서울 본사 사무실 이전과 함께 도도 포인트 서비스양도 등 많은 변화의 시간을 가졌었다. 내가 몸담은 도도 카트 팀도 앞으로 더 나은 서비스를 제공하기 위한 정비 기간을 가졌었는데, 정비 기간에 백엔드 챕터는 현재 운영되고 있는 도도 카트 서비스의 서버에 사용된 언어를 변경하기로 하였다. 이 글에서는 이 기간 동안 왜, 그리고 어떻게 언어 전환을 이루었는지 이야기해보도록 하겠다.
---

## Intro

이 글은 사내 블로그에 작성한 [언어 전환 이야기](https://spoqa.github.io/2022/04/15/all-new-server.html){: target="\_blank" } 내용을 그대로 가져오면서 나의 블로그의 언어톤에 맞게 변경한 글이다.

최근 [스포카](https://www.spoqa.com/){: target="\_blank" }는 서울 본사 사무실 이전과 함께 [도도 포인트 서비스양도](https://zdnet.co.kr/view/?no=20220127161642){: target="\_blank" } 등 많은 변화의 시간을 가졌었다. 내가 몸담은 [도도 카트](https://dodocart.co.kr/){: target="\_blank" } 팀도 앞으로 더 나은 서비스를 제공하기 위한 정비 기간을 가졌었는데, 정비 기간에 백엔드 챕터는 현재 운영되고 있는 도도 카트 서비스의 서버에 사용된 언어를 변경하기로 하였다. 이 글에서는 이 기간 동안 왜, 그리고 어떻게 언어 전환을 이루었는지 이야기해보도록 하겠다.

## 언어 전환이 필요합니다!

![언어전환](https://media.giphy.com/media/3osxYrlGDhnsYtl7vW/giphy.gif)

사실 내가 입사할 때 고민했던 요소 중 하나가 나의 커리어 중 python을 이용한 개발 경험이 없었다는 것이었다. 하지만 많은 개발 서적이나 블로그에서 "언어는 도구에 불과하다"라는 말을 믿고 제품만 잘 만들면 될 거라는 생각과 함께 그 고민을 접어 두고 입사를 결정하게 되었다. 2021년 3월에 입사해서 정비 기간을 시작하는 11월까지 도도 카트 서비스의 서버를 개발하면서 python의 문법도 배우고, 생태계도 알아가면서 미약하게나마(?) python과 프레임워크 들을 배워갔었고 또 더 나은 코드를 위한 노력도 함께 이어갔다. ([도메인 주도 개발 전환 이야기](https://veluxer62.github.io/explanation/domain-driven-development-transition-story/){: target="\_blank" }를 참고하면 좋겠다.)

하지만 아래와 같은 이유로 서버 언어의 전환이 필요하다 판단되었고 CTO님과 팀장님 그리고 챕터 구성원분들과 함께 논의해본 결과 언어 전환을 진행하기로 결정하였다. (_아래에 적힌 이유는 지극히 나의 개인적인 생각과 팀이 처한 상황에 대한 내용이다. 특정 언어의 좋고 나쁨을 이야기하는 것이 아니니 넓은 마음으로 이해해주었으면 한다 :)_)

### 팀 내 python과 그 프레임워크에 대한 높은 이해도를 가진 인력의 부재

앞에서도 언급하였지만, 언어는 도구에 불과하다는 말에는 전적으로 동의한다. 하지만 동일한 제품을 만드는 데 더 나은 품질과 더 빠른 속도로 개발할 수 있다면 그 도구를 선택하는 것이 팀을 위해서, 제품을 위해서 더 나은 선택이라 생각된다. 언어를 전환하는 것이 비록 적은 시간이 있어야 하는 것이 아니긴 하지만 팀원 모두가 더 전문성을 가진 도구를 사용하게 됨으로써 장기적으로 높은 생산성을 가지고 개발을 지속해 간다면 언어 전환 시 사용했던 시간을 충분히 상쇄시킬 수 있을 것이라 판단했다.

### 보다 높은 유지보수성

많은 개발자분이 알고 계시겠지만 python은 동적 프로그래밍 언어이다. 컴파일 없이 개발할 수 있어 초기 프로토타입의 제품을 만들 때에는 빠르고 효율적으로 기능을 만들어 낼 수 있었을 것이다. 하지만 기능이 점점 많아지고 기존 기능을 수정해야 할 필요성이 증가함에 따라 이 동적 타입은 우리의 발목을 잡기 시작했다. 비록 python도 타입 힌트를 줄 수 있지만 이것은 단지 코드 작성 시 도움을 주기 위한 기능이지 런타임에 해당 타입으로 동작함을 보장하지는 않는다. (오히려 적혀진 타입과 다른 값이 들어가 한참을 디버깅했던 기억이 있다.) 또한 [덕 타이핑](https://en.wikipedia.org/wiki/Duck_typing){: target="\_blank" }이라는 기술도 참 유연하고 편리한 기술이지만 기능의 수정이 필요할 때 인터페이스가 없기 때문에 구현 코드를 추적하여 수정하는 것이 쉽지 않다는 문제가 있다.

반면 우리가 선택한 [Kotlin](https://kotlinlang.org/){: target="\_blank" }은 타입 추론을 통해 동적 언어와 유사한 코드 스타일을 가져갈 수 있으며 좀 더 단순하면서 가독성 코드를 작성해 줄 수 있게 해주고, 정적 프로그래밍 언어이기에 타입을 보장해 주어 더 나은 유지보수성을 꾀할 수 있다.

### 보다 높은 비즈니스 집중도

경량 프레임워크는 가볍고 개발자가 원하는 대로 커스터마이징하기 좋은 장점을 가지고 있다. 하지만 개발자의 능력에 따라 제품 품질의 편차가 크고 보편적인 개발 방법이 아닌 경우 새로운 개발 팀원이 합류하였을 때 회사의 프레임워크에 적응하는 시간을 많이 써야 한다는 단점도 가지고 있다.

[스프링 프레임워크](https://spring.io/){: target="\_blank" }는 우리가 개발하는 환경에서 필요한 대부분의 기능을 가져다 사용할 수 있는 풍부한 생태계를 제공하기 때문에 비즈니스 코드에만 집중할 수 있도록 한다. 그리고 많은 레퍼런스와 보편적으로 사용되는 설정 등은 새로운 팀원이 합류하더라도 보다 짧은 시간 내에 회사 코드에 적응하는 장점도 가지고 있다.

### 넓은 인력풀

기술적인 부분에만 집중하면 좋겠지만 제품을 만들 때 혼자만으로 모든 것을 만들 수 없기 때문에 인력풀을 생각하지 않을 수 없다. 최근 python은 주목받는 언어로써 많은 개발자가 사용하고 좋아하는 언어이지만 한국에서는 여전히 웹 서버 개발 시 java와 Spring을 사용하는 개발자 비율이 높다고 생각한다. (최근에는 Kotlin + Spring의 사용률도 점점 올라가고 있어 아주 반갑다.) 감사하게도 [우아한 테크코스](https://woowacourse.github.io/){: target="\_blank" }와 같이 좋은 개발 방법까지 함께 알려주는 교육기관이 늘어나면서 좀 더 우리가 원하는 인재를 더 많이 채용할 기회가 생기고 있다.

## 어떤 계획을 가지고 있나요?

![너는 계획이 다 있구나](/assets/images/posts/all-new-server/you-have-plan.gif)

사실 2년 정도 개발된 제품의 코드를 짧은 시간 내에 변경하겠다는 결정과 그 시간을 산정하는 것은 쉽지 않았다. 다행히 [도메인 주도 개발 전환 이야기](https://spoqa.github.io/2021/09/13/domain-driven-development-transition-story.html){: target="\_blank" }에서 소개한 바와 같이 현재 제품에 사용되는 주요 도메인들의 기능목록이 정리되어 있었고, 기존 코드도 패키지 정리와 함께 도메인별로 코드들을 리팩토링을 해왔었다. (이쯤 되면 언어 전환을 위한 준비를 계속해왔다고 볼 수 있겠다.)

해당 데이터들을 기반으로 아래와 같이 전환할 기능 목록을 정리하고

![Graphql API 목록](/assets/images/posts/all-new-server/graphql-api-list.png)

로드맵을 작성하였으며

![roadmap](/assets/images/posts/all-new-server/roadmap.png)

프론트분들께 협조를 구하여 사용 중인 API들을 선별하고

![API 사용여부 조사](/assets/images/posts/all-new-server/survey-api.png)

JIRA의 작업 항목들을 생성하였다.

![Task 목록](/assets/images/posts/all-new-server/task-list.png)

## 어떻게 전환작업을 하였나요?

![How](https://media.giphy.com/media/26mfigxZrxXIrk7xm/giphy.gif)

### 제약 조건

먼저 전환 작업 시 부득이한 경우를 제외하고 절대 변경하지 않겠다고 다짐한 것이 바로 `API Schema`와 `Database`였다.

언어 전환을 하다 보면 많은 난관에 봉착하게 된다. 기존언어에서 사용된 표현 식이나 라이브러리 기능들이 전환하려고 하는 언어 또는 라이브러리가 지원하지 않아 다른 방식을 모색해야 한다든지, 나름대로 기존언어와 동일한 로직으로 구현한다고 노력하였지만 실제로는 다르게 동작한다는 것과 같은 이슈들이 생기게 된다. 이러한 문제들을 해결해가며 전환작업을 하는 것도 힘든데 기존 API의 Schema와 Database의 Column 또는 Table 들을 좀 더 나은 방식으로 변경하는 욕심까지 부린다면 전환 프로젝트는 영영 끝나지 않거나 약속된 기간을 지키지 못하여 작업 자체가 중단되는 상황이 발생하기도 한다.

우리는 이러한 위험성을 최소한으로 하고 싶었다. 그래서 프론트앤드에서 전혀 사용하지 않는 API를 제거하는 작업과 부득이하게 변경을 필요로 하는 작업 몇몇을 제외하고는 API Schema의 변경을 최소한으로 하였고 Database는 하나도 변경하지 않았다. 그러다 보니 오롯이 비즈니스 코드를 새로운 언어로 옮기는 데에만 집중할 수 있었다. 그리고 기존 통합 테스트 코드의 테스트 케이스를 그대로 가지고 올 수 있었기 때문에 테스트 케이스를 새롭게 고민하여야 하는 공수를 많이 줄일 수 있었다.

### 테스트

기존에 작성된 테스트와 유사하게 이번에도 도메인 레이어에서는 유닛테스트를 어플리케이션 레이어에서는 통합테스트를 작성하며 언어 전환 작업을 수행하였다. 도메인 레이어에서 유닛테스트를 수행한 이유는 도메인 레이어가 의존성이 적기 때문에 각 유즈케이스를 보다 쉽게 그리고 세세하게 다룰 수 있었기 때문에 엣지 케이스에 대한 테스트 케이스 작성이 보다 용이하다고 판단했기 때문이다. 만약 어플리케이션 레이어에서 통합테스트로 도메인이 가진 모든 유즈케이스를 테스트 케이스로 표현하려고 한다면 중복되는 테스트도 엄청나게 많아질 것이고 의존하는 코드가 많기 때문에 코드가 장황해지면서 가독성이 상당히 나쁜 테스트 코드들이 작성될 것 같았다.

이러한 테스트 작성 노력으로 인해 실제로 QA 입고 시까지 로컬이나 개발환경에서 서버를 거의 실행시켜보지 않고도 언어 전환을 완료할 수 있었다. 비록 몇몇 이슈가 발생하긴 하였지만 비즈니스 로직에 의한 버그가 아닌 실행환경의 차이로 인해 미처 발견하지 못했던 버그들이었고 이것 또한 손쉽게 원인을 파악하고 고칠 수 있었다.

## 새로운 서버는 어떤 기술들이 적용되었나요?

이번에 변경될 서버의 기술 스택을 적어보면 아래와 같다.

[kotlin](https://kotlinlang.org/){: target="\_blank" }, [Spring Boot](https://spring.io/projects/spring-boot){: target="\_blank" }, [Gradle](https://gradle.org/){: target="\_blank" }, [DGS](https://netflix.github.io/dgs/){: target="\_blank" }, [Spring Data JPA](https://spring.io/projects/spring-data-jpa){: target="\_blank" }, [QueryDSL](http://querydsl.com/){: target="\_blank" }, [Spring Data Elasticsearch](https://spring.io/projects/spring-data-elasticsearch){: target="\_blank" }, [Kotest](https://kotest.io/){: target="\_blank" }, [Flyway](https://flywaydb.org/){: target="\_blank" }, [Spring Security](https://spring.io/projects/spring-security){: target="\_blank" }, [ktlint](https://ktlint.github.io/){: target="\_blank" }, [Test Logger](https://github.com/radarsh/gradle-test-logger-plugin){: target="\_blank" }, [jib](https://github.com/GoogleContainerTools/jib){: target="\_blank" }, [OpenFeign](https://spring.io/projects/spring-cloud-openfeign){: target="\_blank" }, [AWS](https://spring.io/projects/spring-cloud-aws){: target="\_blank" }, [BigQuery](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/bigquery.html){: target="\_blank" }, [ULID](https://github.com/f4b6a3/ulid-creator){: target="\_blank" }, [Kotlin logging](https://github.com/MicroUtils/kotlin-logging){: target="\_blank" }, [libphonenumber](https://github.com/google/libphonenumber){: target="\_blank" }, [Tika](https://tika.apache.org/1.14/gettingstarted.html){: target="\_blank" }, [Sendgrid](https://github.com/sendgrid/sendgrid-java){: target="\_blank" }, [Firebase](https://github.com/firebase/firebase-admin-java){: target="\_blank" }, [Elastic APM](https://www.elastic.co/guide/en/observability/current/index.html){: target="\_blank" }, [POI](https://poi.apache.org/){: target="\_blank" }, [mockk](https://mockk.io/){: target="\_blank" }, [KFactory](https://github.com/bluegroundltd/kfactory){: target="\_blank" }, [Java Faker
](https://github.com/DiUS/java-faker){: target="\_blank" }

적용된 기술들을 어떻게 사용하는지 모두 하나하나 말해드리고 싶은 마음이 굴뚝같지만 본 글의 주제를 흐릴 수 있고 내용이 너무 방대해질 것 같으므로 별도로 소개하는 글을 적어보도록 하겠다. (이렇게 떡밥들만 던져놓았다가 어떻게 회수하려고...)

![paste-bait](/assets/images/posts/all-new-server/paste-bait.jpeg)

## 점진적으로 언어전환을 할 순 없나요?

![Why](https://media.giphy.com/media/3orif8qr37qRdmcEYU/giphy.gif)

언어 전환 계획을 보면 전환작업을 모두 끝낸 후 한 번에 변경 사항을 배포하는 일정으로 계획되어 있다. 그럼, 여기서 왜 점진적 변경이 아닌 일괄적인 변경으로 큰 리스크를 짊어지나? 라는 질문을 할 수 있을 것 같다.

나도 점진적 변경을 통해 리스크를 최소화하고 서비스 기능 개발과 언어 전환 작업을 함께하는 아주 이상적인 모습으로 이 작업을 수행하고 싶었다. 하지만 기존 서버를 운영하면서 언어 전환을 시도하다 레거시는 레거시대로 2배로 늘어나고 언어 전환은 새로운 기능을 쫓아가지 못해 결국 흐지부지되는 상황을 겪어본 터라 하려면 온전히 집중해서 언어 전환을 빠르게 마무리 짓는 게 나을 것이라 판단했다.

또한 현재 도도 카트 서비스는 Graphql을 사용하고 있는데, Graphql의 특성상 단일 path를 사용한다. 그러다 보니 [Load balancer에서 경로기반 라우팅](https://aws.amazon.com/ko/premiumsupport/knowledge-center/elb-achieve-path-based-routing-alb/){: target="\_blank" }을 이용하여 앱의 배포 없이 점진적으로 서버를 교체하기에는 무리가 있어 보였다. (혹시 대안을 알고 있는 분이 계신다면 알려주시면 정말 감사하겠다!)

## 이슈가 있습니다.

![이슈](https://media.giphy.com/media/t6008cWl3tAFADOCOw/giphy.gif)

쉽지 않은 작업, 문제없이 순탄하게 진행되면 좋겠지만 세상만사 뜻대로 되지 않았다. 언어 전환을 하면서 어떤 이슈들이 있었는지 보겠다.

### 도도 포인트 서비스 양도

[도도 포인트 서비스양도](https://zdnet.co.kr/view/?no=20220127161642){: target="\_blank" } 소식은 내부 직원들에게 큰 이슈였다. 10년 동안 스포카를 있게 해준 서비스였기 때문에 많은 아쉬움과 함께 코드, 인프라 등 이관을 위한 많은 작업을 필요로 하였다. 이 부분은 전환 계획 시 고려되지 않았던 이슈였기 때문에 전환 작업 일정에 많은 영향을 미쳤다.

### JWT

도도 카트 서비스는 API를 사용하기 위한 인증 수단으로 [JWT](https://jwt.io/){: target="\_blank" }를 사용한다. 프레임워크를 변경하면서 이 JWT를 통한 인증 및 인가를 위한 코드를 새롭게 작성하게 되었고 이 과정에서 기존의 인증 방식에 대한 문제점을 발견하게 되었다. 이 이슈에 대한 내용을 적기에는 글의 내용이 너무 많아질 듯하여 추후 별도의 글로 해결 과정들을 소개해 보고자 한다.

### JPA

앞서 말한 바와 같이, 우리 백엔드 챕터는 python으로 서버를 개발할 때에도 언어 전환을 할 때에도 항상 테스트 코드를 작성하고 있다. 유닛테스트와 통합테스트를 섞어서 사용함으로써 좀 더 안정성 높은 서비스를 만들어가기 위해 노력하였다. 하지만 QA 진행 중에 테스트 코드에서는 발견할 수 없었던 JPA와 관련한 이슈가 다수 발생했다. ([JPA](https://en.wikipedia.org/wiki/Jakarta_Persistence){: target="\_blank" }는 Spring Framework를 이용한 개발 시 ORM을 사용할 때 JPA를 빼놓으면 서운할 정도로 요즘에는 많이 쓰는 라이브러리이다.)

테스트 시 데이터 클리닝을 좀 더 편하게 하기 위해서 기능 테스트가 아닌 통합 테스트를 선택하였는데 이 실행 환경의 차이로 인해 QA 시 몇몇 이슈가 생긴 부분에 대해서는 아쉬움으로 남는다. JPA로 인해 발생했던 이슈들도 별도의 글로 소개하면서 해결 과정을 자세히 알아보고자 한다.

## 회고합시다~

언어 전환 작업에 대해서 총 2번의 회고를 진행하였다. 위 로드맵에서 보았다시피 언어 전환 작업은 크게 `도메인` 레이어에 대한 전환 작업과 `어플리케이션` 레이어에 대한 전환작업으로 나누어서 진행하였다. 기간이 짧지 않은 만큼 중간에 쉬어가는 시간도 가질 겸 중간 점검의 목적으로 `도메인` 레이어 작업이 끝나고 난 후 회고를 한번 진행하고 언어 전환이 끝난 후에 전체 작업에 대한 회고를 한 번 더 하는 방식이었다.

회고의 내용은 크게 작업의 퍼포먼스를 돌아보는 부분과 작업 기간 내 변경하고 싶은 점,

![task-performance](/assets/images/posts/all-new-server/task-performance.png)

계속 이어가고 싶은 점, 시도해보고 싶은 점들을 이야기하는 회고 부분,

![retrospective](/assets/images/posts/all-new-server/retrospective.png)

그리고 회고 시 나온 Action Item 들을 적어 다음 수행 시 개선할 부분을 적어보는 부분으로 나누어 진행했다.

![action-items](/assets/images/posts/all-new-server/action-itmes.png)

우리는 다른 부분보다 특히 작업에 대한 퍼포먼스를 돌아보는 부분에 많은 이야기를 나누었는데, 이때 [JIRA의 Control Chart](https://support.atlassian.com/jira-software-cloud/docs/view-and-understand-the-control-chart/){: target="\_blank" }를 이용하여 이야기를 나누었던 것이 큰 도움이 되었다.

Control Chart를 통해서 우리는 정해진 기간 동안 평균적으로 어느 정도 소요 시간을 가지고 작업을 처리해왔고 얼마나 효율적이고 생산성이 나아지는지, 나빠지는지를 파악할 수 있었다. 아래 사진은 저희가 언어 전환 기간 동안 수행했던 작업을 Control Chart로 표시한 것이다. 해당 차트를 통해 언어 전환 작업을 진행하는 동안 점점 생산성이 높아지고 있었음을 알 수 있었고 중간에 어떤 이슈로 인해 작업에 영향을 미쳤는지 등을 파악하는 데 많은 도움이 되었다. (태스크를 생성하고 해당 태스크 하위로 서브 태스크들을 생성해서 작업하다 보니 분포도 표시에 조금 모순이 있다. 이 부분을 감안하여 데이터를 분석하였다.)

![cycle-time](/assets/images/posts/all-new-server/cycle-time.png)

## 마치며

지금까지 우리가 언어 전환을 어떻게 계획하고 수행해 왔는지 이야기해보았다. 기술적인 이야기보다 비기술적인 이야기가 대부분이었는데, 기술적인 부분은 앞서 예고 드린 바와 같이 앞으로 계속해서 구체적인 내용의 글로 찾아오도록 하겠다.

![like](/assets/images/posts/all-new-server/like.jpeg)

이번 작업을 통해서 언어 전환도 전환이지만 현재 우리 도도 카트 서비스에 대한 이해도가 몇 배 이상 증가했다는 점만으로도 유종의 미를 거두었다고 생각한다. 구현하면서 몰랐던 기능들을 아주 많이 알게 되었기 때문이다. 그리고 이전에 하지 못했던 APM이라던지 불필요한 코드들을 제거하는 등의 많은 부분을 개선할 수 있게 되어서 작업자들의 만족도가 아주 높았다는 점도 성공적인 전환작업이었다고 말할 수 있을 것 같다.

한편, 전환 작업을 통해서 언어 변경을 통해 앞으로의 생산성이 높아질 것이란 기대감을 가지고 있다. 향후 많은 새로운 기능들이 기다리고 있는데, 새로운 기능들을 개발해 가면서 이 전환 작업이 어떤 도움이 되었는지 돌아보는 시간을 가지는 것도 재미있을 것 같다.

<br/>

개인적으로 이번 언어전환은 내가 제안하고 주도하게 되면서 나에게 있어 상당히 도전적이고 기억에 많이 남을 프로젝트였다. 앞으로 이런 기회는 거의 찾아오지 않을 것이기 때문에 더욱 값진 경험이라 생각된다. 아직도 남아있는 과제는 많은데 지금까지 열심히 달려온다고 지친 몸과 마음을 조금 추스린 다음 과제들도 잘 진행하면서 더 재밌는 경험들을 많이 하길 기대해본다.
