---
layout: single
title: "2020년 10월 4주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

이전에 만들었던 SKU 서비스를 Inventory 서비스로 명칭을 변경하고 Repository를 새롭게 생성해서 코드를 다시 짜고 있다. 전주에 언급했었지만 테스트코드가 너무 디테일하게 짜여져 있어서 코드의 변경이 쉽지 않는데, 요구사항이 많이 바뀐 상황이라 테스트코드를 변경해 가면서 코드를 짜는 것 보다 기존 코드를 참고하면서 새롭게 구현하는게 속도가 더 빠를 것이란 생각이 들어서 이러한 선택을 하게 되었다. 처음 SKU 서비스를 만들 때 요구사항을 정의하고 도메인 모델을 만들 때 많은 논의를 하면서 시간을 많이 썼었는데, 이번에는 어느정도 도메인 지식을 가지고 있는 상태이고 기존에 코드를 참고할 수 있기 때문에 어플리케이션을 구성하는 코드는 손쉽게 만들 수 있어서 프로젝트 설정은 금방 끝낼 수 있었다. 테스트 코드는 최대한 통합테스트 위주로 하고 필요하다면 유닛테스트를 섞어주면서 코드의 검증력을 높이면서 유연성을 가지도록 해봐야 겠다.

아래는 지난 주 동안 내가 새롭게 알게 되었거나 생겼던 이슈의 내용을 정리한 것이다.

## Undertow

[Undertow](https://undertow.io/){: target="\_blank" }는 Jboss 재단에서 만든 웹 서버이다. Spring Boot에서 Tomcat과 함께 Undertow도 Embedded로 WAS를 제공해 주는데 Tomcat에 비해 성능상 이점이 많다는 벤치마크 결과가 있어 이번에 인벤토리 서비스에서 사용하는 WAS로 채택해서 사용해 보기로 하였다.
벤치 마크 링크 : [https://dev.to/azure/spring-boot-performance-benchmarks-with-tomcat-undertow-and-webflux-4d8k](https://dev.to/azure/spring-boot-performance-benchmarks-with-tomcat-undertow-and-webflux-4d8k){: target="\_blank" }

## 인텔리제이 와일드 카드 Import Disable

kotlin 문법을 체크해주는 라이브러리인 ktlin에서 import 구문에서 `*`(와일드카드)사용을 제한하고 있다.

인텔리제이에서 기본설정으로 사용한다면 아마 자동으로 `*`가 들어가는 경우를 볼 수 있을 텐데 자동으로 `*`가 적히지 않도록 하기 위해서는 `Editor` -> `Code Style` -> `Kotlin`에서 `import` 탭을 선택한 후 모두 `Use single name import`를 선택해주면 된다.

![setting](/assets/images/retrospective/intellij-import-setting.png)

## mock mvc 인코딩 문제

Java example

```java
this.mockMvc = MockMvcBuilders.webAppContextSetup(ctx)
    .addFilters(new CharacterEncodingFilter("UTF-8", true))  // 필터 추가
    .alwaysDo(print())
    .build();
```

```java
class Sample {
  @Bean
  MockMvcBuilderCustomizer utf8Config() {
    return builder ->
        builder.addFilters(new CharacterEncodingFilter("UTF-8", true));
  }
}
```

위와 같은 구문을 Kotlin에서 작성하면 컴파일 에러가 발생한다. 해당 오류과 관련한 이슈는 [https://youtrack.jetbrains.com/issue/KT-31559](https://youtrack.jetbrains.com/issue/KT-31559){: target="\_blank" }링크를 참고하자

그래서 해결한 방법은 application.properties에서 설정값을 넣어주고 프로파일을 설정해서 하는 방식으로 했다.

```yaml
server:
  servlet:
    encoding:
      force: true
```

## jacoco 이슈

통합테스트를 하니까 jacoco의 line coverage가 제대로 동작하지 않는다. Intellij에서 실행하는 테스트 커버리지에서는 라인커버리지가 잡히는데 jacoco 커버리지 결과에는 잡히지 않아서 그냥 branch 커버리지만 잡히도록 설정해 버렸다. 라인커버리지를 다시 설정해보고 되지 않으면 과감히 브런치 커버리지만 사용해야 겠다. 솔직히 브런치 커버리지만으로도 내 테스트 코드가 구현코드를 잘 검증하고 있는지 어느정도 볼 수 있다고 생각해서 적절한 타협점이라 생각한다.

## SonarQube

회사 동료분께서 현재 회사의 지원자분들의 과제 검수 시 소나큐브를 사용해서 코드의 퀄리티를 체크하는 것을 보여주셨다. 손수 코드를 봐가면서 검증하는 것보다 빠르게 스켄한 후 어느정도 지원자를 거르는 용도로 사용하기에 좋을 것 같아 보인다. 내 코드도 한번...검증해봐야지... 무섭다
[소나큐브와 관련한 블로그](https://www.popit.kr/내코드를-자동으로-리뷰해준다면-by-sonarqube/){: target="\_blank" }참고

## nGrinder

이번에 nGrinder를 사용할 일이 생겨서 서버를 세팅해보았다. 이전 회사에서는 EC2에다가 세팅해서 테스트를 했었는데, 현재 회사에서는 도커도 조금 배웠겠다, 나중에 쿠버네티스에 배포까지 생각해서 docker-compose로 서버를 세팅해보기로 하였다.

참고한 사이트는 아래와 같다.
[https://brownbears.tistory.com/25?category=168280](https://brownbears.tistory.com/25?category=168280){: target="\_blank" }

사용한 docker-compose template

네트워크 때문에 좀 헤매긴 했는데 다행히 잘 동작한다. 근데 메모리를 너무 적게 줘서 그런지 스레드를 좀 크게 주면 controller가 죽어버린다ㅠㅠ

```yaml
version: "3.1"

services:
  controller:
    image: ngrinder/controller:3.4
    ports:
      - 9999:80
      - 16001:16001
      - 12000-12009:12000-12009
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    networks:
      - ngrinder-network

  agent-1:
    image: ngrinder/agent:3.4
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    command:
      - controller:80
    networks:
      - ngrinder-network

  agent-2:
    image: ngrinder/agent:3.4
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    command:
      - controller:80
    networks:
      - ngrinder-network

  agent-3:
    image: ngrinder/agent:3.4
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    command:
      - controller:80
    networks:
      - ngrinder-network

networks:
  ngrinder-network:
```

## EKS Pot 고정 IP 설정

특정 외부 서비스가 IP로 방화벽을 설정해야 하는 경우 EKS를 사용한다면 pot에 고정 IP를 사용할 수 없기 때문에 다른 방식으로 우회를 해야 한다. 그중 한 방법이 NAT를 이용하여 IP를 고정하고 내보내도록 AWS에서 설정하는 방법인데, 해당 방법은 자세히 공부해서 한번 블로그로 적어봐야겠다.