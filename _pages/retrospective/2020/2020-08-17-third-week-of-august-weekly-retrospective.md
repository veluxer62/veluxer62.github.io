---
layout: single
title: "2020년 8월 3주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-third-week-of-august-weekly-retrospective/
---

이번주는 SKU 도메인과 관련하여 비지니스 로직과 구현과 관련해서 많은 고민을 하였다. 현재 회사에서 사용하고 있는 SKU라는 개념이 일반적으로 물류 도메인에서 사용하는 SKU와는 조금 다른 의미로 사용되고 있다. 하지만 정확하게 의미가 맞지 않다고 해서 SKU라는 도메인명을 바꾸기에는 전사적으로 소통비용이 발생하고 우리 회사만의 SKU로 사용하자는 의견이 있어 SKU라는 이름을 변경하지 않고 그대로 사용하기로 하였다. 맞다 틀리다 보다 적당한 선에서 타협하는게 좋아 보이긴 하다.

앞에서 설명한 SKU 도메인관련 서비스를 개발하는데 영속성 모델과 도메인 모델을 분리해서 개발하고 있다. 개인적으로 영속성 모델과 도메인 모델은 다른 목적으로 정의되는게 맞다고 생각한다. 코드가 많아지는 단점이 있긴 하지만 JPA의 Entity 클래스는 여러 제약사항을 가지고 있어 도메인 모델이 가진 모든 요구사항을 내포하고 있긴 쉽지 않기 때문이다. 하지만 내가 도메인 모델과 영속성 모델을 분리해서 구현해본 경험이 많지 않고, JPA의 여러 기능이 Entity 클래스를 도메인 모델처럼 사용하였을 때 편리하게 되어 있는 기능들이 많이 있어서 아키텍쳐를 다시 고민해보아야 하나 수없이 고민했다. 결국 처음 생각했던 대로 도메인 모델과 영속성 모델을 분리해서 개발하기로 하였고 개발이 끝나고 나면 해당 개발 내용에 대한 회고를 적어보아야 겠다.

아래는 이번주에 내가 새롭게 알게 되었거나 고민했었던 내용들이다.

## MockRestServiceServer

회사에서 동료분이 테스팅 시 [MockRestServiceServer](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/client/MockRestServiceServer.html){: target="\_blank" }를 사용하시는 코드를 리뷰한 적이 있다. 평소에 Mockito를 애용하고 있는데, restTemplate을 mocking 할때도 Mockito를 주로 사용하였다. `MockRestServiceServer`는 RestTemplate에 한정적으로 사용하긴 하지만 Mockito에 비해 좀더 풍성하게 테스팅이 가능하다는 장점을 가진다. 나중에 사용할 기회가 생기면 써봐야겠다.

## editorconfig

프로젝트내에서 코드 컨벤션을 통일할 수 있는 방법을 찾다가 [editorconfig](https://editorconfig.org/){: target="\_blank" }라는 것이 있다는 것을 알게 되었다. (이전에 본적이 있는데 잊어버린것 같다..) editorconfig는 vim, intellij, visual studio, notepadd++, emacs, webstorm등 수많은 에디터들이 지원하고 있고 `.editorconfig`파일을 통해 간단하게 설정할 수 있다. Intellij에서는 설정된 code convention에 맞게 자동으로 `.editorconfig`를 생성해주는 기능을 제공하니 사용해보면 좋을 것 같다. [https://www.jetbrains.com/help/idea/configuring-code-style.html](https://www.jetbrains.com/help/idea/configuring-code-style.html){: target="\_blank" }

## ktlint

프로젝트 내에서 모든 팀원들이 Kotlin으로 작성된 코드를 컨벤션에 맞추도록 강제할 수 있는 방법이 있을까를 고민하다가 찾은 라이브러리가 [ktlink](https://github.com/jlleitschuh/ktlint-gradle){: target="\_blank" }이다. 플러그인을 gradle에 추가하면 빌드 시 자동으로 스타일 검사를 수행해 준다. 동료들에게 `./gradlew addKtlintCheckGitPreCommitHook` 명령어를 써서 pre-commit hook을 등록하도록 하면 PR 전에 스타일을 맞춰서 코드가 원격저장소에 병합되도록 강제할 수도 있다.

## pre-push

push 전에 모든 테스트 코드를 실행하는 것을 강제할 수 있다면 테스트가 실패하는 코드가 Pull Request 되는 일을 방지할 수 있다. 방지할 수 있는 방법을 찾은게 hook을 이용하는 방법인데 아쉽게도 node 진영의 [husky](https://github.com/typicode/husky){: target="\_blank" }와 같이 hook을 공유하는 기능을 찾을 수 없어 gradle로 pre-hook을 생성하는 기능을 만들어 버렸다. [https://gist.github.com/veluxer62/f6dcb4dee35e55a52ed4a8f8aaba6254](https://gist.github.com/veluxer62/f6dcb4dee35e55a52ed4a8f8aaba6254){: target="\_blank" }

## Kotlin 상태값이 없는 class의 equals 구현

현재는 data class를 사용하기로 하였지만 그전에 상태값이 없는 class의 euqals를 어떻게 구현할지 고민한 적이 있었다. 지금 생각해보면 바보같을 순 있지만 당시에는 엄청 고민을 했었다. class type만으로 같다고 보면 되는건지 등등. 결국 Kotlin의 object class로 구현하기로 하였다. 어차피 상태가 없으니 싱글톤으로 클래스가 유지되어도 될테고 그렇다면 euqals를 복잡하게 구현하지 않아도 되기 때문이다.

## Rest API convention

이번주 백엔드 개발자 회의 때 Rest API의 convention에 대한 논의를 하였다. 이전에는 GraphQL을 사용하였는데 MSA로 전환하기로 결정하면서 백엔드 서버끼리Rest 방식으로 통신하기로 했기 때문이다. Restful 하게 사용하는 것은 무리가 있다고 판단했고 좀 덜 엄격하게 적용하면서 API의 행위를 좀더 잘 표현할 수 있는 방식이 무엇이 있을까 고민을 하였었다. 인상적인 대안으로는 Goole에서 도입한 [custom method](https://cloud.google.com/apis/design/custom_methods?hl=ko){: target="\_blank" }였는데, `:`(콜론)을 이용하여 행위를 표현해주는 방식이었다. `.`(점)이나 `-`(하이픈)과 같이 기존 URL과 혼동이 되지 않으면서 Encoding에 영향을 받지 않는 문자열로 제격인 것 같다.

## Slack GitHub Subscribe

기본적으로 Gihub의 특정 Repository를 Watch(구독)하고 있으면 이슈등록이나 PR 등과 같은 이벤트 발생 시 알림이 메일로 전송된다. 사내에서 메신저로 Slack을 사용하는데 [pull reminder](https://pullreminders.com/){: target="\_blank" }를 이용하여 PR요청을 좀더 잘 인지할 수 있도록 하고 있다. 이러한 방법 외에도 repository 자체를 구독하게끔 해서 우리가 진행중인 프로젝트의 Github에서의 이벤트를 Slack으로 받아 볼 수 있는 방법이 있다. 바로 Slack App중 `github`인데, `github`에서 제공하는 많은 명령어 중 `/github subscribe owner/repository`를 입력하면 해당 채널에 입력한 Repository에서 발생하는 이벤트들을 알려준다.

## code owners 설정

git hub에 code owner를 설정할 수 있고 code owner로 설정되면 PR시 반드시 owner의 승인을 강제하도록 할 수 있다.

[https://docs.github.com/en/enterprise/2.15/user/articles/about-code-owners](https://docs.github.com/en/enterprise/2.15/user/articles/about-code-owners){: target="\_blank" }

## JPA entity isNew logic

Kotlin으로 JPA Entity 클래스를 정의할 때 ID를 `var id: Long? = null`와 같이 nullable 하도록 정의하는 예제 코드들을 쉽게 볼 수 있다. 틀린 코드가 아니다. JPA repository의 `save` 함수에서 Entity의 insert 조건이 ID가 null인 경우이기 때문이다. 하지만 나는 nullable ID를 가급적이면 사용하지 않았으면 한다. 왜냐하면 Primary key는 null 값을 가질 수 없기 때문이다. 그래서 대안으로 0을 넣는 방법이 있다. 관련해서는 블로그로 별도로 정리해봐야 겠다.

## Spring Data domain event

JPA 문서를 보다가 우연히 [Publishing Events from Aggregate Roots](https://docs.spring.io/spring-data/jpa/docs/2.3.3.RELEASE/reference/html/#core.domain-events){: target="\_blank" } 기능을 우연히 알게 되었다. Entity의 save에 대한 이벤트를 발생 시키고 Listener를 두어 해당 이벤트를 처리하도록 할 수 있는 기능인데 현재 개발중인 프로젝트에 AOP와 함께 적용을 검토해보아야 겠다.

## spring boot docker

Spring APP을 docker 이미지로 생성해서 배포하는 방식으로 배포전략을 생각하고 Dockerfile을 작성하려고 했었는데 동료분께서 Gradle 명령어중에 docker 파일을 생성하는 명령어가 있다고 해주셨다. 바로 `./gradlew bootBuildImage` 명령어이다. [BootBuildImage](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/api/org/springframework/boot/gradle/tasks/bundling/BootBuildImage.html){: target="\_blank" }는 `Spring 2.3.0` 버전부터 추가된 명령어이다. 아직도 내가 모르는 기능이 참 많은거 같다.

## JPA `@OneToMany`

JPA에서 `@OneToMany`사용 시 겪었던 이슈에 대한 내용을 적어본다.

1. 자식 Entity는 부모 키를 nullable로 해야한다. 이부분은 내가 몰라서 그런지 모르겠는데 기본설정으로 하면 nullable이 아닌 경우 오류가 발생한다. 매커니즘이 null로 먼저 자식 Entity를 등록한 후 update로 부모 키를 할당해 주는 방식인가보다.

2. `orphanRemoval=true` 옵션을 주지 않으면 부모 키가 null인 데이터가 남아 있다. 이부분은 어느정도 예상했던 부분이긴 한데 1번 항목과 연관성이 있다.

3. `joinColumn`에 `nullable=false`를 주면 update 시 동일한 데이터가 1줄 더생긴다. jpa에서 save 함수를 통한 데이터 갱신 시 자식 엔티티들은 변경사항이 없더라도 새롭게 추가된 후 기존데이터를 삭제하는 방식으로 동작한다. 이때 부모 키가 nullable하지 않도록 `joinColumn`에 `nullable=false` 옵션을 주게 되면 기존데이터가 삭제되지 않고 남아 있게 되면서 의도치 않게 자식 Entity가 2개가 존재하는 현상이 발생한다.

위와 같은 이슈로 save 함수를 이용하는 것보다 dirty check 사용하는 것이 나아보인다. 연관관계는 변경사항이 있는 경우에만 update문이 실행되기 때문이다. 하지만 도메인의 변경사항을 그대로 저장하는 기능만 수행하도록 `save` 함수만 사용하려고 하였는데 dirty check을 사용하려면 entity 모델에 함수를 추가하여야 한다.

## flyway with DataJpaTest

flyway 적용 후 DataJpaTest를 실행하면 오류가 발생하는 것을 볼 수 있었다. 이유는 H2 데이터베이스에서의 DDL 문법과 Maria DB에서의 DDL 문법이 다르기 때문이다. H2에서 mode=mysql을 사용하여 해결할 수 있다고 나와있으나 DataJpaTest에서 해당 옵션을 추가해 보았지만 동일하게 동작하지 않았다. 일단 임시로 DataJpaTest에서는 flyway가 적용되지 않도록 `@DataJpaTest(showSql = true, properties = ["spring.flyway.enabled=false", "spring.jpa.hibernate.ddl-auto=create-drop"])` 이렇게 정의해 두었지만 flyway에서 schema가 제대로 적용되었는지 테스팅이 되지 않기 때문에 좋은 방법은 아닌 것 같다. 해결방법이 있는지 찾아봐야 겠다.
