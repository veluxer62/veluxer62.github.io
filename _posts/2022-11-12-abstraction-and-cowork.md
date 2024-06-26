---
layout: single
title: "추상화와 협업"
categories:
  - EXPLANATION
tags:
  - abstraction
  - cowork
  - 객체지향의 사실과 오해
toc: true
toc_sticky: true
---

이 글은 [객체지향의 사실과 오해](http://www.yes24.com/Product/Goods/18249021){: target="\_blank" }의 내용 중 추상화와 관련된 내용을 읽으면서 좀 더 나은 협업을 위한 나의 생각을 사내 워크샵에서 공유하게 되었고 그 내용을 옮겨 적은 글이다.

## 추상화

[객체지향의 사실과 오해](http://www.yes24.com/Product/Goods/18249021){: target="\_blank" }에서는 추상화에 대해 아래와 같이 설명하고 있다.

> 어떤 양상, 세부사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 생략하거나 감춤으로써 복잡도를 극복하는 방법입니다.
>
> 복잡성을 다루기 위해 추상화는 두 차원에서 이루어집니다.
>
> 첫 번째 차원은 구체적인 사물들 간의 공통점은 취하고 차이점은 버리는 일반화를 통해 단순하게 만드는 것입니다.
>
> 두 번째 차원은 중요한 부분을 강조하기 위해 불필요한 세부사항을 제거함으로써 단순하게 만드는 것입니다.

결국 추상화의 목적은 복잡성을 이해하기 쉬운 수준으로 **단순화** 하는것이다.

## 일상에서의 추상화

개발자들 사이에서는 **추상화**는 아주 익숙한 단어이고 자주 사용하는 단어이다.

하지만 나는 추상화가 우리가 느끼던 느끼지 못하던간에 우리의 일상에서 자연스럽게 사용되고 있기에 개발자들만의 것이라고는 생각하지 않는다. 우리의 뇌는 용량이 아주 작아서 일상 생활에서 마주하는 모든 세부사항을 모두 기억하지 못한다고 한다. 우리 자신도 모르게 뇌는 구체적이고 자세한 일상에서의 정보들을 단순화하여 기억하기 쉬운 정보를 저장하고 필요할 때 해당 정보를 쉽게 찾아서 가져온 후 구체적인 정보들을 기억 저편에서 다시 찾아오는 형태를 가진다고 한다.

즉, 이 글을 읽고 있는 여러분의 뇌에서는 이 글의 내용들을 추상화해서 저장하고 있으며 나중에 기억을 되찾을 때 키워드 또는 이미지, 사례 등 여러분의 방식대로 잘 추상화된 정보를 기억속에서 끄집어내 그 정보를 기반으로 세부사항들을 기억 저편에서 찾아 구체화 하는 것이다.

그럼 우리가 일상에서 볼 수 있는 구체적인 사례를 들어보자. 바로 지하철 노선도이다. 하루에 적어도 두번, 혹은 그 이상 보게되는 지하철 노선도는 우리의 일상에서 볼 수 있는 가장 대표적인 추상화의 사례로 들 수 있다.

우선 아래 서울 지하철 노선도를 보자.

![seoul-subway-line-map](/assets/images/posts/abstraction-and-cowork/seoul-subway-line-map.png)

서울은 아니지만 최초 지하철 노선도는 위 그림과 유사하게 실제 지도를 기반으로 표시하였다고 한다. 지하철 노선도가 실제 지도를 기반으로 했기 때문에 정확한 위치와 거리를 볼 수 있어 현재 내가 어느 위치에 있는지 등을 정확하게 알 수 있다는 장점이 있기에 최초 지하철 노선도를 표현할 때 위와 같이 표현했다고 한다.

하지만 실제 승객들은 이 지도를 보고 자신의 목적지를 찾는데 어려움을 느꼈다고 한다. 서울의 대표 지하철역인 강남역을 위 지도에서 찾아보자. 아마 한참을 들여다봐야 어디인지 찾을 수 있을 것이다. 그마저도 자주 가본 곳이기 때문에 찾을 수 있지 처음 들어보는 역의 경우에는 한참을 찾아봐야 할 것이다.

반면 아래 지하철 노선도를 보자

![seoul-subway-line-map2](/assets/images/posts/abstraction-and-cowork/seoul-subway-line-map2.png)

해당 노선도에서 강남역을 찾아보자. 서울 지하철 노선도가 다소 복잡해 찾는게 어려울 순 있지만 아마 처음 보여준 지하철 노선도 보다는 훨씬 빠르게 강남역을 찾을 수 있을 것이다. 1933년 [해리 벡](https://en.wikipedia.org/wiki/Harry_Beck){: target="\_blank" }이 고안한 이 노선도는 지형과 축적은 무시하고 역 사이의 연결성에만 집중하였다고 한다. 그 결과 승객들은 지하철을 탈 때 나의 목적지가 어디이고 어디서 환승을 해야할지 손쉽게 알 수 있게 되었다고 한다.

그럼 처음 보여주었던 지도와 같이 지형이나 거리에 대한 정보는 중요하지 않은 것일까? 그렇진 않다. 하지만 지하철 노선도에서 중요한 것은 연결 즉, 지하철을 갈아타는 것이 중요한 것이다. 지하철 노선도의 목적에 맞게 불필요한 정보는 과감하게 버리고 목적에 필요한 정보들을 부각시킨 결과 90년 가까이 이 지하철 노선도는 여전히 우리가 이용하고 있는 지하철에서 사용되고 있다.

## 추상화와 협업

이제 본론으로 들어가서, 그렇다면 이러한 추상화를 우리가 수행하고 있는 협업 방식에 어떻게 활용하면 좋을까?

추상화의 정의를 다시 한번 보자.

> 어떤 양상, 세부사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 생략하거나 감춤으로써 복잡도를 극복하는 방법입니다.

좀 더 짧게 표현하자면 "**복잡함**을 좀 더 이해하기 쉽게 **단순화**하는 방법"이 추상화라고 표현할 수 있겠다. 그럼 아래와 같이 질문을 다시 해보겠다.

**_우리가 협업을 잘하기 위해 복잡한 협업 방법을 단순화 하는 방법이 무엇이 있을까?_**

위 질문은 이 글을 작성하게된 이유이고 우리가 함께 고민했으면 하는 주제이다.

우리가 지금도 수행하고 있는 협업을 좀 더 잘하기 위해서 어떻게하면 목적에 맞게 업무를 단순화하고 좀 더 나은 제품을 만드는 팀으로 거듭날 수 있을 지 고민해 보았으면 한다.

## 협업 사례

구체적인 사례를 들어보자. 협업에 필요한 요소는 많지만 대표적으로 직군에 관계없이 우리가 가장 많이 활용하고 있는 `문서`, `회의`, `커뮤니케이션`에 대한 내용만 다루어 보겠다. 소개된 사례가 당연하다고 생각될 수 있다. 하지만 우리가 협업을 하면서 심심치않게 겪게되는 아쉬운 사례이기도 하고 경각심을 가지고 우리의 업무 방식을 되돌아 본다는 관점에서 유의미한 사례가 될 것이라 생각된다.

### 문서

우리가 문서를 작성하는 이유는 무엇일까? 가장 큰 이유는 정보의 전달일 것이다. 그렇다면 정보가 많으면 많을수록 좋은 것일까? 나는 목적에 따라 정보가 많으면 많을수록 좋을 수도 혹은 좋지 않을 수도 있다고 생각한다. 목적에 맞지 않은 과한 정보는 오히려 문서의 가독성을 해치고 목적을 흐리게 한다. 지하철 노선도와 같이 목적에 맞는 정보에 집중하고 버릴 정보는 과감하게 버림으로써 단순화하기 위한 노력을 해야한다.

예를 들어 설명해보겠다. 아래 문서는 `백엔드의 작업 현황 및 우선순위`를 챕터 내/외부에 공유하기 위한 문서이다.

- 문서 1

| 작업                                     |      작업자      | 우선순위 | 예상개발완료일 |
| :--------------------------------------- | :--------------: | :------: | :------------: |
| 엑셀 다운로드 포멧 추가                  |     개발자A      |    1     |   2022-10-27   |
| 품목 정보 내 품목 코드 추가              |     개발자B      |    1     |   2022-10-29   |
| 상품 정보의 상품 수정 및 이력 저장       |     개발자C      |    1     |   2022-11-04   |
| 매니저 추가                              | 개발자D, 개발자E |    2     |   2022-11-10   |
| 거래처에서 매장에 별칭 추가              |     개발자E      |    2     |   2022-11-13   |
| 관리자에서 거래처와 매장 직접 연결       |     개발자A      |    2     |   2022-11-11   |
| 주문서의 수정 이력 노출                  |     개발자C      |    3     |   2022-11-20   |
| 주문서의 배송요청일 필터 기능 추가       |     개발자B      |    3     |   2022-11-15   |
| 매장명 변경 시 변경이벤트 전파 기능 추가 |     개발자A      |    4     |   2022-11-30   |

<br>

- 문서 2

|  분류  | 작업                                     | 정책 및 스펙                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |      작업자      | 우선순위 | 예상개발완료일 |                          작업 현황                           |
| :----: | :--------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------: | :------: | :------------: | :----------------------------------------------------------: |
| 거래처 | 엑셀 다운로드 포멧 추가                  | 주문관리 내 엑셀다운로드 클릭 시 [거래처별 내역] → [주문서별+품목별 내역] <br> 문구 변경 <br> - 엑셀 다운로드 형식*공통 <br>- 다운로드 확장자 형식 : xlsx <br> - 다운로드 초기값 이름 : [YYYY-MM-DD]*주문서별+품목별\_주문내역.xlsx <br>- 엑셀 다운로드 형식 : 첨부 파일 참고 <br> - 엑셀 내 null값인 경우 - 처리                                                                                                                                                                                                                                                                                                                                                                                                 |     개발자A      |    1     |   2022-10-27   |               `준비중`, `작업완료`, `입고완료`               |
| 거래처 | 품목 정보 내 품목 코드 추가              | 품목 관리 목록 <br>no / ERP코드 / 품목명 / 규격 / 단위 / 수정일시 / 비고 / 수정아이콘 / 삭제아이콘 <br>- null값인 경우 `-` 처리                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |     개발자B      |    1     |   2022-10-29   | `준비중`, `설계완료`, `작업생성완료`, `작업완료`, `입고완료` |
| 거래처 | 상품 정보의 상품 수정 및 이력 저장       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |     개발자C      |    1     |   2022-11-04   | `준비중`, `설계완료`, `작업생성완료`, `작업완료`, `입고완료` |
|  매장  | 매니저 추가                              | 휴대전화번호 유효성 체크 <br> - 체크 시점 <br> -- 에러메시지 <br> --- 숫자 입력 후 포커스를 벗어날 때 <br> -- [매니저 추가하기] 버튼 <br> --- 숫자 입력 중숫자 입력 후 포커스를 벗어날 때 <br> - 유효하지 않은 휴대전화번호 <br> -- 앞 3자리 <br> --- 010, 011, 016, 017, 018, 019 외 경우 <br> -- 총 자리수 <br> --- 10자리 또는 11자리가 아닌 경우                                                                                                                                                                                                                                                                                                                                                              | 개발자D, 개발자E |    2     |   2022-11-10   |       `준비중`, `설계완료`, `작업생성완료`, `작업완료`       |
| 거래처 | 거래처에서 매장에 별칭 추가              | - 1.1 거래처는 거래처 웹 - 거래처 관리에서 거래처명 내 메모(별칭)를 추가할 수 있다. <br> - 2.1 거래처는 거래처 웹에서 추가된 메모(별칭)를 확인할 수 있다. <br> - 3.1 유통사는 유통사 앱 - 채팅목록, 주문서에서 메모(별칭)를 확인할 수 있다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |     개발자E      |    2     |   2022-11-13   |             `준비중`, `설계완료`, `작업생성완료`             |
| 관리자 | 관리자에서 거래처와 매장 직접 연결       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |     개발자A      |    2     |   2022-11-11   |             `준비중`, `설계완료`, `작업생성완료`             |
|  매장  | 주문서의 수정 이력 노출                  | - 대상: 유통사웹 > 주문관리 > 주문내역상세 <br> - 상황: 해당 주문서에 품목 수정이 발생한 경우 <br> - 조회 시, 첫 주문서와 (조회 시점의)최종 주문서의 품목을 비교하여 표시한다. <br> - 품목 수량이 변경되었을 경우, 해당 품목 옆에 ‘수정됨’ 표시 <br> - 품목이 추가되었을 경우, 주문서 하단 품목 추가 후 ‘추가됨’ 표시 <br> - 품목이 삭제되었을 경우, 주문서 (품목 추가 하단)하단 품목 삭제 표시 후 ‘삭제됨’ 표시 <br> - 항상 제일 처음 주문서의 값과 비교하여 변경된 품목 상태를 표시한다. <br> - 마감 전까지 또는 마감 완료 후에 변경이 발생했을 때에도 표시한다. <br> - 주문 품목 복사 시에는 해당 정보를 남기지 않고, 최종 품목 정보만 남기면 됨 <br> - 엑셀 다운로드 시에도 해당 변경 정보는 반영할 필요 없음 |     개발자C      |    3     |   2022-11-20   |                     `준비중`, `설계완료`                     |
| 거래처 | 주문서의 배송요청일 필터 기능 추가       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |     개발자B      |    3     |   2022-11-15   |                     `준비중`, `설계완료`                     |
| 관리자 | 매장명 변경 시 변경이벤트 전파 기능 추가 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |     개발자A      |    4     |   2022-11-30   |                           `준비중`                           |

**문서1**은 보여주는 정보는 적지만 한눈에 작업의 목록과 우선순위가 보인다. **문서2**는 각 작업들의 세부정보들을 풍부하게 제공해 줌으로써 각 문서의 상세 정보 링크를 들여다 보지 않고도 많은 정보를 얻을 수 있다.

위 두 문서를 보았을 때 어떤 문서가 **작업들의 현황 및 우선순위 파악**이 쉬울까?

나는 **문서1**이 각 작업들의 현황 및 작업 간 우선순위를 파악하기에 좋다고 생각한다. **문서2**에서도 해당 정보를 파악할 수 있고 정보가 풍부하기는 하지만 문서의 제목에서 말하는 `백엔드의 작업 현황 및 우선순위`의 목적에 필요한 정보 외에 과하게 많은 정보들을 포함하고 있기 때문에 한눈에 작업 현황 및 우선순위를 파악하기 힘들어 **문서1**보다는 원하는 정보의 전달이라는 문서의 목적에는 부족함이 있어 보인다. **문서1**과 같이 각 작업의 상세 정보는 과감하게 생략하여 현황 파악 및 우선순위에 집중하고 만약 세부내용을 확인하고 싶다면 링크를 통해 상세 정보를 볼 수 있는 페이지로 이동하여 좀 더 자세한 정보를 파악하는 것이 좋아 보인다.

나는 이와 같은 사례를 **문서의 추상화**라고 표현하고 싶다. 문서의 추상화는 **목적에 맞게 문서를 최대한 단순화**하는 작업이라고 말할 수 있다. 목적에 맞지 않는 정보는 최대한 줄이고 생략하여 목적에 부합하는 꼭 필요한 정보만을 입력하여 해당 문서를 읽는 이로 하여금 자신이 필요한 정보를 최대한 빠르고 효율적으로 가져갈 수 있도록 작성하는 것이다.

한편, 그러면 정보가 부족하지 않느냐라는 이야기를 할 수 있을 것 같다. 나는 더 자세한 정보가 필요하다면 각 상세페이지를 통해 정보를 확인하면 된다고 생각한다. 각 상세페이지 문서의 목적은 최대한 자세하고 구체적인 정보를 제공하는 것이니 말이다.

우리는 하루에도 수없이 많은 문서를 보고 정보를 공유받고 있다. 반드시 알아야할 정보를 잘 제공해주는 문서들이 있다면 여러분이 문서를 읽는데 들이는 노력을 몇 배나 아낄 수 있을 것이고 이는 곧 효율적이고 생산적인 업무로 이어지지 않을까.

### 회의

회의를 효율적으로 하는 방법에 대해서는 조금만 구글링을 해보면 수없이 많은 글을 통해서 알아볼 수 있다. 이 글을 읽는 여러분들도 회의를 잘할 수 있는 방법에 대해서는 이미 잘 알고 있으리라 생각한다. 여기에서는 다른 좋은 방법들은 생략하고 **추상화**의 관점에서 회의를 목적에 맞게 단순화 하기 위한 노력들을 하는 방법에 대해 다루어보고자 한다.

회의는 정보의 공유 및 의사결정을 수립하기 위한 목적으로 개최된다. 우리가 가진 지식 또는 정보를 문서로만 표현하기에는 한계가 있어 회의를 개최하고 구두로 정보를 풀어 설명하기 위한 것이 하나고 합의를 이루어야 하는 경우 효율적으로 의사결정을 위해서 개최되는 것이 다른 하나일 것이다.

나는 회의도 문서와 마찬가지로 **목적에 맞게 최대한 단순하고 효율적인 회의**를 하면 좋겠다고 생각한다. 이를 **회의의 추상화**라고 표현할 수 있겠다. 이를 위해서는 회의의 주제를 잘 정하고 회의전에 회의에 필요한 전반적인 내용을 파악할 수 있는 문서를 작성해서 회의에 참가할 참가자들이 문서를 숙지하고 회의에 참가할 수 있도록 하는 것이 중요하다.

물론 급하게 잡히는 회의에서는 문서작성을 사전에 하고 숙지 후 참가하도록 하는 것은 힘들 수 있다. 하지만 나는 그럼에도 불구하고 예외 케이스는 과감히 수용하고 위에서 말한 정책을 지키기 위한 규칙을 정하고 실천하기 위해 다같이 노력하면 좋다고 생각한다. 조심스럽게 예상해보건데 이 규칙들이 잘 지켜진다면 우리의 회의시간은 훨씬 더 짧아지고 유익한 회의가 되리라 생각한다.

아래는 우리가 하는 회의의 종류들을 보고 그 종류에 맞는 회의의 목적을 정해보고 실천해볼만한 것들을 적어보았다.

#### 의사 결정 회의

문제 상황을 적은 문서가 작성되고 회의전 모든 참가자들이 해당 문서를 숙지하여 회의가 진행될 때 해당 문제를 해결하기 위한 목적만을 가지가 회의가 진행되면 좋겠다. 만약 문제에 대한 문서가 없다면 의사결정을 위한 전반적인 내용을 설명하는데 회의시간의 절반이 소비될 가능성이 높다. 그리고 해당 회의의 목적을 잘 파악하지 못해 회의에서 얻으려는 의사결정에 대한 판단이 흐려질 가능성이 아주 높다.

그래서 사전에 정보 전달이 되지 않은 회의에서는 정작 중요한 문제 해결에 대한 해결방안은 나오지도 않고 언제 발생할지도 모를 머나먼 미래에 대한 논의만 이루어지거나 논쟁만 하다가 회의가 끝날 수도 있다. 혹은 논쟁이 길어져 회의가 계획했던 시간보다 길어지는 경우도 종종 생길 것이다.

반대로 의사결정 회의 전 배경 설명 및 회의에서 도출하기 위한 의사결정에 대한 목적이 분명한 문서가 작성되고 참가자들이 이를 숙지한 후 참가하게 된다면 의사결정 회의의 질은 180도 잘라질 것이다. 모두가 명확한 목표를 가지고 토론을 진행할 것이고 혹여 누군가 목적에 벗어난 논의를 하려고 한다면 목적에 맞게끔 다른 참가자들이 보정해주는 상황도 쉽게 발생할 것이다. 그리고 아마 회의시간도 아주 짧게 기본으로 30분 내로 잡을 수 있을 것이다. 왜냐하면 보통 배경을 설명하느라 10~20분을 소비하는 경우가 많기 때문이다.

#### 정보 전달 회의

나는 논의할 내용이 없다면 정보 전달 회의의 경우 문서로 대체할 수 있다고 생각한다. 지난 정보 전달 회의를 기억해보길 바란다. 아마 누군가 혼자서 작성된 문서를 읽거나 혹은 자신의 머릿속 정보를 열심히 말하고 있는 모습이 그려질 것이다. 참가자들은 아무말 없이 그 누군가의 말을 듣고만 있을 것이다. 혹은 노트북을 가지고 다른일을 하고 있을 수도 있다.

나는 이런 단방향 성격이 강한 회의에 수많은 참가자가 바쁜 시가능ㄹ 내어 회의에 참가하는 것은 참 아깝다고 생각한다. 차라리 문서를 작성해서 내용을 공유하고 질문이 있다면 문서의 댓글에 질문을 하는 등의 방법으로 정보의 전달 목적을 달성하면 어떨까? 아마 우리가 참가하는 일주일 중 많은 회의시간을 줄일 수 있을 것이다.

#### 정기 회의

정기 회의는 정보 전달과 문제 해결에 대한 내용이 섞여있다고 생각된다. 그래서 정보 전달은 앞서 말한 바와 같이 되도록 문서로 대체하고 이슈와 같이 논의를 통해 해결해야 하는 문제에 초점을 맞추어 해당 문제를 해결하는 논의에 시간을 투자한다면 좀 더 정기적인 회의가 유익하고 단순해 질 것이라 생각한다.

혹시 문서를 읽지 않고 참가하지 않을까 하는 우려가 있을 수 있다. 정기 회의 전 충분한 시간을 두고 문서가 완성되고 공유된다면 그리고 성숙한 회의 참가자라면 당연히 사전에 문서를 숙지하고 회의에 참가할 것이다. 이러한 문화가 당연시 되도록 관리자의 지속적인 노력이 필요하다.

<br>

정리하자면 회의의 목적과 성격에 맞게 회의 방식을 단순화하고 실천 규칙 즉, 회의의 **추상화**하기 위한 방법들을 만들어서 이를 지키고자 노력하면 좋겠다. 이를 위해서는 구성원 모두가 이 부분을 숙지하고 노력해주어야 달성가능하리라 생각한다.

### 커뮤니케이션

직접 만나서 하는 대화나 슬렉과 같은 매체를 통한 대화에서도 **추상화**를 이야기할 수 있다. 뭔가 말이 이상해 보이니 다른 말로 표현하자면 **커뮤니케이션을 할 때에도 목적에 맞게 간결한 소통을 지향하자**로 말할 수 있을 듯 하다. 사담을 제외하고 우리가 수행하는 업무에서 많이 사용하고 있는 대화는 결국 회의와 유사하게 정보의 전달 및 의사결정을 위한 목적을 가진다. 결국 정보를 보다 이해하기 쉽고 간단하게 전달하고 빠르고 효율적으로 의사결정을 내릴 수 있도록 **대화의 추상화**를 하면 좋겠다는 생각이 든다.

효율적인 커뮤니케이션 방법은 찾아보면 아주 많지만 이 글의 주제인 목적에 맞는 **간결함**에 초점을 맞추어서 몇가지 내용만 언급해 보겠다. 좋은 커뮤니케이션을 위해서는 효율성 뿐만아니라 감정적인 요인도 반드시 고려되어야 한다고 생각한다. 하지만 이번 주제는 좋은 커뮤니케이션을 위한 내용만을 다루는 것이 아니라 **대화의 추상화**를 위한 방법들을 이야기하고 있으므로 감정적 요인은 배제하고 말한다는 점을 참고해주길 바란다.

#### 두괄식으로 이야기하기

두괄식은 의사결정을 명확하고 빠르게 할 수 있는 장점이 있다. 커뮤니케이션의 목적이 정보전달에 있다면 이 방식보다 효율적인 커뮤니케이션 방법은 없으리라 생각된다. 서론이 길면 집중력이 흐려지고 필요한 정보를 얻기까지 상당한 시간을 소모하게 된다. 차라리 전달해야할 필수 정보를 가장 앞서 강조한 다음 추가정보는 선택적으로 받아들일 수 있도록 두괄식으로 대화하는 것은 어떨까?

예를 들어보겠다.

> A: B님 이슈가 있습니다.
>
> B: 어떤 이슈인가요?
>
> A: 주문서 전송 시 문제가 좀 있는 것 같아요.
> 
> B: 어떤 문제가 있나요?
> 
> A: 현재 주문을 전송하면 특정 데이터가 누락되어서 저장되고 있습니다.
>
> B: 어떤 데이터가 누락되어서 저장되고 있나요?
>
> A: 배송요청사항이 누락되어서 저장되고 있습니다.
>
> B: 네. 조치 부탁드릴께요.

위 대화를 두괄식으로 말한다면 어떻게 할 수 있을까?

> A: B님 주문을 전송하면 배송요청사항이 누락되는 이슈가 있습니다. 조치가 필요해 보입니다.
>
> B: 네. 조치 부탁드릴께요.

조금 극단적으로 표현한 사례이긴 하지만 업무를 하다보면 이와 유사한 형태를 띄는 대화를 종종 발견하곤 한다. 아래 대화와 같이 두괄식으로 대화를 진행한다면 대화에 참가한 사람들은 자신이 필요한 정보를 빠르게 얻을 수 있을 것이고 대화는 빠르게 종료될 것이다. 만약 추가적인 정보가 필요하다면 해당 정보가 필요한 사람만 대화에 남아 추가 정보를 요구할 수도 있을 것이다.

두괄식이라고해서 커뮤니케이션을 짧게 하자는 말은 아니다. 목적에 맞게 전달하고자 하는 정보를 앞으로 이동시키고 부가적인 세부정보를 뒤로 보내서 전체적인 내용도 함께 파악할 수 있는 선택의 여지를 남겨두는 것이다.

#### 이야기가 아닌 사실을 전달하기

사실(실제로 일어난 일)은 대화에 참가한 사람들이 쉽게 납들하고 이해할 수 있다. 반면 이야기(자신이 해석한 것)은 주관적인 관점에서 대화를 하는 것이기 때문에 그 사람이 하는 말을 이해하기 위해 많은 노력이 필요하다. 듣는 사람에 따라서는 오해의 여지가 있을 수 있어 듣는 이가 오해를 하기 시작하면 대화는 더욱더 힘들어지게 된다. 또한 객관적 사실이 아닌 자신의 생각을 말하다보면 제대로된 의사결정을 하는데 방해요소로 작용할 수 있다.

예를 들어, A(매니저)가 팀 미팅 때 B(팀원)에게 실시간 피드백을 주었다고 가정해 보자. `A가 팀 미팅 때 B에게 실시간 피드백을 주었다`는 "사실"이다. 하지만 `B는 피드백을 기대하지 않았으며, A가 B의 업무에 만족하지 않기 때문에 1:1 미팅 대신 팀 미팅 때 피드백을 공유한 것이라고 생각한다`는 사실인지 아닌지 알 수 없기 때문에 "이야기"이다.

제품에 대한 대화를 할때도 이와 같은 사례는 종종 볼 수 있다. `~때문에 오류가 발생한 것 같아요`보다 `~로 인해 오류가 발생하였어요`와 같이 사실을 기반으로 대화가 진행되어야 대화 당사자간에 필요한 목적을 빠르게 달성하고 의사결정을 내리기 쉽다. 이와 같은 사례에서 정확한 오류 원인이 파악되지 않았다면 `확인하고 다시 말씀드릴께요`와 같이 말하고 사실 확인 후 대화를 다시 재개하는 방법을 사용하면 좋겠다.

우리가 대화를 이어갈 때 이야기를 하는 것은 불가피하다. 하지만 듣는 사람 입장에서는 사실과 이야기를 잘 구분해서 대화를 하기 쉽지 않기 때문에 말을 하는 사람이 대화에서 사실이 아닌 이야기를 하지 않도록 주의를 기울일 필요가 있다.

#### 적절한 쿠션어 사용

"쿠션어"란 간접적으로 의사를 전달하기 위해 덧대는 안전장치 같은 말을 의미한다. 상대방을 배려하거나 감정이 상할 수 있는 대화에서 쿠션어는 큰 효과를 발휘할 수 있다. 하지만 지나친 쿠션어의 사용은 우리의 대화의 주제를 흐리거나 목적을 잃게 만들 수 있다고 생각한다.

객관적인 사실을 전달한다거나 상대방의 감정이 상할 수 있더라도 정확한 정보를 전달해야하는 상황이라면 나는 과감하게 쿠션어를 생략해서 좀 더 명확하고 간결한 대화를 이어가야 한다고 생각한다.

#### 목적을 명확하게 하기

두괄식의 연장선이긴 하지만 대화의 목적을 분명히하고 목적을 달성하기 위한 대화에 집중한다면 불필요한 커뮤니케이션 비용이 발생하지 않을 것이다. 목적성을 잃은 대화는 발산에 불과하다. 목적없이 자신이 말하고 싶은대로 말하고 말하는 것을 멈추지 않는다면 듣는이는 상대방이 무슨말을 하는 지 이해하지 못해 집중력이 흐려지고 말하고 있는 본인 스스로도 무슨말을 하는지 몰라 대화가 끝난 후에도 서로에게 공허함만 남는 그런 대화가 될 것이다. 이런 대화는 서로를 지치게 하고 상대방의 의도를 파악하기 위해 2차, 3차 새로운 대화가 열리는 등 불필요한 커뮤니케이션 비용이 증가하게 되는 것이다.

## 마무리

지금까지 **추상화**라는 단어를 이용하여 우리가 협업에서 추구했으면 하는 것에 대한 나의 생각을 공유해 보앗다. 협업에서 효율성과 단순함만으로 모든 문제를 해결할 수 있다고 생각하진 않는다. 하지만 그렇다고 비효율을 고치지 않고 불편함을 계속해서 유지하는 것 또한 좋은 방향은 아닌것 같다. 이 글로 인해 한번에 여러분의 협업문화를 개선할 순 없겟지만 여러분의 협업방식을 다시 한번 되돌아 보고 조금씩이라도 개선할 수 있는 기회가 있다면 이 글이 유의미하지 않을까라는 생각을 해본다.

협업을 잘하는 것은 한사람만의 노력으로는 이루어낼 수 없다. 팀원들이 협업을 수행함에 있어 누군가는 부족한 부분이 있을 수도 혹은 어느 누군가는 불편한 업무와 협업방식이 있을 수도 있다. 그럼에도 불구하고 모두가 함께 이를 해결하기 위해 적극적인 노력이 이어진다면 더 나은 협업문화를 이루어 낼 수 있으리라 생각한다.

## 참고

> [https://asana.com/ko/resources/effective-communication-workplace](https://asana.com/ko/resources/effective-communication-workplace){: target="\_blank" }
