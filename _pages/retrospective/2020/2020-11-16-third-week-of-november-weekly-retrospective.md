---
layout: single
title: "2020년 11월 3주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-third-week-of-november-weekly-retrospective/
---

저번주는 그 전주에 너무 달려서 그런지 컨디션 난조로 기능 구현을 많이 못했던 것 같다. 거기다가 회의가 너무 많았다. 회의좀 줄여줘 ㅠㅠ

이제 점점 개발중인 상품 서비스에 대한 기능이 어느정도 나와간다. 처음부터 구현기능에 대한 세부논의를 하고 개발을 했더라면 기능을 갈아엎어야 하는 사태는 나오지 않았을 텐데라는 아쉬움이 있지만 지금이라도 구성원들이 지식이 쌓여가면서 기능에 대한 세부사항을 얘기하고 있어서 조금씩이나마 고쳐나가고 있다.

도메인 주도 개발에서 개발을 할때 유비쿼터스 언어를 정의하는 것을 먼저 해야 한다고 말한다. 이번주에 그 필요성을 뼈저리게 느꼈는데, SKU라는 단어를 여기에서는 다른의미로 사용하기 시작하면서 SKU라는 단어에 대한 의미를 부서마다 다르게 이해하고 있었다. 그러다보니 상품에 대한 기능 구현에도 각자 의견이 달라 의견을 합치고 조율하는데 애를 먹었다. 한편으로는 지금 시점에 이런 논의가 되는게 이해가 되기도 하는데, 이전 회사에서도 그렇고 지금도 그렇고 막상 처음부터 유비쿼터스 언어를 정의하려고 하면 참가자들이 지루해 하거나 왜 해야 하는지 잘 못느껴서 집중을 못하는 느낌을 많이 받았다. 반면에 지금과 같이 문제를 인지하고 나니 사람들이 자신의 의견을 적극적으로 내비치는 것을 볼 수 있었다. 결국 유비쿼터스 언어를 정의해야 하는 필요성을 모두 느끼는 상황에서 프로젝트 초기에 정의하려는 회의를 해야 하지 않나 싶다. 이 다음 프로젝트에서는 구성원들이 유비쿼터스 언어 정의를 먼저하자고 하면 좀더 적극적인 모습을 보이지 않을 까 싶다.

아래는 금주동안 내가 겪었거나 고민햇던 이슈들을 적은 내용이다.

## Git multi command alias

git의 다중명령어를 alias로 주려면 `!`를 사용한 후 git을 함게 써주면 된다

아래는 리모트 패치 후 리베이스 해주는 명령어이다.

```bash
fr = !git fetch --prune && git rebase origin/master
```

## ElementCollection in Embeddable

Embeddable 클래스 안에 ElementCollection을 넣어서 정의하면 Auto-DDL 시 Entity의 하위 자식으로 들어가는 것을 볼 수 있다.

하지만 ElementCollection 값이 저장이 되지 않는 것을 볼 수 있었다. 그래서...사용하기 힘들거 같다.

사용 예시

```kotlin
@Embeddable
data class InventoryEmbed(
    @Column(name = "inventory_quantity", nullable = false)
    var quantity: Int,

    @Column(name = "inventory_minimum_quantity_for_notice", nullable = false)
    var minimumQuantityForNotice: Int,

    @Column(name = "inventory_limited", nullable = false)
    var isLimited: Boolean,

    @ElementCollection
    @CollectionTable(name = "kit_inventory_manual_change_histories", joinColumns = [JoinColumn(name = "kit_id")])
    var inventoryHistories: MutableList<InventoryManualChangeHistoryEmbed> = mutableListOf()
)
```

## 깃 이모지

깃에서 이모지 기능을 사용한 사례에 대한 글이 있어 적어본다. 이전에 커밋을 이모지로만 적어서 고생했다던 동료분의 썰을 들은적이 잇는데....

[https://pilgwon.github.io/post/gitmoji](https://pilgwon.github.io/post/gitmoji){: target="\_blank" }

## PageImpl Serialize

PageImpl을 Jackson 라이브러리에서 serialize 할때 오류가 발생한다. 해당 이슈는 [stackoverflow](https://stackoverflow.com/questions/52490399/spring-boot-page-deserialization-pageimpl-no-constructor/52509886){: target="\_blank" }에도 올라와 있는데, 가장 많은 추천을 받은 답변은 커스텀 객체를 만들어 PageImpl을 상속 받아 사용하는 것이다. 사용하지 않는 생성자를 만들어야 한다는 점이 좀 아쉽긴하다.

다른 방법으로 [openfeign](https://woowabros.github.io/experience/2019/05/29/feign.html){: target="\_blank" }을 사용하는 방법이 있는데, 해당 문제를 해결하기 위해서 의존성을 추가하는건 좋아보이지 않아 패스했다. 다만 openfeign에서 Http Client는 좋아보여서 기회가 된다면 사용해 봐야겠다.

코틀린 예제

```kotlin
@JsonIgnoreProperties(ignoreUnknown = true)
class PageDto<T> : PageImpl<T> {
    @JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
    constructor(
        @JsonProperty("content") content: List<T>,
        @JsonProperty("number") number: Int,
        @JsonProperty("size") size: Int,
        @JsonProperty("totalElements") totalElements: Long,
        @JsonProperty("pageable") pageable: JsonNode,
        @JsonProperty("last") last: Boolean,
        @JsonProperty("totalPages") totalPages: Int,
        @JsonProperty("sort") sort: JsonNode,
        @JsonProperty("first") first: Boolean,
        @JsonProperty("numberOfElement") numberOfElement: Int
    ) : super(content, PageRequest.of(number, size), totalElements)

    constructor(content: List<T>) : super(content)

    constructor(content: List<T>, pageable: Pageable, total: Long) : super(content, pageable, total)
}
```

## JPA Pessistic lock

Spring Data JPA에서 pessimistic lock을 걸어도 연관테이블에는 잠금이 걸리지 않는다.
그래서 하위 테이블 잠금을 사용하려면 개별 Table에 잠금을 걸어줘야 한다. 근데 하위 테이블까지 잠금을 걸어줘야 하는 상황이면 설계를 다시해야 하지 않나 싶다. 낙관적 잠금이면 몰라도 비관적 잠금이면 성능이슈나 데드락 문제가 발생할 수 있으니 조심스러워야 한다.

## JPA Lock timeout

JPA에서 잠금 타임아웃 설정 방법

```
@Lock(LockModeType.PESSIMISTIC_WRITE) 
@QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value = "5")}) 
```

타임아웃 단위가 밀리초인줄 알았는데 초단위였다!! 꼭 테스트 해보길

## ElementCollection update

ElementCollection을 사용할 때 데이터가 모두 삭제되고 추가되는 것을 방지하려면 두가지 방법이 있다.

1. OrderColumns를 추가한다.

  해당 방법은 List Collection을 사용할 때 사용할 수 있는 방법으로 중복된 데이터를 넣을 수 있다는 장점이 있다. 다만 order 컬럼이 추가되어야 하는 문제가 있다.

  ```kotlin
  @ElementCollection
  @OrderColumns
  var inventoryHistories: MutableList<InventoryManualChangeHistoryEmbed> = mutableListOf()
  ```

2. Set Collection을 사용한다.

  중복되지 않는 Collection을 사용할 때 사용할 수 있는 방법이다. order 컬럼이 추가되지 않아도 되는 장점이 있지만 중복데이터가 존재하는 경우와 equals를 구현해주어야 한다는 문제가 있다.

  ```kotlin
  @ElementCollection
  var inventoryHistories: MutableSet<InventoryManualChangeHistoryEmbed> = mutableSetOf()
  ```

## Azure pipline codecoverage

에저 파이프라인에서 코드 커버리지를 확인할 수 있는 설정이 있다.

[https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/gradle?view=azure-devops](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/gradle?view=azure-devops){: target="\_blank" }

설정 테스트를 해보았는데 잘못 적용했는지 커버리지가 테스크 결과에 표시되지 않고 있다. 좀더 검토해봐야 겠다.

설정 예시

```yaml
# Gradle
# Build your Java project and run tests with Gradle using a Gradle wrapper script.
# Add steps that analyze code, save build artifacts, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/java

trigger:
  - master

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: Gradle@2
    inputs:
      workingDirectory: ''
      gradleWrapperFile: 'gradlew'
      gradleOptions: '-Xmx3072m'
      javaHomeOption: 'JDKVersion'
      jdkVersionOption: '1.11'
      jdkArchitectureOption: 'x64'
      publishJUnitResults: true
      testResultsFiles: '**/TEST-*.xml'
      tasks: 'clean build'
      codeCoverageToolOption: JaCoCo
      codeCoverageClassFilesDirectories: 'build/classes/kotlin/main/net/class101/inventory/'
      codeCoverageClassFilter: -:net.class101.inventory.infrastructure.config.*
  - task: PublishCodeCoverageResults@1
    inputs:
      codeCoverageTool: 'JaCoCo'
      summaryFileLocation: $(System.DefaultWorkingDirectory)/class101-inventory-service/build/reports/jacoco/test/jacocoTestReport.xml
```

## Spring Boot 2.4.0

Spring Boot 2.4.0이 출시되었다. 릴리즈 노트는 아래 링크를 통해 확인해보자

[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.4-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.4-Release-Notes){: target="\_blank" }

## spring-cloud-starter-sleuth 이슈

Spring Boot 2.4.0 업그레이드 하고 나니까 빌드가 되지 않는다. 확인해보니 sleuth 이슈로 보이는데 sleuth의 3.0.0 버전에서 부터 Spring Boot 2.4.0 지원을 해준다고 한다.

[spring-cloud/spring-cloud-sleuth#1775](spring-cloud/spring-cloud-sleuth#1775){: target="\_blank" }

## 스프링 2.4.0 대응 Mockk 업데이트

Mockk 라이브러리도 Spring 2.4.0 업데이트 대응 업데이트를 진행했다.

[https://github.com/Ninja-Squad/springmockk/releases/tag/3.0.0](https://github.com/Ninja-Squad/springmockk/releases/tag/3.0.0){: target="\_blank" }
