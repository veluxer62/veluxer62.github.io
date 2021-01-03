---
layout: single
title: "2020년 7월 4주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

최근 백엔드 시스템을 MSA로 전환하기 위해 다들 노력하고 있다. 현재 내가 맡고 있는 시스템도 기존의 모놀리식 시스템을 MSA로 전환하기 위해서 동료들과 어떻게 시스템을 분리하면 좋을 지 논의하고 어떤 방식으로 MSA 시스템을 구성할지 고민하고 있다. 다행히도 회사에 [마이크로 서비스 패턴](http://www.yes24.com/Product/Goods/86542732){: target="\_blank" } 서적이 있어서 시스템 구성 전에 부지런히 읽고 있다. 정말 읽기 잘했다고 생각이 들정도 내용들이 많이 도움이 되었고 다 읽고 나서 개인적으로 책을 구매해서 두고두고 다시 볼 예정이다.

회사의 조직 개편이 있어서 기존에 소속되어 있던 커머스 부서가 아닌 플랫폼 부서로 소속이 변경되게 되었다. 맡은 업무는 변경된건 없어서 다시 변화에 적응해야할 필요가 없어서 다행이긴 하지만 이전 회사에서 심심하면 조직 개편을 경험했던 터라 개인적으로는 조직 개편은 자주 안했으면 좋겠다.

## Spring DI 배열 주입

최근에 동료의 코드리뷰를 하면서 Spring에서 여러개의 빈을 배열형태로 주입할 수 있다는 것을 알게 되었다. 그동안 이러한 방식으로 사용할 일이 없어서 배열형태로 주입할 수 있다는 것을 몰랐었는데, 새로운 사실을 알게 되어서 좋았다! 앞으로 이런 형태로 사용할 일이 있다면 사용해봐야 겠다. 근데 배열 형태의 빈을 주입할 일이 많이 있을까 싶기는 하다.

## Octotree

[Octotree](https://chrome.google.com/webstore/detail/octotree/bkhaagjahfmjljalopjnoealnfndnagc){: target="\_blank" }는 Web 환경의 Github 사용 시 코드리뷰를 보다 쉽게 할 수 있도록 지원해주는 Chrome extension이다. 회사에서는 Github을 이용해서 코드관리를 하고 있는데, 코드리뷰나 레파지토리에서 파일 검색 시 파일을 찾는게 불편했는데 Octotree를 사용하게 되면서 보다 쉽게 디렉토리 내 파일에 접근할 수 있게 되었다! 조만간에 사용법에 대해서 글하나 작성해 봐야 겠다.

## Axon

[Axon](https://axoniq.io/){: target="\_blank" }은 EventSourcing과 CQRS를 보다 손쉽게 사용하기 위한 Framework 이다. Spring환경에서 손쉽게 사용할 수 있어 이전부터 한번 튜토리얼이라도 해봐야지 했었는데 이제서야 한번 만들어본다. 러닝커브가 높지 않고 도메인 코드와 손쉽게 분리가 가능하다면 현재 진행중인 MSA로의 전환 시 도입해 보고자 고민 중이다. Axon Server를 사용해야 한다는 점과 Domain 코드에 Axon Framework 의존성이 생기는 구조 때문에 도입은 조금 고민이 된다. Axon Server를 사용하지 않고 Domain 코드와 독립적으로 사용할 수 있는 방법을 찾아봐야 겠다.

## IntelliJ multiple selection

Axon Tutorial 영상을 보다가 영상속에서 [Intellij mutiple selection](https://www.jetbrains.com/help/rider/Multicursor.html#add-delete-clone-caret){: target="\_blank" }을 사용하는 법을 보고 앞으로 익혀두면 좋을 것 같아서 단축키와 사용방법을 알아 보았다. 진즉에 알았다면 더 좋았을 것을 ㅠㅠ 자주 애용 해야 겠다!

## vaadin

이것도 Axon demo 소스를 통해 알게 되었는데, [Vaadin](https://vaadin.com/){: target="\_blank" }자바코드를 이용하여 Web UI를 만들 수 있는 라이브러리 이다. HTML과 Javascript를 잘 몰라도 Web UI를 만들 수 있다는 장점은 백엔드 개발자에게 큰 매력인 것 같다. 관리자 UI를 구성할 때 사용하면 좋을 것 같다. 특히 Reactive하게 화면을 갱신할 수 있다는 점은 너무 매력적인 것 같아서 이것도 간단한 튜토리얼을 만들어 보면서 연습을 해두면 좋을 것 같다!

## 오브젝트 디자인 스타일 가이드

동료개발자가 [오브젝트 디자인 스타일 가이드](https://wikibook.co.kr/odsg/){: target="\_blank" }책을 소개해 주었다. 최근에 내가 개발할 때 항상 염두해두고 고민하던 부분들을 다루고 있어서 코드 작성에 많은 도움을 받을 수 있지 않을 까 싶어서 회사에 구매요청 하고 공부해보면 좋을 것 같다. 다음주에 구매 신청해야겠다.

## kotlin test

얼마전에 개발자 채용 과제 코드를 리뷰하면서 지원자 분 중 [Kotlin Test](https://www.kotlinresources.com/library/kotlintest/){: target="\_blank" }를 사용하신 분이 계셨다. BDD 테스트를 작성하셨는데 가독성이 너무 좋고 코드가 간결해서 앞으로 테스팅에 적용해서 사용하면 어떨까 생각된다. 사용사례가 만지 않은거 같지만 하다가 막히는 부분은 Junit으로 작성 하면 되니 부담도 적고 적용해 볼 만한 것 같다!