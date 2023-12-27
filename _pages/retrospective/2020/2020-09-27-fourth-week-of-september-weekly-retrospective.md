---
layout: single
title: "2020년 9월 4주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-fourth-week-of-september-weekly-retrospective/
---

이번주는 개발 업무는 많이 못하고 회의나 이슈대응 인수인계등등 다른 업무를 많이 하였다. 퇴사하는 개발자가 점점 생기면서 기존에 담당하고 있던 일들이 인수인계되거나 이슈가 생겼을 때 대응하는 시간이 길어져서 개발자 모두가 본업에 집중하는게 쉽지 않은 상황이다. 이래서 문서화가 잘되고 지식의 공유가 필요한 것이겠지만 막상 스타트업에서는 이 이상적인 얘기들이 현실적인 문제로인해 실천하기 힘든부분이 많이 있는게 사실이다. 남은 친구들이 이러한 문제를 어떻게 좀더 해결해 나갈지 고민을 많이 해봐야할 것 같다.

SKU서비스에 대한 1차적인 기획안을 만들고 실무자와 회의하는 시간을 가졌다. 아무래도 개발자가 실무자와의 논의 없이 기존 시스템을 토대로 모델링을 한 터라 현업의 니즈와 맞지 않는게 많지 않을까 걱정을 하였는데, 생각보다는 많지는 않았지만 그래도 좀 헷갈려 하는 부분이 있거나 수정할 부분들은 존재했다. 어서빨리 운영서버에 배포해서 이슈를 대응하고 다음 단계로 넘어가기 위해서 노력해야 겠다.

네이버페이 결재 기능이 오픈하면서 나이스페이 대사 작업에 이어 네이버페이 대사 작업을 해야 한다. 다행히 처음 모델링을 할 때 고민을 많이 해두어서 구현체만 잘 추가하면 될 듯한데 일단 구현을 먼저 해봐야 알 수 있을 듯 하다. 15일까지니까 개발에만 온전히 집중하면 완수 할 수 있을 것 같은데...운영 이슈 대응을 어떻게 하느냐에 따라 일정이 널널할지 촉박할지 달라질 듯 하다.

아래는 이번주에 발생한 이슈 또는 고민한 내용을 정리한 것이다.

## JPA Nullable Embedded

Java와 다르게 Kotlin에서는 nullable 객체를 명시적으로 선언해주어야 한다. JPA를 사용할 때 Embedded 객체를 사용함으로써 DB 테이블의 특정 컬럼을 묶어 클래스화 할 수 있는데, nullable할 필요가 있어서 nullable 객체로 선언해 주었다. 이때 단순히 `@Embedded`어노테이션을 사용하면 되는 것이 아니라 `@get:Embedded`와 같이 선언해주어야 하며 반드시 변경가능한 변수 타입인 `var`를 공개 매서드로 사용해야 한다. 이는 자바빈의 규칙을 따라야 한다는 것인데, 생성자로 `private var`를 사용하여 getter를 따로 두는 방식을 사용할 수 없으니 주의할 필요가 있다. (사실 Intellij가 워닝으로 잘 잡아주니까 따라서 고치면 된다 ㅎㅎ)

## 하향식 기능 개발

[오브젝트 디자인 스타일 가이드](http://www.yes24.com/Product/Goods/91167539){: target="\_blank" }를 읽으면서 인상 깊었던 문구를 적어본다. 이 개발 방법은 이전에 한번 시도해 본적이 있는데 잊고 있다가 다시한번 이책을 통해 리마인드 하게 되었고 앞으로 실천하기 위해 노력해야 겠다.

> 소프트웨어를 테스트할 때는 어느 상세 수준에서 작업하고 있는지도 생각해야 한다. 나 자신도 마찬가지지만 개발자는 더 작은 부품, 즉 나중에 기능을 완성하는 데 사용할 수 있는 구성요소를 작업하는데 치중하는 경향이 있다. 전체 기능에 대한 모든 것을 생각하고 모든 재료를 하나하나 모으기 시작한다. 저장소, 데이터베이스 테이블, 개체 등을 생성한다. 부품을 서로 연결하기 시작하면 몇 가지 초기 가정의 잘못 때문에 다시 만들어야 하는 부분이 보이기 시작하고, 결국 요소들이 조화롭게 연동하지 않는다는 것을 알게 된다. 그동안 개발에 쏟은 노력은 다소 낭비된다.
>
> 그와 반대로 일하길 권한다. 큰 그림을 먼저 보라는 뜻이다. 기능을 어떻게 사용할지 정의하고, 상호 작용의 개요를 먼저 작성한다. 바꿔 말해, 구축을 완료했을 때 해당 응용프로그램이 어떻게 동작하기를 기대하는지 고수준에서 명시한다. 성급히 저수준 상세 내용을 파고 들지 않도록 한다. 일단 응용 프로그램을 블랙박스로 취급하면서 무엇을 할 수 있어야 하는지 알고 난 다음에 더 깊은 수준으로 내려가 세부 구현을 시작해도 늦지 않다.
>
> 이러한 하향식 개발 스타일에서 응용프로그램의 행위를 명시하는 데 테스트를 사용함으로써 완전한 테스트 주도 개발이 가능해진다. 완전한 기능을 기술하는 고수준 테스트는 저수준 테스트를 모두 통과할 때까지 통과하지 못한다. 저수준 테스트를 통과하도록 하면서 동시에 점진적으로 고수준 테스트도 동과하도록 노력한다.

## Gradle Slack Plugin

[Gradle에서 Slack 메시지를 전송하는 플러그인](https://github.com/jongyoul/gradle-slack-plugin){: target="\_blank" }이 있어 사용해 보았다. Gradle 빌드 시 빌드 완료 후 슬렉 메시지를 보내게끔 설정하려고 하였는데, Jenkins의 slack plugin으로 대체하면서 아쉽게도 사용하지 않게 되었다. 다음에 기회가 있으면 기억해두었다고 사용해 보아야 겠다.

## spinnaker

쿠버네티스를 잘 다루던 동료가 CD 도구로 [spinnaker](https://spinnaker.io/){: target="\_blank" }를 소개해 주었다. 아직 사용해 보진 못했지만 CI 도구인 젠킨스를 CD처럼 사용하는 것보다 직관적이고 쿠버네티스와 연동이 아주 잘되어 있어서 베포의 성공여부, 손쉬운 파이프라인 구성 등 많은 장점을 누릴 수 있어서 한번 사용을 검토해 보아야 겠다.

## Spectacle

회사 동료가 무료 화면 분할 도구인 [Spectacle](https://www.spectacleapp.com){: target="\_blank" }을 소개해 주었다. 화면분할 도구는 유료 툴이 존재하는 것은 알고 있었지만 툴 구매에 돈쓰는걸 별로 안좋아하는 나에게 꼭 필요한 기능이 아니라 구매하지 않고 있었는데, 마침 동료가 무료툴을 알려주어서 설치해서 사용하고 있다. 왜 진작에 쓰지 않았나 싶다.

## 젠킨스 slack 플러그인

젠킨스에서 slack 메시지를 전송하는 플로그인인 [Slack Notification](https://plugins.jenkins.io/slack/){: target="\_blank" }을 사용해 보았다. 용도는 현재 개발중인 SKU 서비스의 코드가 마스터에 병합되면 자동으로 쿠버환경에 코드가 배포되도록 하였는데, 이때 배포 성공에 대한 슬렉 알림을 전송할 때 사용하였다. 생각보다 손쉽게 사용할 수 있어서 금방 적용 할 수 있었다.

## Gradle Task issue

배포 시 헬름 차트 배포를 Gradle Task로 등록하여 코드가 병합될 때 젠킨스에서 해당 Task를 실행하도록 설정하려고 하였다. 처음 Task를 만들어서 배포를 해보았는데 어찌된 일인지 원하지도 않은 타이밍에 코드가 배포되는 것을 볼 수 있었고 원인을 찾아보니 빌드 명령어를 실행하면 Gradle에서 Task들을 실행하는 것을 알게 되었다. 솔직히 원인은 알았지만 어떻게 우회할 수 있을지는 찾아보지 못했다. 아직 Gradle 문법이 익숙하지 않은 것도 한몫했다... Gradle 공부하자....

아무튼 그래서 해당 Task를 제거한 후 젠킨스에서 차트 스크립트를 실행하도록 배포 파이프라인을 변경하였다. Task 등록 시 이런 문제가 생기지 않도록 조심해야 겠다.

## datadog 연동

모니터링을 위해 Spring boot에서 Logback을 통해 Datadog 연동을 하였다. 연동방법은 [Datadog Document](https://docs.datadoghq.com/logs/log_collection/java/?tab=log4j){: target="\_blank" }에서 확인하자. 한번 튜토리얼을 적어봐야 겠다.

## JPA Map type 컬럼 사용

JPA에서 비정형 데이터를 하나의 컬럼에 저장하고자 하는경우 Map 타입을 사용하기도 한다. 그런 경우 아래와 같이 사용하면 된다. DB는 Mariadb을 사용했다.
이전엔 HashMapConverter를 기본으로 제공해 줬던것 같은데, 없는것 같아서 새롭게 만들었다.

```kotlin
@Lob
@Column(columnDefinition = "text")
@Convert(converter = MapAttributeConverter::class)
var itemInformation: Map<String, String>
```

```kotlin
class MapAttributeConverter : AttributeConverter<Map<String, String>, String> {
    override fun convertToDatabaseColumn(attribute: Map<String, String>?): String {
        if (attribute == null) {
            return ""
        }

        return jacksonObjectMapper().writeValueAsString(attribute)
    }

    override fun convertToEntityAttribute(dbData: String?): Map<String, String> {
        if (dbData.isNullOrBlank()) {
            return mapOf()
        }

        return try {
            jacksonObjectMapper().readValue(dbData, jacksonTypeRef<Map<String, String>>())
        } catch (e: JsonParseException) {
            mapOf()
        }
    }
}
```

## Logbak Configuration Class

Spring Boot에서 Logback을 설정하는 예제들을 보면 대부분 XML로 설정하는 것을 볼 수 있다. 개인적으로는 Configuration Class를 만들어서 사용하는 것을 선호하는데 Spring Boot에서 다른 설정들은 다 Configuration Class를 만들 수 있게 해두었는데 Logback 설정을 XML로만 작성하도록 해두지 않았을 거란 생각이 든다. 한번 깊게 찾아보는 시간을 가져봐야겠다!!

## Grep Console

Intellij Plugin 중에 [Grep Console](https://plugins.jetbrains.com/plugin/7125-grep-console){: target="\_blank" }이라는 플러그인이 있다는 것을 알게 되었다. 해당 플러그인은 콘솔창에서 표시되는 글자들을 원하는 글자들만 노출되도록 필터를 줄 수 있는 기능이다. 디버깅할 때 유용할 것 같다.

## kotlinfixture 타임존 설정 기능 추가

이전에 kotlinfixture에서 타임존을 factory에서 설정하지 못해 불편했던 이슈가 있다. 특히 ZonedDateTime을 사용할 때 그런데, 테스트 시 euals 비교를 할 때 타임존이 달라서 테스트가 실패하는 **거짓 양성**을 발생시켰다. 그래서 테스트를 성공시키기 위해 하나아나 타임존을 설정해주는 꼼수를 사용했는데 이제는 그럴 필요가 없어졌다. 해당 이슈는 [링크](https://github.com/appmattus/kotlinfixture/issues/65){: target="\_blank" }를 참고하자.

