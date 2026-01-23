---
layout: single
title: "2019년 회고"
toc: true
toc_sticky: true
permalink: /retrospective/2019-retrospective/
categories:
  - RETROSPECTIVE
excerpt: 2019년에 내가 배웠거나 사용했었던 것들을 되돌아보고 앞으로 2020년에 해야할 것들에 대한 정리를 해보고자 한다.
---

2019년에 내가 배웠거나 사용했었던 것들을 되돌아보고 앞으로 2020년에 해야할 것들에 대한 정리를 해보고자 한다.

## Spring WebFlux

[![Spring WebFlux](/assets/images/retrospective/webflux.jpg)](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_800/http://mdabrowski.net/wp-content/uploads/2019/03/webflux.jpg){: target="\_blank" }

Spring에서 Reactive 프로그래밍을 위해 [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux){: target="\_blank" }가 개발되었다. Spring 5.x 버전부터 사용이 가능하다.

개인 프로젝트로 [빈자리 예약 봇](https://github.com/anytimedebug/occupying){: target="\_blank" }을 진행하면서 서버를 Spring WebFlux로 구현해보았다. Spring MVC와 비교해보았을 때 실제로 이점을 잘 느끼지 못하였다. 솔직히 아직 익숙하지 않기도 하고 Spring WebFlux가 요구하는 Reactive 프로그래밍에 맞게 작성하였는 지는 확실치가 않다. 좀더 공부하면서 코드를 개선해 나가야 할것 같다.

2019년에는 구현을 해 본것에 만족하였지만 2020년에는 제대로된 구현에 초점을 맞추고 Reactive 프로그래밍이 어떤것인지 잘 이해하고 사용해 봐야 겠다.

## 카카오 i 오픈빌더

[![카카오 i 오픈빌더](/assets/images/retrospective/kakao-i-open-builder.png)](https://t1.kakaocdn.net/openbuilder/og/opengraph.png){: target="\_blank" }

[카카오 i 오픈빌더](https://i.kakao.com/docs/getting-started-overview#%EC%98%A4%ED%94%88%EB%B9%8C%EB%8D%94-%EC%86%8C%EA%B0%9C){: target="\_blank" }는 카카오에서 제공하는 채널의 봇 설계 플랫폼이다.

개인 프로젝트로 [빈자리 예약 봇](https://github.com/anytimedebug/occupying){: target="\_blank" }을 진행하면서 카카오 채널의 챗봇을 이용하여 사용자 UI를 제공하였다. **_카카오 i 오픈빌더_**는 사용자가 편리하게 봇을 설정할 수 있게 UI를 제공해 준다. 비 개발자라도 손쉽게 챗봇 설정을 할 수 있을 것 같다. 다만 **_카카오 i 오픈빌더_**의 [스킬](https://i.kakao.com/docs/skill-build#%EC%8A%A4%ED%82%AC-%EC%84%9C%EB%B2%84-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0){: target="\_blank" }은 요구사항이나 제약조건을 잘 알고 사용하지 않으면 디버깅에 어려움이 있었다.

2020년에는 챗봇에서 제공해주는 좀더 디테일한 기능(예를 들면, [엔티티](https://i.kakao.com/docs/tutorial-chatbot-useful-tips#%EC%97%94%ED%8B%B0%ED%8B%B0-%EC%9E%91%EC%84%B1-%ED%8C%81){: target="\_blank" })을 적용해서 운영해봐야 겠다.

## Vue.js

[![Vue.js](/assets/images/retrospective/vuejs.png)](https://images.velog.io/post-images/kay/a0ddc410-d858-11e8-a33d-b5e9b030c4e4/vuejs-tutorial2d2a853c-aa2f-44b0-80df-933b495f77f8.png){: target="\_blank" }

[Vue.js](https://kr.vuejs.org/v2/guide/index.html){: target="\_blank" }는 UI를 만들기 위한 프로그래시브 프레임워크이다.

사내 관리자 서비스에는 view template으로 [Freemarker](https://freemarker.apache.org/){: target="\_blank" }와 [javascript](https://developer.mozilla.org/ko/docs/Web/JavaScript){: target="\_blank" }와 [JQuery](https://jquery.com/){: target="\_blank" }를 이용하여 UI를 구현하였었다. 화면이 복잡해짐에 따라 javascript와 JQuery만 사용해서는 데이터의 변경과 UI 변경을 연결해 주는 수많은 코드가 생기면서 읽기 어려우며 유지보수 하기에도 쉽지않는 코드가 많아졌다.

기존 코드가 Spring MVC로 개발되어 있다 보니 전체를 다 변경하기에는 시간이나 인력의 여유가 있지 않았다. 그래서 선택한 방법이 Freemarker 템플릿에 vue.js를 임포트 하는 방법이었다. React도 고려하였으나 양방향 바인딩을 제공하는 vue가 좀더 적합하다고 판단하였다. 결과는 매우 만족스러웠다. 비록 화면을 테스트할 수 없고 UI만을 위해서 정적 페이지가 아닌 Spring 서비스를 실행하여야 한다는 단점이 있었지만(API 서비스와 관리자 서비스가 분리되어 있다.) 데이터의 변경과 UI컴포넌트의 변경을 더이상 코드로써 구현하지 않아도 된다는 점은 개발시간을 많이 단축해 주었고, 좀더 복잡하고 화려한 UI를 구현하는 데에도 부담스럽지 않게 되었다.

## React.js + Typescript

[![react-typescript](/assets/images/retrospective/react-typescript.png)](https://salgum1114.github.io/static/images/covers/react_typescript.png){: target="\_blank" }

[React.js](https://ko.reactjs.org/){: target="\_blank" }는 UI를 만들기 위한 Javascript 라이브러리이다. [Typescript](https://www.typescriptlang.org/){: target="\_blank" }는 자바스크립트를 확장한 언어로 Javascript의 모든 구문과 의미를 지원하면서 정적 유형 및 추가적인 구문과 기능을 제공해주는 프로그래밍 언어이다.

Vue로 개발하던 사내 관리자 서비스에 UI에도 TDD를 통한 개발과 좀 더 나은 유지보수 그리고 정적 페이지로 구현하기 위해서 React.js + Typescript를 도입하기로 결정하였다. 디자인 시스템으로는 [Ant Design](https://ant.design/){: target="\_blank" }을 채택하였다. Ant Design은 React.js와 Typescript 모두 지원한다.

아쉬웠던 점은 백엔드 개발자에게 React.js는 러닝커브가 낮지는 않다는 점이다. 그리고 단순한 구현에도 적지않은 코드가 필요하여 처음에 구현할 때엔 거추장 스럽다고 느껴지기까지 하였다.

하지만 단점보다는 장점이 더 많다고 생각되어 앞으로도 새롭게 추가되는 기능은 그대로 React.js로 구현할 예정이다. 2020년에는 TDD로 개발하는 비율을 좀더 늘려나갈 예정이며 현재는 Redux를 사용하고 있진 않지만 전역으로 관리되는 상태가 필요한 상황이 생기는 경우에는 추가해볼 예정이다.

## Azure DevOps

[![azure-devops](/assets/images/retrospective/azure-devops.png)](https://miro.medium.com/proxy/1*fs244dG4ZD3OwinIf17Kew.jpeg){: target="\_blank" }

[Azure DevOps](https://azure.microsoft.com/ko-kr/services/devops/){: target="\_blank" }는 개발을 위해 Azure에서 제공해주는 협업관리 서비스이다.

기존에 사용하던 [Github](https://github.com/){: target="\_blank" }과, [Jira](https://www.atlassian.com/ko/software/jira){: target="\_blank" }는 사용을 중단하고 Azure DevOps로 변경하였다. Issue관리와 코드리뷰가 좀더 수월해졌고 강력한 CI, CD관리 기능을 제공해 주어서 아주 유용하게 사용하고 있다. 다만 Wiki 기능은 [Confluence](https://www.atlassian.com/ko/software/confluence){: target="\_blank" }에 비해 다소 떨어지는 면이 있어 개발 부서가 아닌 부서에서는 사용을 안하고 있다.

앞으로도 CTO님이 바뀌지 않은 한 Azure DevOps는 계속해서 사용할 것 같다. 2020년에는 Azure Pipelines를 좀더 배워서 사용해보아야 겠다.

## 영어공부

[![grammar-in-use-intermediate](/assets/images/retrospective/books/grammar-in-use-intermediate.jpg)](http://image.yes24.com/momo/TopCate977/MidCate005/7183757.jpg){: target="\_blank" }

최근 영어로된 문서들을 읽을 일이 많아져서 영어 독해 능력을 키우고자 국민 영어 문법책인 [Grammar in use intermediate](http://www.yes24.com/Product/Goods/3376544){: target="\_blank" }을 사서 공부하였다. 이 책을 선택한 또 다른 이유는 [YouTube에 강의영상](https://www.youtube.com/playlist?list=PLoN8iLx8CwQ5WFM9Md37cfcxZEgSGs6yC){: target="\_blank" }이 있어서 였다.

이 책의 장점은 시험위주의 영문법이 아닌 실용적인 영문법으로 실제로 사용되는 예제 위주로 공부를 할 수 있도록 도와준다는 것이었다. 이 책 덕분에 부족했던 문법을 좀더 채울 수 있게 되었고, 독해를 하는데 많은 도움이 되었다.

2020년에는 영어독해 실력을 더 높이기 위해서 [해커스 리딩 스타트](http://www.yes24.com/Product/Goods/376479){: target="\_blank" }를 공부하고 있다. 영어로된 문서를 보는 것이 두렵지 않는 그날까지 꾸준히 멈추지 않고 노력해야 겠다.

## Spring Data REST

[![spring-data-rest](/assets/images/retrospective/spring-data-rest.png)](https://spring.io/projects/spring-data-rest#overview){: target="\_blank" }

[Spring Data REST](https://spring.io/projects/spring-data-rest){: target="\_blank" }는 Spring Data 저장소를 기반으로 REST 웹 서비스를 손쉽게 구축할 수 있도록 해준다. Persist Model과 Repository만 구현해주면 자동으로 REST API를 생성해준다는 점은 생산적인 측면에서 아주 매력적이라고 생각한다. 그리고 제공해주는 웹 API는 [Level 3의 REST 요구조건](https://martinfowler.com/articles/richardsonMaturityModel.html){: target="\_blank" }을 충족하므로 REST API를 제대로 사용한다는 것이 어떤 것인지 직접 볼 수 있다. (제대로 사용하는 것이 얼마나 어려운지도 알 수 있다.)

실무에서는 사용해보지 않았지만 튜토리얼로 구현해보면서 실무에 적용하기에는 다음과 같은 아쉬운 점들이 있어 적용을 검토하지는 않았다.

1. 데이터 모델을 API에 그대로 노출한다.

   데이터 모델이 API에 그대로 노출된다는 것은 곧 API의 변경이 데이터 구조의 변경을 의미하거나 데이터가 변경되게 되면 API의 변경이 발생한다는 것이다. 이는 변경에 취약한 설계가 될 수 있다는 의미이고, API를 사용하는 사용자에게 데이터의 구조를 그대로 노출한다는 문제점을 가진다.

2. 커스터마이징이 쉽지 않다.

   API에서 데이터를 저장, 조회까지의 흐름에 비지니스 로직이 추가되어야 하는 경우 커스터마이징이 불가피하다. 손쉽게 구현할 수 있도록 추상화 되어 있는 만큼 정확히 이해하고 커스터마이징 하지 않는다면 차라리 Spring MVC로 구현하는 것이 더 쉬울 수 있다.

## TDD

[![TDD](/assets/images/retrospective/tdd.gif)](https://nesoy.github.io/assets/posts/20170131/2.gif){: target="\_blank" }

[Test-driven development(TDD)](https://en.wikipedia.org/wiki/Test-driven_development){: target="\_blank" }란 짧은 개발 사이클을 반복하는 소프트웨어 개발 프로세스 중 하나이며 `자동화된 테스트 케이스 작성` -> `코드 작성` -> `리펙토링` 순서로 진행된다.

2019년 초 CTO님께서 새로 오시기 전까지 개발실의 전략은 **_Quick and Dirty_**였다. 현재는 퇴사하셨지만 당시 개발실을 담당하시던 실장님께서는 서비스를 빠르게 성장 시키기 위한 선택으로 이 전략을 택하셨고, 덕분에(?) 빠르게 서비스는 성장할 수 있었다. 서비스가 어느 정도 성장하면서 개발 실장님을 포함한 임원진은 Quick and Dirty 전략 문제점을 개선하고자 CTO님을 영입하셨고 현재 개발 프로세스를 개선해 나가고 있다. ([박재성님의 글](https://www.slipp.net/wiki/pages/viewpage.action?pageId=17924123){: target="\_blank" }을 보면 Quick and Dirty 전략에 대해 잘 설명해주시고 있다.)

다시 TDD로 돌아와서 개발실의 전략이 변경됨에 따라 TEST 코드의 중요성이 부각되기 시작하였다. 기존 코드는 테스트 일단 어쩔 수 없이 그대로 두고 새로운 기능을 추가할때 부터 유닛테스트를 작성하는 것부터 시도해보았다. 현재는 유닛테스트 뿐만 아니라 기능 테스트까지 다양하게 테스트를 적용해가면서 TDD에 대해 많이 알아가고 있는 중이다.

TDD를 하면서 가장 크게 느꼈던 점은 테스트를 어떻게 작성하고 전략을 짜는가에 따라서 안정성이나 유지보수성이 크게 달라질 수 있다는 점이 었다. 이것은 단순히 이론적으로만 봐서는 되는게 아니라 실무에 적용해 보지 않으면 느낄 수 없는 점이라 생각한다.

아직 TDD를 하는 것이 익숙하지 않고 부족한 것이 많지만 좀더 많은 코드를 테스트로 작성 해보면서 공부해야 겠다.

## JPA

[![JPA](/assets/images/retrospective/jpa.png)](http://www.javawebtutor.com/images/jpa/jpa-final.png){: target="\_blank" }

[JPA](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%ED%8D%BC%EC%8B%9C%EC%8A%A4%ED%84%B4%EC%8A%A4_API){: target="\_blank" }란 응용프로그램에서 관계형 데이터베이스의 관리를 표현하는 자바 API를 말한다. 개인적으로 느낀 JPA는 잘 사용하면 정말 좋은 기술이지만 제대로 알고 사용하기가 정말 쉽지 않고 잘못 사용할 경우 성능 이슈가 발생할 가능성이 아주 높은 기술이라고 느껴졌다.

2019년에 사용하면서 겪었던 이슈에 대해 적어본다.

1. cascade(영속성 전이)

   JPA에서는 영속성 전이를 통하여 특정 엔티티에 연관된 엔티티도 함께 영속 상태로 만들 수 있다. 왜인지는 모르겠으나 사내에서는 영속성 전이를 제대로 사용하고 있지 않고 있다. 나 또한 [Mybatis](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%9D%B4%EB%B0%94%ED%8B%B0%EC%8A%A4){: target="\_blank" }를 사용하다가 JPA를 처음 사용하면서 제대로 기술을 익히지 않은 상태로 사용하다 보니 영속성 전이를 올바르게 사용하지 못하였었다.

   영속성 전이를 제대로 사용한다면 코드량을 많이 줄일 수 있고, 명시적인 코드를 작성할 수 있다. 하지만 정확히 알고 사용하지 않는다면 자신도 모르는 사이에 영속 데이터가 변경되어 디버깅에 애를 먹기 쉽다. 특히 운영하고 있는 서비스에서 엔티티 모델은 아주 복잡한 관계를 가진다. 영속성 전이 속성을 잘못 지정해 주는 경우 예기치 못한 버그가 생길 수 있으므로 조심해서 사용해야 한다.

2. Dirty Checking

   [Dirty Checking](https://jojoldu.tistory.com/415){: target="\_blank" }이란 Transaction 내에서 엔티티의 변경이 발생하면 변경 내용을 자동으로 영속 상태로 만들어 주는 JPA의 기능을 말한다.

   사내 코드에는 엔티티 모델을 DTO처럼 사용하며 비지니스 로직을 수행하는 코드를 많이 볼 수 있다. (심지어 API에 노출시키기도 한다...) 다행스럽게도(?) 엔티티를 조회한 후 속성을 바꾸는 로직이 존재하지 않아(못봤을 수도 있다.) Dirty Checking으로 인한 의도하지 않은 데이터 변경 이슈는 없었지만 위험성은 여전히 남아 있다.

   Dirty Checking은 정확히 알고 사용한다면 코드량을 줄일 수 있지만 명시적이지는 못한거 같아 나는 사용을 지양하고 있다.

   두가지 사례만 보더라도 제대로 사용하지 않는 경우 중요한 데이터를 의도치 않게 저장할 수 있는 문제가 있어서 사용에 주의를 더 기울여야 겠다.

## Event sourcing

[Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html){: target="\_blank" }이란 도메인 모델에서 발생하는 모든 이벤트를 기록하는 저장 기법을 말한다.

이벤트가 발행한 이력을 신뢰성 있게 저장하여 데이터를 언제든지 재현할 수 있다는 점에서 중요한 데이터를 다루는 도메인이라면 구현을 검토해볼 가치가 있는 기법인 것 같다. 하지만 구현 방법이 단순하지 않고 신뢰성을 어느정도 까지 만족할지 결정하는 것과 다량으로 저장된 데이터를 어떻게 관리할 것인지 등 고려해야할 사항이 많다는 점은 모든 도메인에 도입을 하기에는 과한 구현이 될 수 있다는 점도 알고 도입을 검토해 보아야 하겠다.

Event Sourcing은 개념과 어떤방식으로 사용하는 지에 대해서만 배웠고 실제 구현경험은 없다. 앞으로 진행하는 프로젝트에서 Event Sourcing을 경험해 볼 것 같다.

## Transaction Isolation Levels

[Transaction Isolation Levels](https://www.geeksforgeeks.org/transaction-isolation-levels-dbms/){: target="\_blank" }은 동시에 여러 Transaction이 발생할 때 특정 Transaction이 다른 Transaction에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지를 결정하는 수준을 말한다.

대량의 데이터를 성능을 위해 멀티 쓰레드에서 업데이트를 하도록 하는 기능을 구현하는 프로젝트를 진행하면서 그동안 내가 얼마나 Isolation에 대한 고려가 부족했는지 느낄 수 있었다. 이 프로젝트를 진행하면서 Isolation을 어떻게 고민하고 처리해야할 지 많이 배울 수 있었다. 그리고 Isolation을 고민하지 않게끔 설계하는 방법을 고려해 보는 것도 좋은 방법이라는 것도 알 수 있었다.

## Git

[![GIT](/assets/images/retrospective/git.png)](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e0/Git-logo.svg/230px-Git-logo.svg.png){: target="\_blank" }

[Git](<https://ko.wikipedia.org/wiki/%EA%B9%83_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4)>){: target="\_blank" }은 컴퓨터 파일의 변경을 추적하고 여러 사용자들 간에 파일들의 작업을 조율하기 위한 분산 버전 관리 시스템을 말한다.

2019년 이전에도 GIT은 사용하고 있었지만 단순히 Branch를 만들고 개발한 후 병합하는 용도 정도로만 사용하였고, 명령어도 Tool의 힘을 빌러 사용해 왔었다. 2019년에는 Git의 사용 시 CLI에서 명령어를 입력하여 사용하도록 변경하였고, rebase 명령어를 적극 활용하면서 다양한 상황에서의 버전관리 방법을 익혔다.

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/){: target="\_blank" }도 도입해 보면서 커밋 메시지 작성에도 많은 노력을 기울이기도 하였다.

2020년에는 자주 사용하지는 않지만 알아두면 좋을 다른 명령어들을 공부해 봐야겠다.

## 회고

[회고](https://github.com/JaeYeopHan/tip-archive/issues/8){: target="\_blank" }란 이전 시점에서 현재 시점까지 발생한 일들을 되돌아보고 좋았던 부분과 아쉬운 부분을 정리한 후 어떻게 개선할지를 알아가면서 앞으로의 일을 개선해 나가는 일련의 과정을 말한다.

CTO님이 새로 오신 이후 개발실 회고를 시작으로 프로젝트 회고까지 많은 회고를 진행해오고 있다. 처음에는 바쁜 일정에 회고라는 시간을 가지는게 불필요하다고 느껴졌었으나 현재는 나의 개인 생활의 회고도 작성할 정도로 장점을 많이 느끼고 있다.

2020년에는 좀더 적극적으로 회고에 참가하고 지난일을 곱씹어보아야 겠다.

## Leaky abstraction

회사의 동료의 소개로 [The Law of Leaky Abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/){: target="\_blank" }글을 접하게 되었다. 이 글을 읽을 당시 Spring의 @Transactional 어노테이션이 잘못 사용되고 있는 부분을 고치던 중이었다. @Transactional 어노테이션을 사용하려면 Bean 개체의 pulic 함수만 사용가능한데 pulic 함수 내 함수에서 사용하면서 실제로는 Tranaction이 동작하지 않고 있었던 것이다.

**_The Law of Leak Abstractions_** 글에서는 추상화는 우리에게 고수준의 기능을 손쉽게 사용하도록 도와주지만 그 추상화된 기능을 이해하는데 드는 비용이 증가하고 있다고 말하고 있다. 또 잘못된 추상화의 구현으로 인해 그것을 고치는 비용이 증가한다고 한다. 즉, 빠르게 개발하기 위한 추상화가 잘못된 개발로 인한 수정 비용을 유발한다는 것이고 이것을 leaky abstraction이라고 표현하였다.

Spring을 사용하는 개발자로써 잘 추상화된 기능은 업무 효율성을 아주많이 올려주었다고 생각한다. 하지만 위의 사례와 같이 정확한 이해 없이 잘못 사용되는 경우 수정의 비용이 적지 않다는 것을 알고 Reference를 꼼꼼하게 체크하는 습관을 가져야 겠다.

## Domain Model is not Persist Model

[Just Stop It! The Domain Model Is Not The Persistence Model](https://blog.sapiensworks.com/post/2012/04/07/Just-Stop-It!-The-Domain-Model-Is-Not-The-Persistence-Model.aspx){: target="\_blank" } 글을 보고 정말 많은 것을 느꼈다.

Domain Model은 실제 문제와 그 문제의 해결방법, 행위를 모델링한 것이라면, Persistence Model은 데이터가 저장되는 대상과 방법, 데이터의 구조를 모델링 한것이다. 이 두 모델은 분명이 역할이 다른 모델이며 구분되어야 하는 모델이다. 하지만 나는 Persistence Model을 Domain Model인 것처럼 문제의 해결방법과 행위를 기술하였고, 데이터 저장을 위한 속성들을 정의하였다. 이렇게 함으로써 Domain Model로써 불필요한 속성을 가지게 되기도 하고 Persistence Model로써 불필요한 행위를 가지기도 한 Model을 사용하였었다.

나는 블로그에서 기술한 것 처럼 분리하는게 맞다고 생각한다. 두 모델은 확실히 다른 책임을 가지고 있고 Persistence 모델이 변경비용이 훨씬 크므로 요구사항의 변경 가능성이 높은 Domain Model과의 분리는 유연한 설계를 위해 필요한 것이라 판단된다.

물론 두 모델을 분리하는 것이 불필요한 코드를 작성하게끔 강제한다는 견해도 있다. 만약 Domain Model과 Persistence Model이 거의 비슷한 단순한 요구사항에 대한 Model이라면 합쳐도 된다고 생각한다. 하지만 이 두가지의 차이점을 알고 두 Model을 함께 사용하는 것과 모르고 사용하는 것은 큰 차이가 있다고 생각한다.

> [https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux){: target="\_blank" }
>
> [https://i.kakao.com/docs/getting-started-overview#%EC%98%A4%ED%94%88%EB%B9%8C%EB%8D%94-%EC%86%8C%EA%B0%9C](https://i.kakao.com/docs/getting-started-overview#%EC%98%A4%ED%94%88%EB%B9%8C%EB%8D%94-%EC%86%8C%EA%B0%9C){: target="\_blank" }
>
> [https://kr.vuejs.org/v2/guide/index.html](https://kr.vuejs.org/v2/guide/index.html){: target="\_blank" }
>
> [https://ko.reactjs.org/](https://ko.reactjs.org/){: target="\_blank" }
>
> [https://azure.microsoft.com/ko-kr/services/devops/](https://azure.microsoft.com/ko-kr/services/devops/){: target="\_blank" }
>
> [https://www.typescriptlang.org/](https://www.typescriptlang.org/){: target="\_blank" }
>
> [https://ant.design/](https://ant.design/){: target="\_blank" }
>
> [http://www.yes24.com/Product/Goods/3376544](http://www.yes24.com/Product/Goods/3376544){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Test-driven_development](https://en.wikipedia.org/wiki/Test-driven_development){: target="\_blank" }
>
> [https://martinfowler.com/eaaDev/EventSourcing.html](https://martinfowler.com/eaaDev/EventSourcing.html){: target="\_blank" }
>
> [https://www.geeksforgeeks.org/transaction-isolation-levels-dbms/](https://www.geeksforgeeks.org/transaction-isolation-levels-dbms/){: target="\_blank" }
>
> [https://blog.sapiensworks.com/post/2012/04/07/Just-Stop-It!-The-Domain-Model-Is-Not-The-Persistence-Model.aspx](https://blog.sapiensworks.com/post/2012/04/07/Just-Stop-It!-The-Domain-Model-Is-Not-The-Persistence-Model.aspx){: target="\_blank" }
