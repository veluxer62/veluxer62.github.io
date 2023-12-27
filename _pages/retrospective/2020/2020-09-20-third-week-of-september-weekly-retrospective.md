---
layout: single
title: "2020년 9월 3주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-third-week-of-september-weekly-retrospective/
---

이번주에 드디어 개발중인 서버를 개발 서버에 배포하였다. 운영서버는 아직 수정할게 조금 더 남아 있어서 개발 완료 후에 배포할 예정이다. 어차피 flayway로 프로젝트를 구성하였기 때문에 DDL 걱정은 없고 헬름차트도 개발용은 이미 완성되어 있기 때문에 프로덕션도 비슷하게 하면 될거 같아서 배포는 금방 끝날것 같다. 다음주에는 이제 현재 운영중인 서버에 작성된 코드들을 변경하기 쉽도록 리펙토링 작업과 대체할 수 있는 데이터 정합성 체크가 필요하며 해당 작업이 완료되면 마이그레이션을 진행한 후 최종적으로 운영서버에 배포할 예정이다. 계획만 보면 손쉬울것 같아 보이지만 실제 코드를 까보기 전까진 모르는 것이니 너무 기대하진 말아야 겠다.

우리팀에도 드디어 기획자가 생겼다. 입사하신지 얼마 되시지 않으셔서 히스토리 파악이 좀 필요하긴 하시겠지만 이해력도 좋으신것 같고 커뮤니케이션도 잘하실것 같아서 앞으로 기대가 된다.

이번주에도 늦은 퇴근이 많아서 아침운동을 2번밖에 가지 못했다. 최소한 3번은 가도록 컨디션 조절을 잘해야 겠다.

아래 내용은 이번주에 내가 겪었던 이슈 또는 배운 것들에 대한 정리이다.

## RDB 인덱스 시 정렬

당연한 예기일 수도 있겠지만 RDB에서 인덱스를 설정할때 데이터 구조를 좀더 잘 알고 있다면 인덱스에 정렬을 사용하여 적용하는 것이 좀더 효율적일 수 있다. 평소에 아무생각없이 정렬없이 인덱스만 적용해왔었는데 동료와 얘기하면서 인덱스 적용 시 정렬 활용에 대한 얘기가 나와서 적어본다.

## Kotlin의 `isEmpty` vs `isBlank`

Kotlin에서 String클래스는 [`isEmpty`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/is-empty.html){: target="\_blank" }함수와 [`isBlank`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/is-blank.html){: target="\_blank" }함수가 있다.

`isEmpty`함수는 문자열이 비어있는지 체크 하는 함수 즉, 아무글자가 존재하지 않는지를 체크하여 그 결과를 `true` 또는 `false`로 반환한다. 예를 들면 `""`는 아무 글자가 존재하지 않으므로 `"".isEmpty()`는 `true`이다. 하지만 `" "`는 공백 글자가 존재하므로 `" ".isEmpty()`는 `false`이다.

`isBlank`함수는 문자열이 공백인지 체크해서 그 결과를 `true` 또는 `false`로 반환하는 함수이다. 그래서 `"".isBlank()`와 `" ".isBlank()`모두 `true`로 반환다.

사용자 입력에 대한 공란 체크 시 `isEmpty`보다는 `isBlank`를 활용하자!

## swagger 3.0

신규 프로젝트에 Swagger를 설정하려고 보니까 3.0버전인것을 확인할 수 있었다. 7월 14일에 업데이트 되었다.
업데이트 되면서 바뀐것인지는 모르겠는데 Swagger 적용을 안내하는 블로그에서는 Configuration 설정 등이 나와 있었는데(나도 이전에는 그렇게 사용했다), 이제는 `springfox-boot-starter`의존성만 추가해 주어도 Swagger를 바로 사용할 수 있다. 정말 편리해 졌다.

또 추가적으로 눈에 띄는 점은 Generic 타입이 이전 버전에는 Swagger에서 표현되지 않았었는데 이제는 Swaager UI에서도 Generic 타입을 가진 클래스를 표시해 준다는 것이다.

## ECR 자동 로그인

ECR 자동 로그인 기능을 사용하기 위해서 [amazon-ecr-credential-helper])(https://github.com/awslabs/amazon-ecr-credential-helper){: target="\_blank" }를 사용하였다. 가이드 라인에 따라서 적용해보려고 하였으나 `docker-credential-ecr-login list`를 실행하니 `Could not list credentials: MissingRegion: could not find region configuration:`와 같은 오류 메시지를 응답할 뿐 정상적으로 적용이 되었는지 확인할 방법이 없었다.

해당 문제를 검색해보니 동일한 [issue](https://github.com/awslabs/amazon-ecr-credential-helper/issues/63){: target="\_blank" }가 등록되어 있는것을 확인할 수 있었고 [HrmesWorld](https://github.com/awslabs/amazon-ecr-credential-helper/issues/63#issuecomment-328318116){: target="\_blank" }님께서 적어주신 해결방안으로 해결 할 수 있었다.

## javascript Moment 모듈 업데이트 종료

백엔드 개발자라 Javascript를 많이 사용하진 않지만 여기저기서 들어보고 써보기도 했던 [Moment.js](https://momentjs.com/){: target="\_blank" }가 공식적으로 레거시 프로젝트로 전환한다고 한다. [https://www.facebook.com/groups/nodejskr/permalink/2816106205156318/](https://www.facebook.com/groups/nodejskr/permalink/2816106205156318/){: target="\_blank" }

앞으로는 [Day.js](https://github.com/iamkun/dayjs){: target="\_blank" }나 [date-fns](https://github.com/date-fns/date-fns){: target="\_blank" }를 사용하자.

## RDB에서 UUID 사용

JPA에서 UUID 타입을 컬럼으로 지정하면 기본적으로 binary 타입을 사용한다. 솔직히 binary 타입을 사용해도 어플리케이션에서는 아무 문제가 되지 않는다. 성능적으로도 용량면에서도 binary(16)이 낫지 varchar(36)은 좋지 않다고 생각한다. 하지만 DB를 직접 봐야 하는 DBA나 DATA 팀, 또는 개발자가 운영이슈를 처리할때 binary 타입은 문제가 될 수 있다.

`select * from foo`를 해보면 볼 수 있겠지만 bianry 타입을 가진 컬럼은 예상한 UUID 문자열이 아닌 알아보기 힘든 글자로 출력된다. 바이너리니 당연하지....그래서 해당 속성을 보려면 아래와 같이 프로시져를 등록해서 볼 수 있다. 참고링크 : [https://mariadb.com/kb/en/guiduuid-performance/](https://mariadb.com/kb/en/guiduuid-performance/){: target="\_blank" }

```
CREATE FUNCTION UuidToBin(_uuid BINARY(36))
    RETURNS BINARY(16)
    LANGUAGE SQL  DETERMINISTIC  CONTAINS SQL  SQL SECURITY INVOKER
RETURN
    UNHEX(CONCAT(
        SUBSTR(_uuid, 15, 4),
        SUBSTR(_uuid, 10, 4),
        SUBSTR(_uuid,  1, 8),
        SUBSTR(_uuid, 20, 4),
        SUBSTR(_uuid, 25) ));
//
CREATE FUNCTION UuidFromBin(_bin BINARY(16))
    RETURNS BINARY(36)
    LANGUAGE SQL  DETERMINISTIC  CONTAINS SQL  SQL SECURITY INVOKER
RETURN
    LCASE(CONCAT_WS('-',
        HEX(SUBSTR(_bin,  5, 4)),
        HEX(SUBSTR(_bin,  3, 2)),
        HEX(SUBSTR(_bin,  1, 2)),
        HEX(SUBSTR(_bin,  9, 2)),
        HEX(SUBSTR(_bin, 11))
              ));
```

프로시저를 등록하여 사용하는 방법도 나쁘지는 않는데, 해당 프로시저를 각각의 팀에 공유해야 하고 유지보수의 용이함을 위해서 varchar 타입으로 타협을 봤다. JPA에서 UUID를 varchar 타입으로 생성되도록 하려면 아래와 같이 컬럼에 주석을 추가하면 된다.

```kotlin
@Type(type = "uuid-char")
private var slug: UUID,
```

## ApplicationEventPublisher 사용시 Transaction 분리

스프링에서 `ApplicationEventPublisher`를 사용하여 발행구독 모델을 사용하는 경우 `EventListener`를 사용하지 않고 `TransactionalEventListener`를 사용하여 트랜젝션을 분리할 수 있다. 하지만 동기 모드에서는 `TransactionalEventListener`가 이전 트렉잭션과 분리되어 동작하지 않는 것을 볼 수 있는데 이유는 `TransactionalEventListener`를 이용하여 트랜잭션을 분리하려면 `Async`를 사용해야 하기 때문이라고 한다.

## Slug

서비스 내부용 ID가 존재하는 상황에서 외부에 노출할 고유 식별자의 이름으로 `slug`이라는 단어를 사용하기도 한다는 것을 듣게 되었다. 워드프레소에서도 활용하는것 같아 보인다. [https://ivynet.co.kr/what-is-slug/](https://ivynet.co.kr/what-is-slug/){: target="\_blank" }

## JPA에서 ID를 생성하여 insert 하는 경우

신규 프로젝트를 진행하면서 도메인모델을 생성하고 생성된 이벤트를 발생시켜 그 이벤트를 수신받은 리스너가 데이터베이스에 도메인 정보를 저장하는 방식으로 서비스 로직을 구현해 보았다. 이때 도메인 모델 생성 이벤트에는 ID가 존재해야 하는데 이유는 구독하고 있는 다른 리스너에서 해당 ID를 이용하여야 하는 경우가 있기 때문이다. 만약 이벤트 내에 ID가 존재하지 않는다면 처음에 말했던 이벤트를 수신해서 데이터베이스에 저장하면 그 때 ID가 새롭게 생성될 것이고 생성이벤트를 처리해야할 다른 리스너들은 ID가 필요한 경우 데이터저장 리스터를 의존해야 하는 상황이 생길 수 있기 때문에 이는 내가 처음 구상했던 그림과는 다른 것이었다.

서론이 길었는데 아무튼 이러한 이유로 도메인 생성시 시퀀스 테이블을 이용하여 새로운 ID를 발급받아 도메인을 생성하고 생성이벤트를 발생시키고자 하였다. 근데 JPA를 사용할 때 이렇게 ID가 존재하는 Entity를 저장하려고 하면 persist 전략이 아닌 merge 전략을 채택한다는 것을 알 수 있었다. 그래서 cascade 를 사용할때 persist 전략만 선택해서 적용한 경우 insert가 원하는대로 동작하지 않을 수 있다.

이러한 문제를 해결하기 위한 대안으로 아래와 같이 생각해 봤다.

- querydsl 을 이용한 insert 기능을 직접 해주는 방법

  이는 cascade를 활용할 수 없어서 직접 연관 Entity를 하나하나 다 insert 구문을 적어줘야 해서 별로라 생각했다.

- cascade 전략을 merge를 추가하거나 all로 하는 방법
  
  이는 내가 cascade를 persist 전략만 채택한 이유에 부합하지 않기 때문에 좋지 않은 방법같다.

- isNew를 재정의 하는 방법

  생각했던 대안중에 가장 나아 보이는데 update와 insert를 어떻게 구분지을지가 문제이다.

아직 해결방안을 찾지 못했다... 좋은 방법이 없을까?
