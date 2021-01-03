---
layout: single
title: "2020년 11월 2주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

한주간 새로운 기능을 다시 만들기위해서 바쁘게 일했다. 매일 12시를 넘겨서 퇴근하고 오전에 출근하는 루틴을 계속하니 몸이 점점 나빠지는게 느껴진다. 일도 중요하지만 몸을 좀 챙겨야겠다.

요즘 Entity와 Value Object의 경계에 대해 고민을 많이 한다. 처음에는 단순하고 쉬울거라 생각했던 경계가 작업을 하다 보니 명확히 구분을 하지 못할 때도 있었는데, 이럴 경우 일단 Value Object에 넣고 보기로했다. 최근에 `도메인 주도 개발`이라는 유명한 책을 읽고 있는데 100% 이해하면서 읽기엔 힘들거라 생각되고 개념을 잡아가면서 앞에서 말한 고민거리에 대한 해답을 찾을 수 있기를 바란다.

아래 내용은 전주동안 내가 겪었던 이슈와 고민거리를 적은 것이다.

## Swagger 뷰어 확장 프로그램

[https://chrome.google.com/webstore/detail/swagger-viewer/nfmkaonpdmaglhjjlggfhlndofdldfag/related](https://chrome.google.com/webstore/detail/swagger-viewer/nfmkaonpdmaglhjjlggfhlndofdldfag/related){: target="\_blank" } 에서 Chrome의 Swagger 확장 프로그램을 받으면 Github에서 yaml 파일 작성을 통해 Swagger UI를 볼 수 있다.

반드시 서버가 가동중인 경우에만 Swagger UI를 볼 수 있었는데 이렇게 API정의를 미리 하고 클라이언트에 공유하면 아주 유용하게 사용할 수 있다.

[Github](https://github.com/arx-8/swagger-viewer){: target="\_blank" }

[VS code Extention](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: target="\_blank" }

## MS API 가이드

[https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md){: target="\_blank" }

MS에서 API 가이드 문서를 제공해준다. API 설계시 참고하면 좋아보인다. 이번에 filter 부분을 참고해서 작업해볼까 한데, 동일하게 하진 않을거고 POST 함수로 보내서 Body에서 규칙을 정한 후 사용해 보고자 한다.

## mockStatic

[https://notwoods.github.io/mockk-guidebook/docs/mocking/static/](https://notwoods.github.io/mockk-guidebook/docs/mocking/static/){: target="\_blank" }

Mockk 라이브러리에서 제공하는 `mockStatic` 함수를 이용해 현재 시간을 고정된 값으로 반환하도록 할 수 있다.

간단한 테스트 코드를 보면 아래와 같다.

```kotlin
val now = ZonedDateTime.now()

mockkStatic(ZonedDateTime::class)

every { ZonedDateTime.now() } returns ZonedDateTime.of(2020, 1, 1, 0, 0, 0, 0, ZoneId.systemDefault())

now shouldBeAfter ZonedDateTime.now()
```

테스트 시 함수내에 현재시간을 생성하는 기능이 있는 경우 테스트가 어려울 때가 많은데 위와 같은 코드를 사용하면 테스트 시 시간값을 고정할 수 있어 테스트 시 유용하게 사용할 수 있다. (단, 시간이 중요한값이 아니라면!)

사용후 Mocking을 초기화 하고 싶으면 `clearAllMocks`를 사용하면 된다.

```kotlin
clearAllMocks()
```

## NULL 컬럼도 인덱스를 탈까?

해당 질문에 대한 답으로는 DB마다 다를 수 있다 이다.
오라클 DB의 경우 Null 컬럼에 대해서는 인덱스를 타지 않는다. 하지만 MariaDB에서는 Null 컬럼에 대해서 인덱스를 태우는 것을 볼 수 있다.
테스트 할 때 임시로 데이터를 넣을 때 사용했던 프로시져 코드인데 간단히 사용하기 유용한거 같다.

```
CREATE PROCEDURE `p_nulltest`()
BEGIN
 DECLARE number INT DEFAULT 0; 
 SET @number = 1; 
  WHILE @number < 100000 DO 
   INSERT INTO nulltest (col1,col2) VALUES (
           (CASE mod(@number,2)  WHEN 0 THEN NULL ELSE CASE(ROUND(RAND()*10000)) END), 
           (CASE MOD(@number,3)  WHEN 0 THEN NULL ELSE CAST(ROUND(RAND()*10000)) END) );
   SET @number = @number+1;
  END WHILE;
 END
```

## Nullable 컬럼의 Multi Column Unique Index

Nullable한 컬럼에서 null 값들에 대해 유니크 인덱스를 사용하면 유일성을 보장하지 않는다. 비지니스 요구사항에 따라 다르겠지만 null컬럼에 대해서도 유일성을 보장받고 싶었던 나는 해당 이슈로 인해 유니크 인덱스를 주기 위해 다른 방식을 도입했어야 했다.

내가 하고자 했던 방식은 아래와 같다.

`deletedAt` 컬럼을 nullable로 사용하여 소프트 딜리트를 구현하고자 하였고 `barcode`라는 컬럼을 유일값으로 사용하고 싶었다. 여기에서 삭제되지 않은 데이터에 대해서만 유일성을 보장받고 싶었는데 `barcode`값과 null 값을 가진 `deletedAt`을 함께 유니크 인덱스를 주니까 동일한 값이 계속 저장되는 것을 볼 수 있었다. 결국 `deletedAt`에 값이 있어야지만 `barcode`와 함께 유일성을 보장받을 수 있기 때문에 해당 방법은 사용할 수 없었다.

## Unit Test

새롭게 시작하는 프로젝트에서는 단위테스트는 하지 않고 통합테스트만 하기로 마음먹고 진행중에 있다. 근데 무조건 단위테스트는 배제할게 아니라 도메인 모델의 중요한 로직에 대해서는 유닛테스트를 작성하는게 개발 안정성도 높이면서 작업에 대한 속도도 높일 수 있는 것 같아서 병행해서 작업해야겠다고 판단했다. 뭐 나중에 테스트가 중복된다면 제거하면 되니까 유연하게 가는게 더 좋아 보인다.

## Value Object

JPA에서 Value Object를 표현하기 위해서 `Embedded`와 `ElementCollection`을 적극 활용하자. Table에는 무조건 ID가 존재해야 한다는 고정관념을 버리고 Entity 객체가 아닌 테이블에 대해서는 `Elementcollection`을 DB 테이블의 컬럼은 Flat 하게 가져가지만 Entity Model로 표현할때는 Value Object로 표현하기 위해서는 `Embbed`를 사용하자.