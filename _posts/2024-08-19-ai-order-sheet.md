---
layout: single
title: AI를 활용한 주문서 생성 자동화
categories:
  - TUTORIALS
tags:
  - spring-ai
  - chatgpt
  - langchain
toc: true
toc_sticky: true
excerpt: "최근 ChatGPT를 비롯한 생성형 AI가 주목받으면서 우리도 AI 스터디를 진행했다. AI에 대해 들어본 적은 있었지만, 실제로 접해본 적은 없어서 스터디를 통해 프롬프트 엔지니어링, RAG, 랭체인 등의 개념을 배우고 이를 어떻게 활용할 수 있는지에 대한 기초를 익힐 수 있었다. 이번 글에서는 우리가 AI를 어떻게 제품에 활용했는지 소개해 보고자 한다. RAG를 사용하거나 복잡한 기술을 도입한 것은 아니지만, AI 초보 개발자도 사용자의 편의를 위해 제품에 AI를 효과적으로 활용할 수 있다는 점에서 의미가 있다고 생각한다. 모쪼록 재미있게 봐주길 바란다."
---

## Intro

이 글은 사내 블로그에 작성한 [AI를 활용한 주문서 생성 자동화: 카카오톡 주문을 키친보드 주문으로](https://spoqa.github.io/2024/08/19/ai-order-sheet.html){: target="\_blank" } 내용을 그대로 가져오면서 나의 블로그의 언어톤에 맞게 변경한 글이다.

최근 [ChatGPT](https://chatgpt.com/){: target="\_blank" }를 비롯한 생성형 AI가 주목받으면서 우리도 AI 스터디를 진행했다. AI에 대해 들어본 적은 있었지만, 실제로 접해본 적은 없어서 스터디를 통해 [프롬프트 엔지니어링](https://en.wikipedia.org/wiki/Prompt_engineering){: target="\_blank" }, [RAG](https://en.wikipedia.org/wiki/Retrieval-augmented_generation){: target="\_blank" }, [랭체인](https://www.langchain.com/){: target="\_blank" } 등의 개념을 배우고 이를 어떻게 활용할 수 있는지에 대한 기초를 익힐 수 있었다.

이번 글에서는 우리가 AI를 어떻게 제품에 활용했는지 소개해 보고자 한다. RAG를 사용하거나 복잡한 기술을 도입한 것은 아니지만, AI 초보 개발자도 사용자의 편의를 위해 제품에 AI를 효과적으로 활용할 수 있다는 점에서 의미가 있다고 생각한다.

모쪼록 재미있게 봐주길 바란다.

## 도입배경

키친보드를 사용하기 전, 매장의 점주님들은 유통사로 식자재를 주문할 때 주로 문자나 카카오톡을 이용했다.

![kakao-order](/assets/images/posts/ai-order-sheet/kakao-order.png)

이 방식에는 여러 가지 불편함이 있다.

- 유통사에서 매장의 주문 내용을 취합하고 정리하기가 어렵다.
- 매장에서 과거 주문 내역을 확인하기 어렵다.
- 유통사가 다수의 매장 주문을 취합하면서 누락되거나 잘못 정리되는 경우가 많다.
- 이로 인해 매장은 주문한 품목을 원하는 시간에 정확히 받지 못할 수 있다.

키친보드는 이러한 불편을 해소하기 위해 주문톡 서비스를 운영하고 있다.

<br/>

하지만 여전히 일부 매장은 카카오톡으로 유통사에 식자재를 주문하고 있다. 유통사는 키친보드의 편리함을 느끼고 이를 적극적으로 사용하지만, 몇몇 매장은 기존의 카카오톡 주문 방식을 고수하고 있는 것이다.

매장에서는 `피망 6개, 양파 2망, 올리브오일 2개 주세요`와 같이 간단히 주문하는 것이 편리할 수 있다. 유통사가 알아서 품목을 해석하고 적절한 제품을 보내주기 때문이다. 하지만 유통사는 매장의 주문을 해석하고 취합하는 데 많은 시간을 소모하게 된다. 더욱이 이러한 지식은 오랜 경험을 바탕으로 작업자들이 처리해 왔기 때문에, 작업자가 바뀌면 배송 실수가 발생할 가능성이 크다.

<br/>

그래서 키친보드는 오래전부터 카카오톡으로 받은 주문을 어떻게 하면 손쉽게 주문톡의 주문서로 변환할 수 있을지 고민해 왔다. 주문톡 서비스를 처음 만들 때도 카카오톡 주문을 주문서로 생성하려는 시도도 물론 있었다. 일정한 패턴만 정의할 수 있다면 카카오톡 주문을 해석하고, 이를 기반으로 주문서를 생성하는 것이 가능할 것으로 생각했기 때문이다.

하지만 우리는 아래와 같이 매장에서 주문하는 다양한 패턴에 좌절하고 만다.

```
피망 6개, 양파 2망, 올리브오일 2개 주세요
```

```
다진김치1  콩나물2개 깻잎2개 계란5판 청양고추1 깐마늘1 다진마늘1 쌈무4개 날치알3개 떡사리1개 모짜치즈1개 버섯1 만두2개 맛소금1 마요네즈1개
다진김치는 중국산으로 주세요 3kg
```

```
맛살-2
치즈떡-3
연근-3
메추리알-3
고구마떡-2
비엔나-3봉지
```

```
콩나물4개 깻잎4개 날치알4개 튀김고구마1개 만두4개 슬라이스치즈1개 오이 3숙주나물 0,5 청양고추1 스위트곤1개 다진마늘1참맛기름1개 설탕1개 계란5 쌈장1개 쌈무6개 당면1개 마요네즈1개 떡사리1개 모짜치즈2개 고추맛기름1 김가루1개 김2개 짜파게티5개입 2 사세바팔로윙스틱1개 미풍1개 물엿1 마카로니1개 감자튀김 1
레몬 4
```

어떤 매장은 품목명과 개수를 띄어쓰기도 하고, 어떤 매장은 품목명과 개수 사이에 `-`를 넣기도 한다. 또 어떤 매장은 `00개`라고 표기하거나, 같은 내용의 주문에서도 다른 표기법을 사용하기도 한다.

결국, 우리는 다양한 주문 패턴을 단순히 조건 처리로 해결하기에는 무리가 있다고 판단했고, 카카오톡 주문을 주문톡으로 자동 변환하는 과제를 이후로 미루기로 했다.

## AI 활용

AI 스터디를 진행하면서 우리는 과거에 좌절했던 카카오톡 주문 해석 문제를 AI로 해결할 수 있다고 생각했다. AI를 활용하면 다양한 패턴의 문자 내용을 우리가 원하는 포맷으로 변환할 수 있고, 이를 바탕으로 주문서를 생성할 수 있을 것으로 생각했기 때문이다.

그래서 우리는 아래와 같이 몇 가지 사례를 기반으로 테스트를 진행했다.

### 사례1

- Prompt
  ```
  Convert the given 'text' into JSON format with 'name' and 'count' as keys. Where 'name' is of type String and 'count' is of type integer.

  text:
  다진김치1  콩나물2개 깻잎2개 계란5판 청양고추1 깐마늘1 다진마늘1 쌈무4개 날치알3개 떡사리1개 모짜치즈1개 버섯1 만두2개 맛소금1 마요네즈1개
  다진김치는 중국산으로 주세요 3kg
  ```

- Output
  ```
  [
    { "name": "다진김치", "count": 1 },
    { "name": "콩나물", "count": 2 },
    { "name": "깻잎", "count": 2 },
    { "name": "계란", "count": 5 },
    { "name": "청양고추", "count": 1 },
    { "name": "깐마늘", "count": 1 },
    { "name": "다진마늘", "count": 1 },
    { "name": "쌈무", "count": 4 },
    { "name": "날치알", "count": 3 },
    { "name": "떡사리", "count": 1 },
    { "name": "모짜치즈", "count": 1 },
    { "name": "버섯", "count": 1 },
    { "name": "만두", "count": 2 },
    { "name": "맛소금", "count": 1 },
    { "name": "마요네즈", "count": 1 }
  ]
  ```

### 사례2

- Prompt
  ```
  Convert the given 'text' into JSON format with 'name' and 'count' as keys. Where 'name' is of type String and 'count' is of type integer.

  text:
  콩나물 8
  깻잎 3
  청양고추 2
  깐마늘 2
  다진마늘 2
  홍고추 1
  양배추 1
  날치알 3
  만두 2
  갖다주세요
  당면 3개추가좀 해주세요 ㅠ
  ```

- Output
  ```
  [
    { "name": "콩나물", "count": 8 },
    { "name": "깻잎", "count": 3 },
    { "name": "청양고추", "count": 2 },
    { "name": "깐마늘", "count": 2 },
    { "name": "다진마늘", "count": 2 },
    { "name": "홍고추", "count": 1 },
    { "name": "양배추", "count": 1 },
    { "name": "날치알", "count": 3 },
    { "name": "만두", "count": 2 },
    { "name": "갖다주세요", "count": 0 },
    { "name": "당면", "count": 3 },
    { "name": "개추가좀 해주세요 ㅠ", "count": 0 }
  ]
  ```

### 사례3

- Prompt
  ```
  Convert the given 'text' into JSON format with 'name' and 'count' as keys. Where 'name' is of type String and 'count' is of type integer.

  text:
  야채 유통사에 넣어주세요.
  새송이-1박스
  숙주나물 -1박스
  깐마늘-2봉지
  팽이버섯-1박스
  대파-1단
  치즈떡-1봉지
  메추리알-1봉지
  불고기햄-1봉지
  맛소금-1봉지
  토마토 케첩-봉지에 들은걸로 2.5킬로?? 그걸로 ?? 9봉지
  ```

- Output
  ```
  [
    { "name": "새송이", "count": 1 },
    { "name": "숙주나물", "count": 1 },
    { "name": "깐마늘", "count": 2 },
    { "name": "팽이버섯", "count": 1 },
    { "name": "대파", "count": 1 },
    { "name": "치즈떡", "count": 1 },
    { "name": "메추리알", "count": 1 },
    { "name": "불고기햄", "count": 1 },
    { "name": "맛소금", "count": 1 },
    { "name": "토마토 케첩", "count": 9 }
  ]
  ```

`개추가좀 해주세요 ㅠ`, `갖다주세요` 와 같이 몇 가지 잘못 해석하는 부분은 있었지만 `토마토 케첩-봉지에 들은걸로 2.5킬로?? 그걸로 ?? 9봉지`를 `{"name"="토마토 케첩", "count"=9}`로 인식한다든지 패턴이 일정하지 않더라도 품목명과 주문수를 원하는 대로 구분해 주는 것들을 보면 AI를 통해 충분히 카카오톡 주문 내용을 해석하여 원하는 패턴으로 변환할 수 있을 것으로 생각했다.

## 주문서 품목 선택

앞에서 우리는 AI를 통해 `품목명`, `주문 건수`와 같이 일관된 양식으로 변환할 수 있다는 것을 알게 되었다. 하지만 이번에는 해석된 품목명을 키친보드에 입력된 유통사의 품목으로 변환하는 데 문제가 있었다.

키친보드에서 다루는 유통사의 주문 품목은 품목명만 있는 것이 아닌 규격과 단위가 존재한다. 예를 들어 같은 `계란`이라는 품목이 있더라도 규격은 `대란`, `중란`, `특란`과 같이 나눠질 수 있고 단위도 `6개입`, `개`, `판`과 같이 나누어질 수 있다. 그래서 아래와 같이 `계란`을 주문하려고 하면 다수의 품목이 검색되어 어떤 품목을 주문해야 할지 시스템이 판단하기 어렵다는 문제가 있다.

![eggs](/assets/images/posts/ai-order-sheet/eggs.png)

또 다른 사례로, 매장에서는 유통사가 입력해 둔 정확한 품목명을 기억하고 주문하지 않는 경우가 많다. 예를 들어 `깐마늘`을 생각해 보자. 유통사는 `깐마늘`을 그대로 저장하지 않고, `마늘/깐(대)`와 같이 유통사의 규칙에 따라 품목명을 저장해두고 주문을 받고 있다.

이 때문에 키친보드에서는 주문톡을 사용하는 매장이 `깐마늘`이라고 검색하더라도, 유통사에서 저장한 `마늘/깐(대)` 품목을 조회할 수 있도록 유사도 검색 기능을 지원하고 있다. (다만 유사도 검색에는 원치 않는 품목이 함께 조회될 가능성이 있다는 단점도 존재한다)

예를 들어 `깐마늘`을 검색했을 때, 아래처럼 다양한 품목들이 조회되는 것을 볼 수 있다.

![search-products](/assets/images/posts/ai-order-sheet/search-products.png)

그래서 우리는 주문할 품목 선택을 시스템이 자동으로 하기보다는 사용자에게 위임하기로 했다. 잘못된 품목으로 주문서가 생성되는 위험보다는 매장에서 원하는 품목을 한 번 더 확인하도록 하는 것이 낫다는 판단에서이다.

다만, 사용자 편의를 위해 입력한 품목명과 가장 유사한 품목을 자동으로 선택하고, 최근에 주문했던 품목을 우선 선택하도록 하여 최대한 편리하게 주문서를 생성할 수 있도록 했다.

## AI 주문서 생성 순서

위에서 말한 내용을 기반으로 사용자가 AI 주문서를 생성하는 순서를 그려보면 아래와 같다.

![ai-order-flow](/assets/images/posts/ai-order-sheet/ai-order-flow.png)

### 주문서 초안 생성

먼저 사용자는 카카오톡 주문 내용을 복사해서 주문서 초안을 생성한다.

![create-draft](/assets/images/posts/ai-order-sheet/create-draft.png)

### 주문 품목 선택

앞서 “주문서 품목 선택 이슈”에서 말한 바와 같이 주어진 품목명이 원하는 주문서 품목과 일치하는지 정확히 알기 힘들기 때문에 생성된 주문서 초안에서 원하는 품목을 선택한다. 다만, 가장 많이 주문하는 품목이 먼저 선택되어 있도록 함으로써 사용자가 품목을 선택하는 행위를 최소화하였다.

![select-product](/assets/images/posts/ai-order-sheet/select-product.png)

### 주문서 확인 및 주문

사용자는 최종적으로 생성될 주문서를 확인하고 주문서를 생성할 수 있다. 이때 원치 않는 품목을 제거하거나 추가하여 AI가 생성해 준 품목을 토대로 기대한 주문서를 생성할 수 있게 된다.

![check-order-sheet](/assets/images/posts/ai-order-sheet/check-order-sheet.png)

## 구현

이제 기술적인 측면을 살펴보자. 사실 AI를 활용한 주문서 생성 기능은 기존의 주문서 생성 기능을 크게 변경하지 않았다. 앞서 설명한 AI 주문서 생성 순서에서, 초안을 생성하는 부분만 추가되었을 뿐이다.

![ai-order-flow-for-dev](/assets/images/posts/ai-order-sheet/ai-order-flow-for-dev.png)

그래서 주문서 초안을 생성하는 흐름도를 조금 더 자세히 그려보면 아래와 같이 그릴 수 있다. 조금 더 자세한 내용은 아래에서 다루어보겠다.

![create-draft-diagram](/assets/images/posts/ai-order-sheet/create-draft-diagram.png)

### Spring AI

AI 스터디를 진행하면서 읽었던 책은 [랭체인으로 LLM 기반의 AI 서비스 개발하기](https://product.kyobobook.co.kr/detail/S000212568407){: target="\_blank" }였다. 이 책은 LLM 기반의 AI 서비스를 예제로 쉽게 따라 할 수 있도록 가이드해 주어, AI 초보자인 저도 손쉽게 학습할 수 있었다. (광고 아님!)

처음 학습할 때 랭체인으로 시작했기 때문에, AI 모델에게 주문 내용을 해석해 주는 기능을 별도의 Python 서버로 구현하는 방안을 고민하기도 했다. 그러나 팀 정책상, 관리 포인트를 불필요하게 늘리지 않기 위해 키친보드 서비스의 서버에서 최대한 해결하는 것이 좋겠다고 판단했다. 그러던 중 발견한 프레임워크가 바로 Spring AI였다.

[Spring AI](https://spring.io/projects/spring-ai){: target="\_blank" }는 AI 엔지니어링에 특화된 프레임워크로, 랭체인만큼 다양한 기능을 제공하지는 않지만, 우리가 구현하고자 했던 카카오톡 주문 내용을 원하는 형식으로 변환하는 데는 부족함이 없었다. 또한, 스프링과 쉽게 통합할 수 있었기에 더할 나위 없는 선택지였다. (자세한 사용법은 [공식 문서](https://docs.spring.io/spring-ai/reference/getting-started.html){: target="\_blank" }를 참고하자!)

### 주문서 초안 생성 로직

추문서 초안 생성을 위한 로직을 보면 아래와 같다.

![create-draft-data-flow](/assets/images/posts/ai-order-sheet/create-draft-data-flow.png)

주문 내용을 해석해 줄 `MessageInterpreter`가 AI 서비스에 해석요청을 보내고 그 결과를 바탕으로 유통사의 품목을 조회해 초안을 생성해 준다는 것을 알 수 있다.

우리는 `MessageInterpreter`의 구현 클래스에서 Spring AI를 활용하였다. 구현 클래스인 `AIMessageInterpreter`를 보면 상당히 단순한 것을 볼 수 있다. Spring AI는 `ChatClient`를 통해 AI 서비스와의 Prompt 통신을 단순화한 것을 볼 수 있다. 또한, `responseEntity` 함수와 같이 응답 결과를 원하는 데이터 구조로 손쉽게 변환할 수 있도록 해주기 때문에 AI 채팅의 결과를 원하는 데이터 객체로 변환하기 위한 복잡하고 번거로운 코드를 작성하지 않아도 된다는 장점도 있다.

Java가 아닌 Kotlin으로 Spring AI를 사용하시는 분들께 드리는 한가지 팁은 `responseEntity`의 반환 타입인 `OrderProduct`를 정의할 때 `@JvmRecord`를 선언해 주지 않으면 오류가 발생하니 이 부분은 기억해 두면 좋겠다.

```kotlin
interface MessageInterpreter {
    fun interpretOrderMessage(message: String): List<OrderProduct>
}

@Component
class AIMessageInterpreter(
    private val openAiChatClient: ChatClient,
    private val aiChatHistoryRepository: AIChatHistoryRepository,
) : MessageInterpreter {
    override fun interpretOrderMessage(message: String): List<OrderProduct> {
        val promptMessage =
            """
            |Convert the given 'text' into JSON format with 'name' and 'count' as keys. Where 'name' is of type String and 'count' is of type integer.

            |text:
            |$message
            """.trimMargin()

        val response =
            openAiChatClient
                .prompt()
                .user(promptMessage)
                .call()
                .responseEntity(parameterizedTypeRef<List<OrderProduct>>())

        saveHistory(message, response)

        return response.entity
    }
    
    // 생략...
}

@JvmRecord
data class OrderProduct(
    val name: String,
    val count: Float,
)
```

### 주문 품목 병렬 조회

위에서 보여준 예시처럼, 매장에서는 한 번에 여러 품목을 주문하는 경우가 많다. 예를 들어, 주문서 품목을 조회하는 데 1초가 걸린다고 가정하면, 30개의 품목을 조회하는 로직을 아래와 같이 구현했을 때, 주문서 초안을 생성하는 데 최소 30초가 소요될 것이다.

```kotlin
fun getOrderSheetDraftProducts(data: OrderSheetDraftProductsSearchData): List<OrderSheetDraftProduct> {
    val orderProducts = aiMessageInterpreter.interpretOrderMessage(data.text)

    return orderProducts.map {
        val products = searchOrderableProducts(data, it.name)

        OrderSheetDraftProduct(
            name = it.name,
            count = it.count,
            foundProducts = products,
        )
    }
}
```

당연한 이야기겠지만, 주문서 초안을 생성하는 데 시간이 오래 걸리면 사용자는 불편함을 느껴 AI 주문서 기능을 더 이상 적극적으로 사용하지 않게 될 수 있다. 이를 방지하기 위해 저희는 병렬 처리 방식을 도입하여 주문서를 조회하고 초안을 생성함으로써, 최대한 짧은 시간 안에 사용자의 요청을 처리할 수 있도록 했다.

아래와 같이 코드를 작성하면 다수의 품목을 조회하더라도 약 1초 만에 전체 품목의 조회 결과를 반환할 수 있다.

```kotlin
fun getOrderSheetDraftProducts(data: OrderSheetDraftProductsSearchData): List<OrderSheetDraftProduct> =
    runBlocking {
        val orderProducts = aiMessageInterpreter.interpretOrderMessage(data.text)

        orderProducts.map {
            async {
                val products = searchOrderableProducts(data, it.name)

                OrderSheetDraftProduct(
                    name = it.name,
                    count = it.count,
                    foundProducts = products,
                )
            }
        }.awaitAll()
    }
```

## 마무리

지금까지 Spring AI를 활용하여 카카오톡 주문을 키친보드 주문서로 자동 생성하는 기능을 구현한 사례를 소개해 보았다. 비록 이번 구현이 단순한 수준이어서 AI 도입 사례로 소개하기에 다소 민망할 수 있지만, 이 과정에서 우리가 가진 문제를 어떻게 해결할지 고민한 점과, AI 도입이 생각만큼 복잡하거나 어려운 일이 아니라는 점을 봐주면 좋겠다.

최근 OpenAI에서 [GPT-4o-mini 모델이 출시](https://openai.com/index/gpt-4o-mini-advancing-cost-efficient-intelligence/){: target="\_blank" }되어, 우리가 구현한 것처럼 경량 모델이 필요한 경우에는 GPT-3.5 Turbo 모델보다 비용 효율적이고 다양한 기능을 사용할 수 있게 되었다. 그래서 앞으로 이미지 기반 주문서 생성 기능도 도입할 계획이다. 이를 통해, 유통사에서 화이트보드나 손 글씨로 작성해 카카오톡으로 보내던 주문 요청을 키친보드로 쉽게 전환할 수 있도록 할 예정이다. (생각보다 변환 결과가 괜찮았다!)

또한, 아직 경험이 부족하여 키친보드의 데이터를 사전 학습시켜 주문서를 생성하는 RAG 방식을 도입하지는 못했지만, 더 나은 기능을 제공할 아이디어가 떠 오른다면, 조금 더 학습하여 RAG 방식을 도입해 AI를 조금 더 풍부하게 활용해 보고 싶다.

또 다른 사용 사례가 생기면, 다음에 더 유익한 글로 찾아오겠다.
