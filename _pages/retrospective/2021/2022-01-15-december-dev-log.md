---
layout: single
title: "2021년 12월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2022-december-dev-log/
---

참 시간이 빠르다. 어쩌다보니 다사다난했던 2021년의 12월도 지났다. 2021년에 대한 회고는 [2021년 회고](https://veluxer62.github.io/retrospective/2021-retrospective/){: target="\_blank" }에서 다루기로 하고 12월에 겪었던 개발적인 이슈들을 적어보자.

## Elasticsearch field boost

ES의 Mutimatch 쿼리에서 아래와 같이 field에 `^`를 주면 socre 점수를 조절할 수 있다. 이를 [Field boost](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#field-boost){: target="\_blank" }라 하는데 스코어별 정렬 시 사용하면 아주 유용하다.

```
GET /_search
{
  "query": {
    "multi_match" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ]
    }
  }
}
```

## Jackson PropertyNamingStrategies

Jackson 이용시 Snake case를 Camel case로 변경해줘야 하는 경우가 있을 수 있다.
많지않은 프로퍼티나 특정 프로퍼티만 변경해줘야하는 경우 `JsonProperty`를 이용해서 아래와 같이 사용할 수 있다.

```kotlin
data class Foo(
  @JsonProperty("first_name")
  val firstName: String,

  @JsonProperty("last_name")
  val lastName: String,
)
```

하지만 필드가 많은 경우에는 위와 같이 프로퍼티 하나하나마다 `JsonProperty`를 주는 것은 비효율적으로 보인다. 그럴때 사용하면 좋은게 바로 `PropertyNamingStrategies`이다.

```kotlin
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class Foo(
  val firstName: String,
  val lastName: String,
)
```

## Spring Boot에서 resource 파일 읽는 법

Spring Boot에서 테스팅이나 혹은 다른 이유로 인해 resource 파일을 읽어들여야 할때 다양한 방법이 존재한다.
구글링을 해보면 많은 방법들을 볼 수 있는데 개인적으로 가장 마음에 드는 방법이 있어 적어본다.

```kotlin
val resource = ClassPathResource("something.txt")
val content = IOUtils.toString(resource.inputStream, StandardCharsets.UTF_8)
```

개인적으로 여러 변환 코드 없이 단 두줄로 내용을 읽어들이는 코드라 마음에 든다.

## curioustore

코드를 작성할때 가장 힘든게 변수명 짓기다.

이를 도와주는 좋은 사이트가 있어서 적어둔다.

[https://curioustore.com](https://curioustore.com){: target="\_blank" }

## AWS RDS Maximum database connections

RDS의 connection 최대 개수는 인스턴스의 사양에 따라 다르다.

계산 공식은 DB에 따라 다른데, MariaDB의 경우 `DBInstanceClassMemory / 12582880`이다. 다른 데이터 베이스들에 대한 정보를 확인하려면 아래 링크를 참고하자.

[https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Limits.html](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Limits.html){: target="\_blank" }

## README 뱃지

요즘 README에 뱃지를 다는 재미에 빠졌다..;; Circle CI에 뱃지를 다는 방법도 최근 발견했는데, 아래 링크를 참고해서 추가하면 된다.

[https://circleci.com/docs/2.0/status-badges/](https://circleci.com/docs/2.0/status-badges/){: target="\_blank" }

기본적으로는 기본 브랜치를 기준으로 pass 여부를 보여주는데 특정 브랜치 기준으로도 할 수 있다.

이외에도 Gradle plugin을 통해서 [jacoco coverage generator](https://github.com/dawnwords/jacoco-badge-gradle-plugin){: target="\_blank" }도 추가하는 방법도 있다.

아래 사진과 같이 현재 개발중인 서비스의 README에 뱃지를 적용해보았다.

![readme-badge](/assets/images/retrospective/readme-badge.png)

## Git Action add-and-commit

Git Action에서 커밋을 자동으로 추가해 주는 등의 git 명령어를 실행해주는 Action이 있어서 적어둔다.

[https://github.com/EndBug/add-and-commit](https://github.com/EndBug/add-and-commit){: target="\_blank" }

현재 회사에서는 주로 Circle CI를 사용해서 Git Action 사용비중이 높진 않지만 앞으로 써봄직한 것 같다.

## jacoco badge generator

[https://github.com/cicirello/jacoco-badge-generator](https://github.com/cicirello/jacoco-badge-generator){: target="\_blank" }는 현재 우리 회사에서는 Github을 팀플랜으로 사용하기 때문에 사용할 수 없다.

그래서 [https://github.com/dawnwords/jacoco-badge-gradle-plugin](https://github.com/dawnwords/jacoco-badge-gradle-plugin){: target="\_blank" }을 사용하기로 결정하였다.

다만 아쉬운점은 gradle plugin은 직접 수동으로 관리해주어야 한다는 점이다. 별로도 파이프라인에서 플러그인을 실행해서 변경하도록 할 수 있을 것 같은데 좀 더 찾아봐야겠다.

## kfactory

최근 테스트 fixture를 생성하기 위한 도구로 [kfactory](https://github.com/bluegroundltd/kfactory){: target="\_blank" }를 도입해서 사용하고 있다.

이전까지는 [kotlinFixture](https://github.com/appmattus/kotlinfixture){: target="\_blank" }를 도입해서 사용하고 있었는데, 동일한 클래스에 값에 따라 네이밍을 준다던지 하는 편의기능이 kfactory가 더 많은것 같아서 kfactory를 사용하기로 결정하였다.

이와 함께 사용하기로 한 라이브러리가 있는데 바로 [java-faker](https://github.com/DiUS/java-faker){: target="\_blank" }이다. 꼭 필요하진 않지만 fixture 생성 시 실제 데이터와 유사한 값을 랜덤하게 생성해 주어서 나쁘지 않은것 같다.

## Mockk constructor verify

[Mockk](https://mockk.io/){: target="\_blank" } 라이브러리는 아직 constructor verify를 제공해 주지 않는다. 이슈는 올라와 있는 상태인데 아직 적용검토는 하지 않고 있나보다.

이슈링크: [https://github.com/mockk/mockk/issues/163](https://github.com/mockk/mockk/issues/163){: target="\_blank" }

## garticphone

최근 지인들과 [garticphone](https://garticphone.com/ko){: target="\_blank" }을 함께 플레이 하였는데 너무 재미있었다.

## Mockk private functions mocking / dynamic calls

[Mockk](https://mockk.io/){: target="\_blank" } 라이브러리에서도 PowerMock과 같이 사설함수에 대한 mocking을 제공해준다. 사실 이 방법은 Test에서 좋은 방법은 아니다. 하지만 실무에서 테스트코드를 작성하다보면 검증력을 좀더 높이거나 어쩔수 없는 경우 (예를 들어 JPA의 연관관계 매핑과 같이 특정환경에 의해 동작하는 코드) 사용할 수 밖에 없는데, private property를 아무리 적용해서 사용해보아도 정상작동하지 않았다.

원인을 찾다보니 컴파일 후 바이트 코드에서 private property는 getter와 setter를 만들지 않는다는 것을 발견할 수 있었다. 그래서 `getProperty`를 사용하면 응답하지 않고 정상동작하지 않는 것이었다.

#### private property를 컴파일한 결과

```kotlin
data class Foo(
  private var a: String,
)
```

```
public final class com/spoqa/cart/application/configuration/security/Foo {


  // access flags 0x2
  private Ljava/lang/String; a

  // 생략...
}
```

해결방법은 간단한데 private 대신 protected를 사용하면 된다. 하지만 위에서 말했다시피 좋은 패턴은 아니며 되도록이면 지양하는 방향으로 구현하자.

#### protected property를 컴파일한 결과

```kotlin
data class Foo(
  protected var a: String,
)
```

```
public final class com/spoqa/cart/application/configuration/security/Foo {


  // access flags 0x2
  private Ljava/lang/String; a
  @Lorg/jetbrains/annotations/NotNull;() // invisible

  // access flags 0x14
  protected final getA()Ljava/lang/String;
  @Lorg/jetbrains/annotations/NotNull;() // invisible
   L0
    LINENUMBER 4 L0
    ALOAD 0
    GETFIELD com/spoqa/cart/application/configuration/security/Foo.a : Ljava/lang/String;
    ARETURN
   L1
    LOCALVARIABLE this Lcom/spoqa/cart/application/configuration/security/Foo; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x14
  protected final setA(Ljava/lang/String;)V
    // annotable parameter count: 1 (visible)
    // annotable parameter count: 1 (invisible)
    @Lorg/jetbrains/annotations/NotNull;() // invisible, parameter 0
   L0
    ALOAD 1
    LDC "<set-?>"
    INVOKESTATIC kotlin/jvm/internal/Intrinsics.checkNotNullParameter (Ljava/lang/Object;Ljava/lang/String;)V
   L1
    LINENUMBER 4 L1
    ALOAD 0
    ALOAD 1
    PUTFIELD com/spoqa/cart/application/configuration/security/Foo.a : Ljava/lang/String;
    RETURN
   L2
    LOCALVARIABLE this Lcom/spoqa/cart/application/configuration/security/Foo; L0 L2 0
    LOCALVARIABLE <set-?> Ljava/lang/String; L0 L2 1
    MAXSTACK = 2
    MAXLOCALS = 2

  // 생략....
}
```

참고로 Intellij에서 kotlin bytecode를 보는 방법은 `Tools` > `kotlin` > `Show Kotlin Bytecode`를 실행하면 된다.

## ponicode

지인이 테스트 코드를 자동 생성해주는 도구로 [ponicode](https://www.ponicode.com/){: target="\_blank" }를 소개해 주었다. 아직 써보진 않았지만 시간 나면 한번 만져봐야겠다.

## clonable clone

cloneable의 clone 함수는 기본적으로 protected 함수이다. 외부에서 사용하도록 하려면 public으로 정의해야 한다.

[https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#clone()](<https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#clone()>){: target="\_blank" }

## Circle CI only build pull requests

Circle CI에서 별다른 설정이 없으면 repository에 remote push때마다 CI가 실행되게 된다. PR 이후에야 CI가 실행되면서 테스트 체크를 해주는게 맞지만 PR을 올리지도 않앗는데 CI가 실행되는것은 불필요한 낭비이다. (Circle CI는 실행시간별로 과금한다.)

PR이 작성되었을때에만 build를 실행하도록 하기 위해서는 아래와 같이 `Only build pull requests`를 활성화 해주면 된다.

![only-build-pull-request](/assets/images/retrospective/only-build-pull-request.png)

## git prune option enable

아래와 같이 prune 옵션을 활성화 하면 git pull 시에도 prune이 실행되어 remote branch를 트래킹할 수 있다.

```
git config --global fetch.prune true
```
