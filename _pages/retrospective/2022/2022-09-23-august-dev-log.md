---
layout: single
title: "2022년 8월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2022-august-dev-log/
---

요즘들어 느끼는 고민은 내가 당연하다고 생각하는 개발지식을 주변동료들도 당연하게 알아야 하는가에 대한 것이다. 이렇게만 말하면 너무 추상적이라 구체적인 예로 적어보면 우리 회사에서는 웹 어플리케이션을 개발하고 있는데 HTTP에 대한 기본적인 지식(예를들면 HTTP 함수는 GET, POST, PUT, DELETE, OPTION이 있고 HEADER와 BODY가 있다와 같은)들을 모르거나 제대로 이해하지 못한채로 사용하는 개발자가 있다고 가정해보자. 이럴 때 HTTP에 대한 기본지식을 모르는 것을 감안하고 계속해서 HTTP에 대한 기본지식을 전달해주어야 하는가? 혹은 HTTP에 대해서 기본적인 지식을 익힐 수 있도록 지속적으로 교육을 진행해야하나?

물론 채용과정에서 회사에 필요한 인재인지를 검증하면 된다고 생각할 수 있다. 하지만 다른 부서의 개발자라면 어떨까? 우리 부서의 개발자는 내가 채용에 관여할 수 있지만 다른부서의 개발자는 내가 채용에 관여할 수 없기 때문에 회사에서 필요로하는 기술력을 보유하고 있는지 검증하며 채용하기 어렵다. 그렇다면 지속적인 교육이나 매 개발시마다 동료 개발자가 모르는 부분을 매번 가이드 하면서 개발을 수행해야 할 것이다. 하지만 스타트업의 특성상 이러한 교육 세션을 주기적으로 가지거나 일정이 바쁜 와중에 매번 가이드를 수행하기는 어려울 수 있다고 생각한다.

하지만 그럼에도 동료 개발자가 모른다고 불평만 할수는 없다고 생각한다. 이미 일어난 일이라면 불평보다는 해결방법을 찾아야 한다고 생각한다. 그래서 고민이다. 상대방에게 필요성을 인지시켜주고 공부하도록 유도는 할 수 있지만 본인이 기술 습득에 대한 의지가 없다면 이 방법도 도움이 되지 않는다 생각이 들기 때문이다. 좋은 방법이 없을까?

아래는 8월동안 정리한 이슈 내용들이다.

## POI setCellType 이슈

엑셀의 cell 에 숫자 입력 시 cell type 이 NUMERIC 이 되는데 이것을 cell.stringCellValue 를 할때 오류가 발생했다. 이유는 [`setCellType`문서](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html#setCellType-org.apache.poi.ss.usermodel.CellType-){: target="\_blank" }에 나와있다.

> If what you want to do is get a String value for your numeric cell, stop! This is not the way to do it. Instead, for fetching the string value of a numeric or boolean or date cell, use DataFormatter instead.

그래서 [DataFormatter](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/DataFormatter.html){: target="\_blank" }를 사용해서 numeric cell을 String value로 변경하게 바꾸었다.

```kotlin
val dataFormatter = DataFormatter()
val cellValue = dataFormatter.formatCellValue(row.getCell(0, CREATE_NULL_AS_BLANK)).trim()
```

## Graphql N+1 문제

최근 개발하면서 Graphql 호출 시 재귀호출을 할 수 있도록 기능을 구현해 두었다. 그러면서 Field Resolver에서 다수의 필드를 호출하도록 하였는데 여기에서 N+1문제가 발생하는 걸 APM을 통해 확인할 수 있었다.

![N+1 problem](/assets/images/retrospective/graphql-n+1-problem.png)

그나마 다행인점은 Spring의 [hibernate batch_size](https://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch04.html){: target="\_blank" }를 설정해둔 덕분에 데이터베이스에서는 N+1 호출이 Graphql Field Resolver의 N+1 문제만큼 많이 발생하지는 않았다.

하지만 그럼에도 불구하고 네트워크 비용으로 인해 API의 전체 응답속도는 느려졌다. 그래서 Field Resolver를 구현한 내용을 제거하고 Graphql 응답 Root에서 자식 field들 모두 만들어준 후 전체를 반환하는 방식으로 변경하였다. 사실 batch loader를 구현해서 사용해도 되는데 전체를 반환하도록 만들어도 부하가 크지 않으므로 굳이 복잡하게 batch loader를 사용하지 않고 전체 데이터를 만들어서 반환하도록 하였습니다.

그 결과 아래와 같이 N+1 문제가 해결된걸 확인할 수 있습니다.

![N+1 problem resolve](/assets/images/retrospective/graphql-n+1-problem-resolve.png)

## H2 UUID 타입 정의

인메모리 DB로 H2 데이터베이스를 사용할때 JPA의 컬럼 타입을 UUID로 정의할때 제대로 동작하지 않는 문제가 있었다. 해결방법은 아래와 같이 `columnDefinition`를 적어주면 된다. 이전에도 해당이슈를 겪었던것 같은데.... 기억력이 점점 안좋아 지는 것 같다.

```kotlin
@Id
@Column(columnDefinition = "uuid")
private val id: UUID = UUID.randomUUID()
```

## 느낌/이미지별 영어단어 사이트

느낌이나 이미지별로 영어단어를 찾고 싶을 때 아래 사이트를 이용하면 도움이 될것 같아 적어둔다.

[https://www.englishplus.co.kr/data/engname.asp?CATE_DIV=03](https://www.englishplus.co.kr/data/engname.asp?CATE_DIV=03){: target="\_blank" }

## 직장인 블로그

직장인과 관련한 내용을 적는 좋은 블로그가 있어서 적어둔다.

[https://topofcomp.tistory.com/](https://topofcomp.tistory.com/){: target="\_blank" }

## 코틀린 playground

코틀린 코드를 간단하게 웹 환경에서 실행해 보고 싶다면 playground를 사용해 보면 좋다.

[https://play.kotlinlang.org/](https://play.kotlinlang.org/){: target="\_blank" }

## Azure 장애

30일에 Azure 장애가 발생했다. 2018년도에 AWS의 한국 리전에서 대규모 장애가 있었는데, 클라우드를 사용하는 비율이 높아지면서 클라우드 서비스가 단일 장애 지점이 되어가고 있지 않나 싶다.

[https://www.anoopcnair.com/azure-outage-because-of-dns-issues-ubuntu/](https://www.anoopcnair.com/azure-outage-because-of-dns-issues-ubuntu/){: target="\_blank" }

## 부하 테스트 도구

부하 테스트 도구로 [nGrinder](https://naver.github.io/ngrinder/){: target="\_blank" }나 [jMeter](https://jmeter.apache.org/){: target="\_blank" } 정도만 알고 있었는데, [k6](https://k6.io/){: target="\_blank" }와 [Locust](https://locust.io/)도 있다는걸 알게되었다. 한번 써봐야겠다.
