---
layout: single
title: "2023년 10월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2023-october-dev-log/
---

20대 초반에 어금니가 썩어서 신경치료 후 크라운을 씌웠었다. 최근 잇몸이 내려앉아서 치과를 갔는데 엑스레이를 찍어보니 잇몸 속에 이가 썩어서 잇몸이 내려앉은거라고 하였다. 그래서 결국 발치 후 임플란트를 하기로 했다. 임플란트는 할저씨가 되면 하는줄 알았는데 벌써 한다니 조금 슬픈 기분이 들었다.

동네병원에서 진단을 받고 견적을 받았는데 아내가 강남에서도 견적을 한번 받아보라고 하여 유명한 치과에 가서 견적을 받아보았다. 그랬는데 왠걸 강남이 훨씬 싸고 더 좋은 서비스를 제공해주는 것이 아닌가! 역시 의료쪽(?)은 강남이 좋은것 같다.

<br/>

저번달 부터 그랜드 슬램 달성을 위한 국토종주를 시작하였다. 추워지기 전에 부지런히 달려야 하기 때문에 10월에 나머지 두 구간인 영산-섬진강 종주와 제주환상종주길을 가기로 하였다.

영산-섬진강 종주는 오천-금강 종주길처럼 강둑을 주로 달렸기 때문에 저번달에 갔던 동해안 자전거 길보다는 재미가 없긴 하였다. 그리고 여름에 수해를 입어서 종주길이 막혀 있거나 길이 끊어져 우회해야하는 경우가 많았다. 그래서 계획했던 일정보다 더 많은 시간과 체력을 소비했어서 조금 스트레스를 받기도 하였다. 하지만 전라도 지역을 달리다 보니 맛있는 음식을 많이 먹을 수 있어서 좋았다.

제주도 환상종주길은 3박 4일을 잡고 아내와 함께 제주도를 가서 달리기로 하였다. 200KM정도로 마음먹으면 하루만에도 갈 수 있었지만 아내가 힘들게 달리지는 말라고 해서 1박2일로 달리고 나머지 시간은 아내와 제주도 여행을 즐기기로 하였다. 제주도 환상종주길은 대부분이 평지고 바닷가를 달리기 때문에 좋은 풍경을 구경하며 재미있게 달리기 좋았다. 만약에 다시 또 제주도를 간다면 이번에는 천천히 달리면서 주변 구경도 하고 여유롭게 달려봐야겠다.

<br/>

제주도 이야기를 더 하자면 이번에 제주의 신라호텔을 큰마음 먹고 가게 되었다. 학생때 아내와 연애시절 제주도를 갔을 때는 돈이 없어서 최대한 저렴한 숙소와 먹을거리들을 먹었었는데, 지금은 직장인이라 나름 여유롭게 즐겨보고자 마지막 숙소는 신라호텔을 가기로 하였다. 수영장도 가고 유명한 짬뽕도 시켜먹었는데 이 글을 쓰고 있는 지금도 가보고 싶을 정도로 다시 또 가보고 싶은 곳이다. 만약 지인이 제주도를 간다고 하면 꼭 신라호텔을 가보라고 추천해주고 싶다.

<br/>

아래는 10월동안 정리한 이슈 내용들이다.

## DGS 7.x.x 업그레이드 이슈

DGS Framework를 7.x.x로 업그레이드 하면 스프링 부트 실행이 되지 않는다. 이유는 [graphql-java](https://github.com/graphql-java/graphql-java){: target="\_blank" } 버전과 충돌이 나면서 발생하는 이슈인데 아래와 같이 gradle.properties에 버전을 명시해주면 해결할 수 있다.

```
graphql-java.version=21.2
```

관련한 이슈는 [https://github.com/Netflix/dgs-framework/issues/1672](https://github.com/Netflix/dgs-framework/issues/1672){: target="\_blank" }를 참고하자.

## 기능테스트 전환

백엔드 챕터의 오랜 숙원 사업이었던 기능테스트 전환을 드디어 마무리했다. 자세한 이야기는 [기능 테스트 전환 이야기](https://spoqa.github.io/2023/10/20/functional-testing-converting-story.html){: target="\_blank" }에 적었으니 읽어보면 좋겠다.

기능테스트로 전환하니 서버의존성 업그레이드가 한결 더 수월해졌다. 그리고 성능적인 이슈가 발생했을 때 통합테스트에서는 불가능한 경우가 많았는데 기능테스트에서는 실제 동작과 거의 유사하게 동작하다보니 손쉽게 테스트를 수행할 수 있게 되었다.

## ktlint

ktlint가 드디어 메이저 버전이 1버전이 드디어 나왔다. 자세한 설정방법은 [공식 홈페이지](https://pinterest.github.io/ktlint/0.49.1/rules/configuration-ktlint/){: target="\_blank" }에서 참고하자.

## POI

java excel 라이브러리인 poi에서 압축한 파일과 압축을 푼 파일의 파일 크기 차이가 크면 의도적으로 예외를 발생시킨다. 최근 QA에서 동일한 품목명을 가진 대량의 row를 가진 엑셀을 업로드 하다가 발견하게 되었는데 해당 예외를 발생시키는 이유는 보안상 이유라고 하였다. 설정을 통해서 예외가 발생하지 않도록 할 수 있지만 보안상 문제가 생길 수 있으니 되도록 해당 설정을 풀지 않는게 좋다고 생각한다.

참고: [https://stackoverflow.com/questions/44897500/using-apache-poi-zip-bomb-detected/44900653#44900653](https://stackoverflow.com/questions/44897500/using-apache-poi-zip-bomb-detected/44900653#44900653){: target="\_blank" }