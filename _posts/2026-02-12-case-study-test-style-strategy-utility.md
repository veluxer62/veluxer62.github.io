---
layout: single
title: 지속가능한 테스트 작성 전략
categories:
  - EXPLANATION
tags:
  - testing
  - kotest
  - mockk
  - TDD
toc: true
toc_sticky: true
toc_h_max: 2
excerpt: 실무에서 좀 더 테스트를 오래 유지하기 위해 실무에서 고민하고 적용해왔던 읽히는 테스트, 전략적 검증, 공통 유틸을 통해 지속 가능한 테스트 작성법을 정리하고 소개한다.
---

## 들어가며

프로젝트가 커질수록 테스트는 단순히 “동작 확인”을 넘어, 팀의 개발 속도와 품질 문화를 결정하는 핵심 장치가 된다. 기능은 빠르게 늘어나는데 테스트가 이를 따라오지 못하면, 작은 수정에도 영향 범위를 예측하기 어려워지고 배포에 대한 심리적 부담이 커진다. 반대로 테스트 코드가 일정한 기준 아래 축적되면, 새로운 기능을 추가하거나 리팩터링할 때도 팀이 훨씬 안정적으로 움직일 수 있다.

이 글은 이런 문제의식에서 출발했다. “좋은 테스트”를 원칙만으로 이야기하기보다, 실제 서비스 코드베이스에 쌓인 사례를 통해 어떤 방식이 실무에서 오래 살아남는지 정리해보려 한다. 특히 테스트를 작성할 때 자주 마주치는 질문들—어떤 테스트를 어디까지 작성해야 하는지, 중복되는 준비 코드를 어떻게 줄일지, 권한/외부 연동/시간 같은 까다로운 요소를 어떻게 다룰지—에 대해 실전 관점으로 풀어볼 예정이다.

그래서 글의 흐름도 실제 작업 순서를 가져가려 한다. 먼저 **읽고 이해하기 쉬운 테스트 문장과 구조**를 살펴보고, 그다음 **무엇을 어떤 경계에서 검증할지에 대한 판단 기준**을 다루며, 마지막으로 **반복 작업을 줄이고 팀 전체 생산성을 높여주는 공통 도구들**을 소개하려고 한다. 이름만 거창한 원칙이 아니라, 팀이 매일 마주하는 코드 리뷰와 유지보수 상황에서 바로 체감할 수 있는 기준을 중심으로 정리할 생각이다.

즉, “테스트를 많이 쓰는 법”보다는 **“테스트를 계속 유지할 수 있게 쓰는 법”**에 초점을 맞춰보고자 한다.

## 읽히는 테스트는 어떻게 만들어지는가

좋은 테스트는 복잡한 기법에서 시작하지 않는다. 먼저, **읽히는 이름**과 **예측 가능한 구조**에서 출발한다. 테스트가 실패했을 때 “무엇이 깨졌는지”보다 먼저 “왜 이 테스트가 존재하는지”를 이해할 수 있어야 하기 때문이다. 이 장에서는 실제 사례를 바탕으로 테스트 문장을 어떻게 작성하고, 시나리오를 어떤 순서로 배치해야 팀 전체가 같은 속도로 이해할 수 있는지 살펴보겠다.

### 테스트 이름만 바꿔도 리뷰 품질이 올라간다

리뷰에서 가장 먼저 읽히는 것은 구현이 아니라 **테스트 이름**이다. 테스트 이름이 명확하면 리뷰어는 코드를 열기 전에 “누가(권한/주체) 무엇을(행동) 왜(기대결과) 검증하는지”를 빠르게 파악할 수 있다. 반대로 이름이 짧고 추상적이면, 같은 검증이어도 리뷰어가 본문을 끝까지 읽어야 의도를 이해하게 된다.

예를 들어 아래처럼 **권한 + 행동 + 결과**를 한 문장으로 드러내면, 실패 로그만 봐도 시나리오가 바로 떠오른다.

- 점주 권한으로 견적서를 조회하면 견적서를 반환한다.
- 유통사 권한으로 견적서 목록을 조회하면 PERMISSION_DENIED 오류를 반환한다.
- 필터로 청구서 목록을 조회하면 조건에 맞는 청구서 목록을 반환한다.

반면 아래처럼 이름이 상대적으로 짧은 케이스는, 테스트 본문을 보기 전에는 맥락이 부족할 수 있다.

- 매장의 활성화된 청구서 목록을 조회한다.
- ID 목록으로 청구서 목록을 조회한다.
- 소숫점 첫째자리는 버림한다.

물론 이런 이름도 틀린 것은 아니다. 다만 리뷰 효율을 높이려면, **입력 조건과 기대 결과를 조금 더 드러내는 형태**로 바꾸는 것이 좋다. 예를 들면 다음과 같다.

- `매장 ID가 주어지면 활성화 상태의 청구서 목록만 반환한다`
- `청구서 ID 목록이 주어지면 해당 ID와 일치하는 청구서만 반환한다`
- `수수료 계산 결과에서 소수점 첫째 자리는 버림 처리한다`

실무에서 유용한 최소 규칙은 다음과 같다.

1. 이름에 가능한 한 **주체(권한/역할)**를 넣는다.
2. 이름에 **행동(조회/생성/취소/갱신)**을 넣는다.
3. 이름 끝에 **기대 결과(반환/예외/오류코드)**를 넣는다.

이 세 가지가 갖춰지면, 테스트 본문을 자세히 읽지 않아도 리뷰어가 의도를 빠르게 판단할 수 있고, 회귀가 발생했을 때도 실패 로그만으로 원인 후보를 좁히기 쉬워진다.

### Given-When-Then을 코드 블록이 아니라 사고 순서로 쓴다

많은 팀이 `Given`, `When`, `Then` 주석을 쓰지만, 주석이 있다고 해서 자동으로 읽기 쉬운 테스트가 되지는 않는다. 핵심은 주석의 유무가 아니라 사고의 순서이다. 즉, “상황을 만들고(Given) → 한 가지 행동을 수행하고(When) → 그 행동의 결과를 검증한다(Then)”는 흐름이 코드의 위에서 아래로 자연스럽게 이어져야 한다.

예를 들어 아래의 테스트를 보면 Given에서 fixture와 mock 반환값을 정리하고, When에서 실제 호출을 단 한 번 수행한 뒤, Then에서 반환값을 검증하는 것을 볼 수 있다.

```kotlin
context("getAllByFilter") {
    test("필터와 페이징 정보가 주어지고 청구서 목록을 조회하면 조건에 맞는 청구서 목록을 반환한다.") {
        // Given
        val vendor = VendorFactory().produce()
        val pageable = Pageable.unpaged()
        val filter =
            BillFilter(
                vendorId = vendor.id,
            )

        // When
        val actual = billService.getAllByFilter(filter, pageable)

        // Then
        val expected = testHelper.getBills(vendor.id)
        actual shouldBe expected
    }
}
```

오류에 대한 검증도 살펴보자. 아래는 권한/입력값을 준비하고, 요청을 1회 실행한 뒤, 기대한 권한 오류를 검증하는 코드이다.

```kotlin
context("quotationImageUploadUrls") {
    test("매장 권한으로 견적 이미지 업로드 url을 조회하면 PERMISSION_DENIED 오류를 반환한다.") {
        // Given
        val token = testHelper.generateStoreToken()

        val query =
            """
            query quotationImageUploadUrls(${"$"}fileNames: [String!]!) {
                quotationImageUploadUrls(fileNames: ${"$"}fileNames) {
                uploadUrls {
                    uploadUrl
                    objectKey
                }
                }
            }
            """.trimIndent()
        val fileNames = listOf("test1.jpg", "test2.jpg")
        val variables = mapOf("fileNames" to fileNames)

        // When
        val actual =
            clientBuilder.token(token).build()
                .executeQuery(query, variables)
                .errors

        // Then
        actual shouldPermissionDeniedWith listOf("quotationImageUploadUrls")
    }
}
```

이처럼 테스트 종류가 달라도 `Given-When-Then`의 사고 순서가 유지되면, 리뷰어는 코드를 따라 내려가기만 해도 “무엇을 검증하려는 테스트인지”를 놓치지 않게 된다.

반대로 `Given-When-Then`이 사고 순서로 정리되지 않으면 다음 문제가 자주 생긴다.

- Given 구간에 검증 코드가 섞여 준비/결과의 경계가 흐려짐
- When에서 두 개 이상의 액션을 실행해 실패 원인 추적이 어려워짐
- Then에서 핵심 검증과 부가 검증이 뒤섞여 테스트 의도가 흐려짐

실무에서 적용하기 쉬운 기준은 다음과 같다.

1. **Given은 입력과 전제만** 준비한다. (fixture, stub, 환경)
2. **When은 행동 하나만** 실행한다. (함수 호출/요청 전송)
3. **Then은 기대 결과를 계층화**한다. (핵심 결과 → 부가 검증 순)

결국 `Given-When-Then`은 “주석 템플릿”이 아니라, 리뷰어와 미래의 내가 같은 사고 순서로 테스트를 읽게 만드는 설계 규칙이다. 이 원칙이 지켜지면 테스트가 길어져도 의도가 흐려지지 않고, 실패가 났을 때 어느 구간에서 문제가 시작됐는지 훨씬 빠르게 좁힐 수 있다.

### 중복보다 더 위험한 것은 ‘의도 숨김’이다

테스트 리팩터링을 하다 보면 가장 먼저 줄이고 싶은 것이 중복이다. 하지만 실무에서는 **중복 자체보다 의도를 가리는 추상화가 더 위험**할 때가 많다. 테스트는 실행 코드이기도 하지만, 동시에 팀이 읽는 문서이기 때문이다. 눈앞의 중복을 없애는 데만 집중하면, 정작 “이 테스트가 무엇을 보장하는지”가 흐려질 수 있다.

아래 코드는 이러한 문제에 대한 균형점을 잘 보여준다. 권한 검증이 반복되지만, 이를 무조건 하나의 거대한 헬퍼로 감추지 않고 `testDeniedRoles` 같은 목적형 유틸로만 공통화한다.

```kotlin
context("quotationImageUploadUrls") {
    val query =
        """
        query quotationImageUploadUrls(${"$"}fileNames: [String!]!) {
            quotationImageUploadUrls(fileNames: ${"$"}fileNames) {
            uploadUrls {
                uploadUrl
                objectKey
            }
            }
        }
        """.trimIndent()

    testDeniedRoles(
        allowRoles = setOf(Constants.STORE_ADMIN, Constants.STORE_MANAGER),
    ) { label, token ->
        test("$label 권한으로 견적 이미지 업로드 url을 조회하면 PERMISSION_DENIED 오류를 반환한다.") {
            val variables = mapOf("fileNames" to listOf("test1.jpg", "test2.jpg"))

            val actual =
                clientBuilder.token(token).build()
                    .executeQuery(query, variables)
                    .errors

            actual shouldPermissionDeniedWith listOf("quotationImageUploadUrls")
        }
    }
}
```

핵심은 공통화 범위가 “권한 조합 반복”에만 머물고, **쿼리/입력/검증 의도는 테스트 본문에 남아 있다**는 점이다. 반대로 조회/검증까지 한 번에 수행하는 범용 헬퍼로 바꾸면, 실패 시 원인 위치를 찾기 어려워진다.

또 다른 좋은 사례는 Fixture/Factory 패턴이다. 기본값은 재사용하되, 시나리오 핵심 값은 테스트 본문에 남기기 때문에 의도가 숨겨지지 않는다.

```kotlin
test("가격정책이 비어있으면 기본 수수료를 반환한다.") {
    // Given
    val sut =
        fixture<NicepayProperties>().copy(
            feeRules = listOf(),
        )

    // When
    val actual = sut.toPgFee()

    // Then
    assertSoftly(actual) {
        it.transferPgFee shouldBe 10000.toBigDecimal()
        it.cardFeeRate shouldBe 0.03f.toBigDecimal()
    }
}

test("품목이 0개인 주문서를 생성시도하면 예외를 던진다") {
    // Given
    val emptyProducts = listOf()

    // Expect
    val actual =
        shouldThrow<IllegalArgumentException> {
            OrderSheetFactory(
                replaceOrderSheetProductDataList = { emptyProducts },
            ).produce()
        }

    actual.message shouldBe ErrorMessage.INVALID.ORDER_SHEET_PRODUCTS_EMPTY.message
}

test("점주 권한으로 추천한 매장 ID 를 등록하면 점주의 관리 매장의 추천 매장으로 등록한다.") {
    // Given
    val referrerStore = testHelper.매장_생성(name = "추천매장")

    val referredStore = testHelper.매장_생성()
    val manager = referredStore.managers.last()

    val input = RecordReferrerStoreInput(referrerStoreId = referrerStore.id)
    val variables = mapOf("input" to input)

    // When
    val actual =
        clientBuilder
            .token(testHelper.점주_토큰_생성(manager.id))
            .build()
            .executeQuery(mutation, variables)
            .fetchAsObject(
                "recordReferrerStore",
                typeRef<RecordReferrerStore>(),
            )

    // Then
    assertSoftly(actual.referrerStore) {
        it.id shouldBe referrerStore.id
        it.name shouldBe "추천매장"
    }

    val storeReferral = testHelper.추천_매장_조회(referredStore.id)
    assertSoftly(storeReferral) {
        it.referrerStoreId shouldBe referrerStore.id
        it.referredStoreId shouldBe referredStore.id
    }
}
```

반대로 의도 숨김이 발생하는 전형적인 패턴은 다음과 같다.

- 여러 단계의 fixture 생성을 `createDefaultScenario()` 같은 이름으로 한 번에 감싸 버려, 어떤 전제가 필요한지 테스트 본문에서 보이지 않는 경우
- 조회/생성/검증까지 수행하는 범용 헬퍼를 만들어 When과 Then이 호출 한 줄 뒤로 숨어 버리는 경우
- 공통화 과정에서 테스트 이름과 본문의 어휘가 달라져, “테스트가 말하는 것”과 “실제로 하는 것”이 어긋나는 경우

아래는 의도 숨김이 발생하기 쉬운 안티패턴 예시이다.

```kotlin
// 안티패턴 1: 전제가 통째로 숨겨져 테스트 의도가 보이지 않음
val scenario = createDefaultScenario() // 어떤 권한/상태/시간 조건인지 본문에서 알기 어려움

// 안티패턴 2: 행동 + 검증이 헬퍼 뒤로 숨겨짐
executeAndAssertQuotationSuccess(scenario)
```

```kotlin
// 안티패턴 3: 이름은 '권한 거부'인데 실제 본문은 성공 경로를 검증
test("점주 권한으로 조회하면 PERMISSION_DENIED 오류를 반환한다.") {
    val actual = clientBuilder.token(storeAdminToken).build()
        .executeQuery(query, variables)
        .fetchAsObject("quotation", typeRef<QuotationQueryModel>())

    actual.id shouldBe expectedId
}
```

그래서 중복 제거는 “얼마나 많이 지웠는가”보다 “의도를 얼마나 남겼는가”로 판단해야 한다. 앞서 소개한 코드처럼 `Factory` 기반 데이터 생성, `forAll` 기반 케이스 확장, 권한 테스트 유틸 같은 **작은 단위의 목적형 추상화**는 중복을 줄이면서도 맥락을 유지하는 데 유리하다.

실무에서 적용할 수 있는 기준은 다음 3가지이다.

1. 추상화 후에도 테스트 본문만 읽고 **전제와 기대 결과를 설명할 수 있는가**
2. 헬퍼가 “행동”까지 숨기지 않고, 준비/검증 중 어디를 담당하는지 **역할 경계가 분명한가**
3. 중복을 줄인 뒤에도 실패 로그를 보고 **원인 위치를 빠르게 특정할 수 있는가**

요약하면, 테스트에서 제거해야 할 것은 단순 반복이 아니라 “이 테스트가 왜 존재하는지 보이지 않게 만드는 코드”이다. 중복을 조금 남기더라도 의도가 선명하다면, 그 테스트는 장기적으로 더 잘 유지된다.

### 실패 메시지가 좋은 테스트가 유지보수성을 만든다

테스트는 “성공할 때”보다 “실패할 때” 더 큰 가치를 드러낸다. 특히 장애 대응이나 회귀 분석 상황에서는 테스트 코드 본문을 처음부터 다시 읽을 시간이 부족하기 때문에, **실패 메시지 자체가 바로 다음 행동을 결정할 수 있어야** 한다. 결국 유지보수성이 높은 테스트는 assertion의 개수보다, 실패했을 때 남기는 정보의 품질로 구분된다.

현재 회사에서 활용했던 좋은 예시를 소개해보고자 한다. 앞서 소개했던 `shouldPermissionDeniedWith`를 보면 같은 도메인형 assertion을 통해 오류 타입과 경로를 명시적으로 검증하는 방식을 보여준다. 이렇게 작성하면 권한 이슈가 발생했을 때 “왜 실패했는지”가 테스트 이름+실패 지점에서 바로 연결된다. 그리고 `assertSoftly`를 활용해 복합 객체의 필드를 한 번에 검증할 수 있는데 한 필드씩 흩어진 실패를 보는 대신, 어떤 속성들이 기대와 달랐는지 묶어서 확인할 수 있어 수정 범위를 빠르게 좁힐 수 있다.

```kotlin
testDeniedRoles(
    allowRoles = setOf(Constants.STORE_ADMIN, Constants.STORE_MANAGER),
) { label, token ->
    test("$label 권한으로 추천 매장 등록을 요청하면 PERMISSION_DENIED 오류 메시지를 반환한다") {
        // Given
        val input = RecordReferrerStoreInput(referrerStoreId = generateId())
        val variables = mapOf("input" to input)

        // When
        val actual =
            clientBuilder
                .token(token)
                .build()
                .executeQuery(mutation, variables)
                .errors

        // Then
        actual shouldPermissionDeniedWith listOf("recordReferrerStore")
    }
}
```

또한 `shouldThrow` 뒤에 메시지까지 확인하는 패턴도 유지보수에 도움이 된다. 예외 타입만 검증하면 동일 타입의 다른 실패 원인이 섞여도 통과할 수 있지만, 메시지(또는 에러 코드)까지 검증하면 정책 변경/회귀를 더 정확히 탐지할 수 있다.

```kotlin
test("취소 상태인 청구서를 만료 처리하면 오류를 던진다.") {
    // Given
    val bill = BillFactory().produce()
    bill.cancel()

    // Expect
    val actual =
        shouldThrow<IllegalStateException> {
            bill.expire()
        }
    actual.message shouldBe "청구서가 활성화 상태가 아닙니다."
}
```

반대로 아래와 같은 테스트는 실패 메시지 품질을 낮추기 쉽다.

- `actual shouldBe true`처럼 맥락이 없는 단일 불리언 검증
- 여러 의미를 한 assertion에 섞어 실패 원인 분리가 어려운 검증
- 도메인 오류인데도 일반 예외 타입만 체크하고 경로/코드를 확인하지 않는 검증

실무에서 바로 적용할 수 있는 체크리스트는 다음과 같다.

1. 실패 시 로그만 보고도 **어느 규칙이 깨졌는지** 알 수 있는가
2. 도메인/권한 오류는 가능한 한 **에러 타입 + 경로 + 코드**까지 드러내는가
3. 복합 결과는 `assertSoftly`/도메인 assertion으로 **차이점을 묶어** 보여주는가

좋은 테스트는 “맞다/틀리다”를 알려주는 데서 끝나지 않는다. **어디를 어떻게 고쳐야 하는지까지 안내하는 실패 메시지**를 남길 때, 테스트는 장기 운영 환경에서 진짜 유지보수 도구가 된다.

### 팀 규칙으로 추천하는 체크리스트

테스트 품질은 개인 역량보다 **팀의 합의된 기준**에서 더 크게 좌우된다. 특히 규모가 있는 코드베이스에서는 “잘 짠 테스트 한두 개”보다, 누구나 비슷한 품질로 작성할 수 있는 최소 규칙이 더 중요하다. 기준이 없으면 리뷰는 취향 싸움이 되고, 기준이 너무 많으면 작성 속도가 급격히 떨어진다. 그래서 아래처럼 **작지만 강한 체크리스트**를 팀 규칙으로 고정하는 것이 효과적이다.

#### 테스트 이름 규칙

- `[주체/권한] + [행동] + [기대 결과]`를 한 문장에 포함한다.
- 가능하면 “~하면 ~한다” 형태로 결과를 명시한다.
- 금지: `성공 케이스`, `예외 케이스`, `test1` 같은 포괄적 이름.

예) `점주 권한으로 견적서를 조회하면 견적서를 반환한다.`처럼 이름 자체가 요구사항 역할을 하게 만든다.

#### `Given-When-Then`(사고 순서) 규칙

- Given: 전제/입력/fixture/stub만 준비한다.
- When: 행동은 1개만 실행한다.
- Then: 핵심 결과를 먼저 검증하고, 부가 검증은 뒤에 둔다.
- 금지: Given에서 assert, When에서 다중 액션, Then에서 맥락 없는 boolean 검증.

이 규칙 하나만 지켜도 실패 지점을 찾는 시간이 크게 줄어든다.

#### 중복 제거 규칙

- 중복 제거의 목적은 라인 수 절감이 아니라 **의도 보존**이다.
- 공통화는 “반복되는 의도”만 묶고, 시나리오 핵심(입력/행동/기대)은 본문에 남긴다.
- 헬퍼 함수는 준비/실행/검증 중 어느 역할인지 함수명에서 드러나야 한다.

즉, 한 줄로 줄였는데 의도가 사라지면 그 리팩터링은 실패다.

#### 실패 메시지 품질 규칙

- 도메인/권한 검증은 가능한 한 에러 타입·경로·코드까지 검증한다.
- 복합 응답은 `assertSoftly` 또는 도메인 assertion으로 차이를 묶어 보여준다.
- 예외 테스트는 타입만 보지 말고 메시지(또는 에러 코드)까지 확인한다.

리뷰 기준도 “assertion이 몇 개인가”보다 “실패 시 다음 행동이 명확한가”로 맞춘다.

위 항목들은 테스트 코드 작성 및 리뷰를 빠르게 해준다. 모든 테스트를 완벽하게 만들기보다, **누가 작성해도 최소한의 신뢰도를 확보하는 기준**을 먼저 고정하는 것이 장기적으로 훨씬 강한 테스트 문화를 만든다.

## 어디까지 테스트할 것인가: 유지 가능한 검증 전략 세우기

앞 장에서 테스트를 읽기 쉽게 만드는 기준을 정리했다면, 이제는 조금 더 현실적인 질문으로 넘어가야 한다. **같은 개발 시간 안에서 어떤 테스트를 먼저 작성하고, 어떤 검증은 어느 레이어에서 담당하게 할 것인가**에 따라 팀의 속도와 안정성은 크게 달라지기 때문이다. 전략이 없는 테스트는 열심히 쌓여도 우선순위가 흔들리고, 반대로 전략이 있는 테스트는 개수가 많지 않아도 배포 리스크를 효과적으로 낮춘다. 

이 장에서는 “많이 작성하는 법”보다 “오래 유지되는 검증 구조를 선택하는 법”에 초점을 맞춰, 핵심 시나리오 우선순위 설정부터 레이어 분리, 그리고 운영 단계에서의 실행 기준까지 순서대로 살펴보겠다.

### 테스트 레이어를 먼저 나누면 의사결정이 쉬워진다

테스트 전략에서 가장 먼저 정해야 할 것은 “무엇을 검증할까”가 아니라 **“어느 레이어에서 검증할까”**이다. 같은 요구사항이라도 검증 위치를 잘못 선택하면 테스트가 느려지거나, 반대로 중요한 회귀를 놓치기 쉽다. 그래서 실무에서는 기능 단위보다 먼저 테스트 레이어를 구분해 두는 것이 의사결정을 크게 단순화한다.

현재 회사에서는 테스트 레이어를 세 가지 레이어로 구분하였다.

- **Unit 테스트**: 도메인 규칙/계산/분기처럼 빠른 피드백이 필요한 로직을 검증
- **Repository 테스트**: 쿼리 조건, 정렬, 필터링 등 저장소 계층의 정합성을 검증
- **Functional 테스트**: API/GraphQL 기준으로 권한, 요청-응답 계약, 주요 사용자 흐름을 검증

이렇게 나누면 테스트 작성 시 판단 기준이 분명해진다. 예를 들어 “가격 계산 규칙이 바뀌었다”면 Unit 테스트를 먼저 보강하고, “조회 조건이 달라졌다”면 Repository 테스트를 우선 수정하며, “권한 정책이 추가됐다”면 Functional 테스트를 중심으로 회귀를 막는 식이다. 즉, 실패 원인에 따라 어디를 수정해야 하는지가 곧바로 연결된다.

핵심은 레이어 간 역할을 겹치지 않게 유지하는 것이다. Unit 테스트에서 DB 동작까지 검증하려 하거나, Functional 테스트 하나에 세부 계산 규칙까지 모두 넣기 시작하면 테스트는 느려지고 의도는 흐려진다. 반대로 각 레이어가 자기 책임만 정확히 검증하면, 테스트 개수가 많지 않아도 중요한 변경에서 안정성을 확보할 수 있다.

실무에서 바로 적용할 최소 규칙은 다음 세 가지이다.

1. **도메인 규칙은 Unit에서 먼저 수행하고**, 통합 테스트에서는 결과 연결만 확인한다.
2. **데이터 조회/저장의 정확성은 Repository에서 책임지고**, 서비스 테스트는 비즈니스 의사결정에 집중한다.
3. **권한/계약/주요 흐름은 Functional에서 최종 보증**하되, 세부 분기까지 중복 검증하지 않는다.

레이어를 먼저 정리해두면 테스트 작성 속도뿐 아니라 리뷰 품질도 올라간다. “왜 이 테스트를 여기서 검증했는가”가 설명 가능해지기 때문에, 테스트 추가/수정에 대한 팀 합의가 훨씬 빨라진다.

### 핵심 시나리오부터 잠그는 우선순위 전략

테스트 전략이 흔들리는 가장 큰 이유는 “가능한 케이스를 전부 다루려는 욕심”이다. 물론 이상적으로는 모든 분기를 검증하면 좋지만, 실제 프로젝트에서는 시간과 리뷰 비용이 항상 제한된다. 그래서 중요한 것은 테스트를 많이 작성하는 순서가 아니라, **실패 비용이 큰 시나리오를 먼저 잠그는 우선순위**를 정하는 일이다.

우선순위를 정할 때 실무에서 가장 유용한 기준은 아래와 같다.

- **장애 파급력**: 실패했을 때 매출/운영/고객 경험에 바로 영향을 주는가
- **변경 빈도**: 최근 수정이 잦아 회귀 위험이 큰 영역인가
- **정책 민감도**: 권한/정산/가격/계약처럼 “틀리면 안 되는 규칙”인가

이 기준으로 보면 같은 기능 안에서도 테스트 순서가 달라진다. 예를 들어 목록 조회 기능을 개선할 때도, 먼저 잠가야 하는 것은 “정렬 옵션 추가”보다 **권한 누락으로 데이터가 노출되는 시나리오**일 수 있다. 결제/정산처럼 정책 민감도가 높은 영역이라면, 정상 흐름보다 먼저 “잘못된 입력/경계값/예외 처리”를 고정하는 편이 회귀 예방에 더 효과적이다.

실행 방법은 3단계로 해볼 수 있다.

1. 이번 변경에서 깨지면 가장 치명적인 흐름 1~2개를 먼저 정의한다.
2. 그 흐름을 레이어별(Unit/Repository/Functional)로 나눠 최소 보호막을 만든다.
3. 남은 시간에는 부가 케이스를 확장하되, 우선순위 기준을 벗어나지 않는다.

여기서 효율을 높이는 핵심은 **테스트 상세도도 레이어별로 다르게 가져가는 것**이다. 세세한 분기·경계값·예외 조합은 실행 비용이 낮고 원인 추적이 빠른 Unit 테스트에 집중하고, Functional 테스트에서는 사용자 관점의 주요 플로우(성공 흐름, 권한 실패, 핵심 정책 위반)만 선별해 검증 범위를 관리하는 방식이 효과적이다. 이렇게 역할을 나누면 같은 검증을 여러 레이어에서 중복 작성하지 않아도 되고, 테스트 시간은 줄이면서도 릴리스 안정성은 유지할 수 있다.

이 접근의 장점은 명확하다. 테스트 수가 적어도 핵심 위험을 먼저 제어할 수 있고, PR 리뷰에서도 “왜 이 케이스를 먼저 작성했는지”가 설명된다. 결국 좋은 전략은 완전함보다 **리스크 대비 효과가 높은 순서**를 선택하는 데서 시작된다.

### 권한·외부연동·시간 의존성은 별도 전략으로 다룬다

테스트가 불안정해지는 지점은 대개 비즈니스 로직 자체보다 **권한, 외부 시스템, 시간값** 같은 의존성에서 시작된다. 이 세 가지는 환경 영향이 크고 변경 시 파급 범위가 넓기 때문에, 일반 시나리오와 같은 방식으로 다루면 테스트가 쉽게 flaky해지거나 디버깅 비용이 급격히 커진다. 그래서 전략 단계에서부터 별도 규칙으로 분리해 관리하면 좋다.

권한 테스트의 핵심은 “모든 역할을 다 테스트”가 아니라, **허용/거부 경계가 깨지지 않는지**를 빠르게 확인하는 것이다. 주요 API나 GraphQL 엔드포인트마다 정상 권한 1~2개와 거부되어야 할 대표 권한군을 고정해두면, 정책 변경 시 회귀를 조기에 탐지할 수 있다. 이때 세부 비즈니스 결과까지 한 테스트에 과도하게 섞지 말고, 권한 검증 자체를 독립된 실패 신호로 유지하는 것이 좋다.

```kotlin
// 샘플링을 이용한 권한 테스트
test("매장 권한으로 견적 이미지 업로드 url을 조회하면 PERMISSION_DENIED 오류를 반환한다.") {
    // Given
    val token = testHelper.generateStoreToken()

    val query =
        """
        query quotationImageUploadUrls(${"$"}fileNames: [String!]!) {
            quotationImageUploadUrls(fileNames: ${"$"}fileNames) {
            uploadUrls {
                uploadUrl
                objectKey
            }
            }
        }
        """.trimIndent()
    val fileNames = listOf("test1.jpg", "test2.jpg")
    val variables = mapOf("fileNames" to fileNames)

    // When
    val actual =
        clientBuilder.token(token).build()
            .executeQuery(query, variables)
            .errors

    // Then
    actual shouldPermissionDeniedWith listOf("quotationImageUploadUrls")
}


// 유틸리티를 이용한 모든 권한 검증
testDeniedRoles(
    allowRoles = setOf(Constants.STORE_ADMIN, Constants.STORE_MANAGER),
) { label, token ->
    test("$label 권한으로 견적 이미지 업로드 url을 조회하면 PERMISSION_DENIED 오류를 반환한다.") {
        val variables = mapOf("fileNames" to listOf("test1.jpg", "test2.jpg"))

        val actual =
            clientBuilder.token(token).build()
                .executeQuery(query, variables)
                .errors

        actual shouldPermissionDeniedWith listOf("quotationImageUploadUrls")
    }
}
```

외부 연동 테스트는 실제 네트워크 안정성에 기대기보다, **응답 계약과 예외 상황을 통제 가능한 형태로 고정**하는 데 집중해야 한다. 성공 응답뿐 아니라 타임아웃, 오류 코드, 예상치 못한 응답 구조를 시나리오로 만들어 두면, 장애 상황에서도 서비스의 보호 로직이 의도대로 동작하는지 확인할 수 있다. 중요한 것은 “외부 시스템이 정상일 때”보다 “외부 시스템이 비정상일 때도 우리 서비스가 안전한가”를 검증하는 관점이다.

아래 코드를 보면 테스트 시작 전에 외부 API 응답을 고정하고(`mockery.leaveSendbirdGroupChannel()`), 실행 후에는 실제 요청 계약을 검증하는 것을 볼 수 있다(`mockery.verifyLeaveSendbirdGroupChannel(...)`).

```kotlin
context("clearMatchingQuotationInquiryChannels") {
    test("마지막 채팅이 30일 이상 된 매칭 견적 문의 채널들이 정리된다.") {
        // Given
        // ... 생략 ...
        mockery.leaveSendbirdGroupChannel()

        // When
        matchingQuotationInquiryChannelClearScheduler.clearMatchingQuotationInquiryChannels()

        // Then
        // ... 생략 ...
        mockery.verifyLeaveSendbirdGroupChannel(
            channelUrl = quotationRequestedVendor.matchingQuotationInquiryChannelUrl!!,
            sendbirdUserIds = store.managersSendbirdIds + listOf(vendorAccount.sendbirdId!!),
        )
    }
}
```

또한 Mockery 확장 함수에서 HTTP 메서드/경로/본문을 명시적으로 검증하기 때문에, 외부 API 스펙이 깨졌을 때 테스트가 즉시 실패 신호를 준다.

```kotlin
fun Mockery.verifyLeaveSendbirdGroupChannel(channelUrl: String, sendbirdUserIds: List<String>) {
    sendbirdApi.verifyWithDetailedError(
        HttpRequest.request()
            .withMethod(HttpMethod.PUT.name())
            .withPath("/v3/group_channels/$channelUrl/leave")
            .withBody(json(body)),
    )
}
```

아래 코드는 실패응답에 대한 예외처리를 검증하는 코드이다.

```kotlin
test("API 응답이 실패하면 예외를 던진다.") {
    // Given
    every {
        nicepayDataApi.getReconciliations(any())
    } returns
        NicepayDataPayload(
            header =
                NicepayDataPayloadHeader(
                    sid = NicepayDataSid.TRANSACTION,
                    trDtm = LocalDateTime.now(),
                    gubun = NicepayPayloadType.R,
                    resCode = "1392",
                    resMsg = "등록되지 않은 ip로 접근하였습니다",
                ),
            body = NicepayTransactionReconciliationResponse(),
        )

    val request =
        NicepayReconciliationRequest(
            transactionDate = LocalDate.of(2023, 1, 1),
            sid = NicepayDataSid.TRANSACTION,
            paymentMethod = NicepayPaymentMethod.VIRTUAL_ACCOUNT,
        )

    // Expect
    val actual =
        shouldThrow<NicepayClientException> {
            nicepayClient.getReconciliations(request)
        }
    actual.message shouldBe "등록되지 않은 ip로 접근하였습니다"
    actual.code shouldBe "1392"
}
```

시간 의존성은 특히 재현성을 해치기 쉬운 영역이다. `now()` 같은 현재 시각 참조가 테스트 곳곳에 직접 등장하면 실행 시점마다 결과가 달라지고, 야간 배치/타임존/월말 경계에서만 깨지는 테스트가 생긴다. 따라서 시간은 가능한 한 주입 가능한 값(고정 시각/Clock)으로 다루고, 경계 케이스(자정 직전·직후, 월말, 타임존 변환)를 명시적으로 잠가두는 편이 좋다.

```kotlin
test("수정일시로 미수금 동기화가 필요한 연관관계를 조회하고 미수금을 동기화한다") {
    // Given
    val updatedTimeGte = OffsetDateTime.now()

    // 생략...

    // When
    accountReceivableService.syncByUpdatedTime(updatedTimeGte = updatedTimeGte)

    // Then
    assertSoftly(expected) {
        it.accountReceivable shouldBe 20_000.toBigDecimal()
        it.accountReceivableLimitUsageRate shouldBe 133.3.toBigDecimal()
        it.limitExceededAmount shouldBe 50_000.toBigDecimal()
    }
}
```

상황에 따라서는 시간값을 주입하기 힘든 경우도 있다. 이런경우 아래와 같이 Mocking 도구를 이용하여 시간을 고정한다음 테스트하는 방법도 있다.

```kotlin
test("인증 토큰이 만료기간이 지나면 오류를 반환한다.") {
    // Given
    val expired = Instant.now().minusMillis(32400000)
    val token =
        withConstantNow(expired) {
            sut.generateToken(principal)
        }

    // Expected
    val actual =
        assertThrows<BadCredentialsException> {
            sut.getPrincipal(token)
        }
    actual.message shouldBe "인증 토큰이 만료되었습니다."
}
```

정리하면, 권한·외부연동·시간 의존성은 “부가 케이스”가 아니라 **전략적으로 먼저 통제해야 하는 리스크 축**이다. 이 축을 별도 규칙으로 분리하면 테스트의 신뢰도와 실행 효율을 함께 확보할 수 있고, 운영 이슈가 발생했을 때도 원인 추적 속도가 눈에 띄게 빨라진다.

### 테스트 데이터 설계가 전략의 절반이다

테스트 전략을 이야기할 때 자주 놓치는 부분이 데이터이다. 하지만 실제로는 “무엇을 검증할지” 못지않게 **어떤 데이터로 검증할지**가 결과 품질을 좌우한다. 데이터 준비가 어렵거나 읽기 힘들면 테스트 추가 자체가 느려지고, 반대로 데이터 설계가 잘 되어 있으면 같은 시간에 더 많은 회귀 위험을 잠글 수 있다. 그래서 테스트 데이터 설계는 구현 편의가 아니라 전략의 핵심으로 다뤄야 한다.

좋은 테스트 데이터의 첫 조건은 재사용성보다 **의도 가시성**이다. 예를 들어 주문 합계 계산을 검증하는 테스트라면, 필요한 최소 필드만 노출하고 나머지는 기본값으로 숨겨야 시나리오가 한눈에 들어온다. 반대로 모든 필드를 매번 나열하거나, 너무 많은 기본값이 섞인 거대한 fixture를 쓰면 “왜 이 값이 중요한지”를 파악하기 어려워진다.

아래와 같이 복잡한 이벤트 payload는 기본 fixture를 재사용하되, 시나리오 핵심 필드만 `copy`로 드러내 의도를 유지하는 것을 볼 수 있다.

```kotlin
val orderSheetSubmittedData =
    fixture<OrderSheetSubmittedData>().copy(
        orderSheetId = 1L,
        orderSheetNumber = "orderSheetNumber",
        feeType = FeeType.PERCENT,
    )
```

이때 효과적인 방식이 Factory 기반 데이터 구성이다. 공통 기본 객체는 Factory에서 만들고, 시나리오마다 필요한 속성만 override하는 방식으로 테스트를 작성하면 중복은 줄이면서도 의도는 선명하게 유지할 수 있다. 특히 권한/상태/금액처럼 자주 바뀌는 속성을 메서드나 trait 형태로 조합 가능하게 두면, 케이스 확장 속도가 눈에 띄게 빨라진다.

```kotlin
test("active 상태의 프로모션 배너 목록을 조회한다") {
    // Given
    val inactivePromotionBanner = PromotionBannerFactory().withTrait(InactivateTrait).produce()
    val activePromotionBanner = PromotionBannerFactory().withTrait(ActivateTrait).produce()
    val expiredPromotionBanner = PromotionBannerFactory().withTrait(ExpiredTrait).produce()

    testEntityManager.persist(inactivePromotionBanner)
    testEntityManager.persist(activePromotionBanner)
    testEntityManager.persist(expiredPromotionBanner)

    val pageRequest = OffsetPageRequest(limit = 10)

    // When
    val actual =
        promotionBannerRepository.findByFilter(
            PromotionBannerStateFilter(listOf(PromotionBannerState.ACTIVE)),
            pageRequest,
        )

    // Then
    actual.content shouldContainExactlyInAnyOrder listOf(activePromotionBanner)
}
```

또 하나 중요한 기준은 “레이어별 데이터 복잡도”를 다르게 가져가는 것이다. Unit 테스트에서는 경계값과 예외 조합을 빠르게 실험할 수 있도록 작고 명시적인 데이터를 쓰고, Functional 테스트에서는 사용자 플로우를 설명하는 최소 시나리오 중심으로 데이터 세트를 줄이는 편이 효율적이다. 이렇게 하면 데이터 준비 비용이 테스트 범위를 압도하는 상황을 피할 수 있다.

실무에서 바로 적용할 체크리스트는 다음과 같다.

1. 테스트 데이터가 시나리오 의도(권한/상태/조건)를 이름과 값으로 드러내는가
2. 공통 fixture는 Factory로 관리하되, 핵심 입력값은 테스트 본문에서 보이게 남겨두는가
3. 레이어별로 데이터 크기와 상세도를 분리해 불필요한 준비 비용을 줄였는가

결국 테스트 데이터 설계가 잘 된 팀은 테스트를 “작성해야 하는 일”이 아니라 “빠르게 확장 가능한 자산”으로 다룰 수 있다. 전략의 완성도는 테스트 개수보다, 필요한 시나리오를 얼마나 낮은 비용으로 정확히 추가할 수 있는지에서 결정된다.

### CI에서 실행 순서와 범위를 분리해 속도와 신뢰도를 함께 잡는다

테스트 전략이 문서에서만 끝나지 않으려면, 결국 CI에서 어떻게 실행되는지가 중요하다. 같은 테스트 세트라도 **언제, 어떤 범위로 실행하느냐**에 따라 개발 체감 속도와 배포 안정성은 크게 달라진다. 모든 테스트를 매번 동일하게 돌리면 안정성은 높아 보이지만 피드백이 늦어지고, 반대로 너무 가볍게만 돌리면 릴리스 직전에 리스크가 높아진다. 그래서 CI는 실행 순서와 범위를 의도적으로 분리해 운영해야 한다.

실무에서는 보통 두 단계로 나누는 방식이 효과적이다. PR 단계에서는 빠른 피드백이 가능한 Unit/핵심 Repository 테스트를 우선 실행해, 개발자가 수정 직후 회귀를 빠르게 확인할 수 있게 한다. 반면 main 머지 전후나 배포 파이프라인에서는 Functional 테스트와 외부 연동/권한/시간 경계 시나리오까지 포함해, 실제 운영 리스크를 집중 점검한다.

아래 코드는 Gralde에서 Functional 테스트와 그외 테스트를 나누어서 실행할 수 있도록 설정한 코드이다.

```kotlin
// build.gralde.kts
tasks.register("functionalTest", Test::class) {
    filter {
        include("com/**/**/application/presentation/**")
        include("com/**/**/application/scheduler/**")
        include("com/**/**/ApplicationTest**")
    }
}

tasks.register("testWithoutFunctionalTest", Test::class) {
    filter {
        include("com/**")
        exclude("com/**/**/application/presentation/**")
        exclude("com/**/**/application/scheduler/**")
        exclude("com/**/**/ApplicationTest**")
    }
}
```

이 분리 전략의 핵심은 “중요도”와 “실행 비용”을 함께 고려하는 것이다. 예를 들어 정책 민감도가 높은 테스트는 실행 시간이 조금 길어도 배포 게이트에 반드시 포함하고, 변경 영향이 작고 계산 중심인 테스트는 PR 단계에서 높은 빈도로 돌려 빠른 경고 신호를 받는 식이다. 이렇게 설계하면 팀은 느린 전체 실행을 기다리지 않고도 일상 개발 속도를 유지하면서, 최종 단계에서는 필요한 신뢰도를 확보할 수 있다.

물론 처음부터 효율성과 속도를 따지면서 신뢰도를 낮추기보다 CI 환경에서 전체 테스트 검증에 소요되는 시간을 줄이는 노력을 선행하는 것이 좋다. 캐싱, 메모리 최적화, 증분 컴파일, 병렬 실행 등은 테스트 실행 속도를 높일 수 있는 좋은 수단이다.

```kotlin
// build.gradle.kts
tasks.withType<Test> {
    // 병렬 실행
    maxParallelForks = Runtime.getRuntime().availableProcessors() / 2

    // 메모리 설정
    jvmArgs(
        "--add-opens=java.base/java.time=ALL-UNNAMED",
        "-Xshare:off",
        "-XX:+UseG1GC",
        "-XX:MaxGCPauseMillis=200",
        "-Xmx5g",
    )

    // 생략...
}
```

```
// gralde.properties
org.gradle.caching=true
org.gradle.parallel=true
org.gradle.jvmargs=-Xmx8g
kotlin.daemon.jvmargs=-Xmx8g
kotlin.incremental.useClasspathSnapshot=true
# 생략...
```

```
// settings.gradle.kts
// 리모트 캐시를 설정하는 경우
plugins {
    id("com.github.burrunan.s3-build-cache") version "1.8.1"
}

buildCache {
    local {
        isEnabled = false
    }

    remote<com.github.burrunan.s3cache.AwsS3BuildCache> {
        lookupDefaultAwsCredentials = true
        region = "ap-northeast-2"
        bucket = "gradle-build-cache"
        prefix = "cache/"
    }
}
```

결국 CI 전략의 목적은 테스트를 더 많이 돌리는 것이 아니라, **적절한 시점에 적절한 테스트를 돌려 의사결정을 빠르게 만드는 것**이다. 실행 순서와 범위를 분리하면 개발 속도와 배포 신뢰도를 동시에 지킬 수 있고, 테스트가 팀 생산성을 떨어뜨리는 병목이 아니라 안정적인 가속 장치로 기능하게 된다.

### 전략이 팀에 정착되었는지 측정하는 운영 지표

테스트 전략은 문서로 선언하는 순간보다, 팀의 일상 작업에서 반복될 때 비로소 정착된다. 그래서 마지막에 꼭 필요한 질문은 “좋은 전략을 세웠는가”가 아니라 **“지금 전략이 실제로 작동하고 있는가”**이다. 이를 확인하려면 테스트 개수나 커버리지 수치만 보는 것에서 벗어나, 개발·리뷰·배포 흐름에서 체감 가능한 운영 지표를 함께 봐야 한다.

가장 먼저 볼 지표는 **피드백 속도**이다. PR을 올린 뒤 테스트 결과가 돌아오기까지 너무 오래 걸리면, 개발자는 테스트를 안전망보다 대기 시간으로 인식하게 된다. 따라서 PR 단계 평균 소요 시간, 재실행 비율, 실패 후 수정 완료까지의 리드타임을 함께 관찰하면 “전략이 속도를 지키고 있는지”를 비교적 정확히 판단할 수 있다. CI 파이프라인의 실행시간은 보통 이상적으로는 10분 내외가 권장되기는 하지만 팀내에 적정한 가드레일을 세워두고 실행 시간이 넘어가는 경우 시간을 단축시키기 위한 지속적인 관리를 해주면 좋겠다.

두 번째는 **신뢰도 지표**이다. 대표적으로 main 병합 후 테스트 실패율, 배포 직전 실패율, flaky 테스트 비율을 추적하면 좋다. 특히 flaky 비율이 높아지면 팀은 실패를 신호가 아니라 노이즈로 받아들이기 시작하고, 결국 진짜 회귀까지 놓치게 된다. 이때는 테스트를 더 추가하기보다 불안정 케이스를 먼저 정리하는 것이 전략적으로 더 큰 효과를 낸다. CircleCI와 같은 CI 도구에서는 [Flaky Test Detection](https://circleci.com/blog/introducing-test-insights-with-flaky-test-detection/)과 같은 도구를 제공해주니 이를 활용하면 좋다.

세 번째는 **회귀 예방 효과**이다. 장애가 발생했을 때 포스트모템 등을 통해 “이미 테스트가 있었는지”, “있었다면 왜 막지 못했는지”, “사후에 어떤 테스트가 추가되었는지”를 기록하면 전략의 빈틈이 드러난다. 이 기록이 누적되면 특정 레이어(Unit/Repository/Functional) 중 어디가 반복적으로 비어 있는지 파악할 수 있고, 다음 스프린트의 보강 우선순위를 데이터 기반으로 정할 수 있다.

실무에서 자주 쓰는 최소 운영 지표를 정리하자면 아래와 같다.

1. PR 테스트 피드백 평균 시간
2. main 병합 후 테스트 실패율
3. flaky 테스트 비율
4. 장애 사후 테스트 보강 리드타임
5. 레이어별 테스트 공백(회귀 발생 지점 기준)

핵심은 지표를 “평가용 숫자”로 쓰지 않는 것이다. 지표는 팀을 압박하기 위한 성과판이 아니라, 전략의 마찰 지점을 조기에 발견해 개선 우선순위를 맞추기 위한 나침반에 가깝다. 이 관점을 유지하면 테스트 전략은 일회성 캠페인이 아니라, 팀의 개발 시스템 안에서 꾸준히 진화하는 운영 습관으로 자리 잡게 된다.

## 테스트를 빨리 쓰게 만드는 공통 도구의 힘

전략이 정리되었다면, 이제 남는 과제는 실행 속도이다. 테스트가 중요하다는 데 모두 동의해도 실제 개발 일정 안에서 꾸준히 작성되지 않으면 전략은 금방 문서에만 남게 된다. 이때 팀의 속도를 바꾸는 것은 개인의 의지보다 **반복 작업을 줄여주는 공통 도구**이다. 잘 설계된 유틸리티는 테스트를 “추가 업무”가 아니라 “기본 개발 흐름”으로 바꿔주고, 작성 속도와 리뷰 품질을 동시에 끌어올린다. 이 장에서는 assertion, fixture, 권한 검증, 응답 파싱처럼 자주 반복되는 작업을 어떻게 표준화해야 팀 전체 생산성을 안정적으로 높일 수 있는지 살펴보려고 한다.

### 도메인 assertion 유틸: 실패 메시지를 ‘디버깅 가이드’로 바꾸기

테스트가 실패했을 때 가장 먼저 필요한 것은 “다시 읽기”가 아니라 “바로 다음 액션”아다. 도메인 assertion 유틸은 이 지점을 크게 개선한다. 단순히 `true/false`를 확인하는 대신, **도메인 규칙(권한, 에러 경로, 에러 타입, 에러 코드)**을 한 번에 검증하도록 캡슐화하면 실패 메시지 자체가 디버깅 가이드 역할을 하게 된다.

```kotlin
actual shouldPermissionDeniedWith listOf("quotationImageUploadUrls")

private fun List<GraphQLError>.assertGraphqlError(
    message: String,
    path: List<Any>,
    errorType: ErrorType,
) {
    this.shouldNotBeEmpty()
    assertSoftly(this.first()) {
        this.message shouldBe message
        this.path shouldContainExactly path
        this.extensions?.errorType shouldBe errorType
        this.extensions?.debugInfo?.variables?.get("errorCode") shouldBe ErrorMessage.findErrorCode(message)
    }
}
```

덕분에 테스트가 깨졌을 때도 “권한 실패 자체가 틀린 건지”, “에러 path가 잘못 연결된 건지”, “에러 코드 매핑이 어긋난 건지”를 로그에서 바로 구분할 수 있다.

또한 복합 응답 검증에서는 `assertSoftly`를 사용해 관련 필드 차이를 한 번에 보여주므로, 한 필드씩 순차 수정하는 비효율을 줄일 수 있다.

```kotlin
assertSoftly(actual) {
    it.id shouldBe quotation.id
    it.status shouldBe QuotationStatusEnum.REQUESTED
    it.store.id shouldBe store.id
    it.store.name shouldBe store.name
}
```

실무에서 적용할 때의 핵심은 아래와 같다.

1. 자주 반복되는 검증(권한/에러/시간/주소 등)은 도메인 assertion으로 승격한다.
2. assertion 이름에 의도를 담아, 테스트 본문이 “무엇을 검증하는지”를 문장처럼 읽히게 한다.
3. 실패 시 원인 분리가 가능하도록(메시지/경로/타입/코드) 충분한 정보를 함께 검증한다.

이 원칙을 지키면 테스트는 통과 여부만 알려주는 스크립트를 넘어, 장애 대응 속도를 높이는 운영 도구로 변화할 수 있다.

### Factory/Fixture 유틸: 테스트 데이터 준비 시간을 줄이는 구조

테스트 작성 시간을 가장 많이 잡아먹는 구간은 검증 로직보다 데이터 준비이다. 특히 도메인이 복잡해질수록 “테스트하려는 조건”보다 “객체를 만들기 위한 필드 채우기”가 더 길어지기 쉽다. 이때 Factory/Fixture 유틸을 잘 사용하면, 준비 코드를 줄이면서도 테스트 의도를 더 또렷하게 드러낼 수 있다.

현재 회사에서는 테스트를 수행할 때 **기본 데이터는 Factory/fixture로 생성하고, 시나리오 핵심 값만 override**하는 방식을 사용한다.

```kotlin
val event =
    OrderSheetSubmittedEvent(
        submittedData =
            WithUserData(
                userId = generateId(),
                userType = UserType.STORE,
                data = fixture<OrderSheetSubmittedData>().copy(interpretationId = history.id),
            ),
    )
```

이 패턴의 장점은 복잡한 `OrderSheetSubmittedData` 전체를 매번 조립하지 않아도 되고, 이 테스트가 실제로 확인하려는 조건인 `interpretationId`만 눈에 띄게 드러나도록 한다는 것이다.

Factory를 활용한 조합형 데이터 준비도 효과적이다.

```kotlin
val manager = ManagerFactory(phoneNumber = { null }).produce()
val store = StoreFactory(managers = { listOf(manager) }).produce()
```

여기서는 “전화번호 없는 매니저가 포함된 매장”이라는 시나리오가 fixture 조합만으로 간결하게 표현된다. 즉, 필요한 속성만 바꿔 테스트 목적을 코드에서 바로 읽을 수 있게 만든다.

시간 의존성이 있는 데이터도 Factory와 유틸을 함께 쓰면 안정적으로 고정할 수 있다.

```kotlin
val orderSheet =
    withConstantNow(TestOffsetDateTime.of(2022, 11, 11, 11, 11, 11)) {
        OrderSheetFactory(
            storeId = { storeId },
            vendorId = { vendorId },
            replaceOrderSheetProductDataList = {
                listOf(
                    ReplaceOrderSheetProductData.of(vendorProducts[0], (2).toDouble()),
                    ReplaceOrderSheetProductData.of(vendorProducts[1], (3).toDouble()),
                )
            },
        ).produce()
    }
```

이렇게 하면 테스트 데이터 생성과 시간 고정을 함께 다뤄 flaky 위험을 낮추면서, 시나리오별 핵심 입력값도 명확히 유지할 수 있다.

실무 적용 체크포인트는 다음 3가지이다.

1. 기본 데이터는 Factory/fixture로 만들고, 본문에는 시나리오 핵심 값만 남긴다.
2. Factory 인자 이름(`phoneNumber`, `managers` 등)만 읽어도 테스트 의도가 보이게 설계한다.
3. 시간/외부 상태 의존값은 생성 단계에서 함께 고정해 재현성을 확보한다.

이 원칙을 지키면 테스트 데이터 준비는 병목이 아니라, 빠른 실험과 안전한 회귀 검증을 가능하게 하는 기반이 된다.

### 권한 테스트 헬퍼: 허용/거부 검증을 표준화하는 방법

권한 테스트는 중요하지만, 엔드포인트마다 같은 패턴을 반복하다 보면 금방 길어지고 누락도 생긴다. 특히 “허용 권한은 누구인지”, “거부 권한이 모두 막히는지”를 매번 수작업으로 작성하면 테스트 품질이 사람마다 달라지기 쉽다. 이때 권한 테스트 헬퍼를 사용하면 검증 범위와 작성 방식이 동시에 표준화된다.

우리는 `FunctionalTestBase`에 `testDeniedRoles` 헬퍼를 두고, 허용 권한 집합만 선언하면 나머지 권한(및 무토큰 케이스)을 일괄적으로 거부 검증하도록 구성해두었다.

```kotlin
protected suspend fun FunSpecContainerScope.testDeniedRoles(
    allowRoles: Set<String>,
    testFn: suspend (label: String?, token: String?) -> Unit,
) {
    val roles = setOf(
        Constants.STORE_ADMIN,
        Constants.STORE_MANAGER,
        Constants.OWNER,
        Constants.AGENT,
        Constants.SALES_MANAGER,
        Constants.VENDOR,
        Constants.PARTNER,
        Constants.FINANCE,
        Constants.PAYMENT_OPERATOR,
        null,
    )
    val notAllowedRoles = roles.filterNot { allowRoles.contains(it) }

    context("권한 테스트") {
        forAll(*notAllowedRoles.map { row(it, generateToken(it)) }.toTypedArray()) { label, token ->
            testFn(label, token)
        }
    }
}
```

실사용 테스트에서는 허용 권한만 명시하고, 거부 검증은 공통 assertion으로 연결해 짧고 일관된 형태를 유지했다.

```kotlin
testDeniedRoles(
    allowRoles = setOf(Constants.STORE_ADMIN, Constants.STORE_MANAGER),
) { label, token ->
    test("$label 권한으로 견적 이미지 업로드 url을 조회하면 PERMISSION_DENIED 오류를 반환한다.") {
        val actual =
            clientBuilder.token(token).build()
                .executeQuery(query, variables)
                .errors

        actual shouldPermissionDeniedWith listOf("quotationImageUploadUrls")
    }
}
```

이 구조의 장점은 세 가지이다.

1. **누락 방지**: 신규 권한이 추가되어도 헬퍼 기준으로 거부 검증 범위를 쉽게 확장할 수 있다.
2. **가독성 개선**: 테스트 본문은 비즈니스 시나리오에 집중하고, 권한 반복 로직은 헬퍼로 숨길 수 있다.
3. **실패 일관성**: `shouldPermissionDeniedWith`를 함께 사용해 에러 경로/타입/코드 검증 방식까지 통일할 수 있다.

권한은 회귀 시 파급이 큰 영역이므로, 개별 테스트의 꼼꼼함보다 **팀 전체가 동일한 방식으로 빠짐없이 검증하는 구조**가 더 중요하다. 권한 테스트 헬퍼는 그 구조를 가장 낮은 비용으로 유지하게 해주는 도구이다.

### GraphQL/API 응답 파싱 유틸: 본문은 시나리오에만 집중하기

GraphQL/API 테스트가 길어지는 대표적인 이유는 파싱 코드가 본문을 잠식하기 때문이다. 요청-응답 검증 자체보다 `extractValue`/타입 변환/에러 처리 보일러플레이트가 더 많아지면, 테스트는 시나리오 문서가 아니라 파싱 구현처럼 보이게 된다. 이때 응답 파싱 유틸을 공통화하면 테스트 본문은 “무엇을 검증하는지”에 집중할 수 있다.

아래 코드에는 `GraphQLResponse.fetchAsObject(...)` 확장을 제공해, path와 타입만 전달하면 응답 파싱을 일관된 방식으로 처리하는 것을 볼 수 있다.

```kotlin
fun <T> GraphQLResponse.fetchAsObject(
    path: String,
    typeRef: TypeRef<T>,
): T {
    return runCatching {
        this.extractValueAsObject(path, typeRef)
    }.onFailure {
        logGraphQLErrors(path, typeRef.toString(), it)
    }.getOrThrow()
}
```

실사용 테스트에서는 `executeQuery(...).fetchAsObject(...)` 조합으로 핵심 흐름만 남기고, 파싱/오류 로깅 상세는 유틸에 위임한다.

```kotlin
val actual =
    clientBuilder.token(testHelper.점주_토큰_생성(manager.id)).build()
        .executeQuery(query)
        .fetchAsObject("terms", typeRef<List<TermsField>>())
```

권한 실패 케이스도 동일한 실행 경로를 유지한 채 `.errors`로 분기하므로, 성공/실패 시나리오 간 테스트 형태가 크게 달라지지 않는다.

```kotlin
val actual =
    clientBuilder.token(token).build()
        .executeQuery(query)
        .errors

actual shouldPermissionDeniedWith listOf("terms")
```

또한 파싱 실패 시에는 유틸 내부의 `logGraphQLErrors`가 path, errorType, debugInfo 등을 묶어 예외 메시지로 남기기 때문에, 단순한 `PathNotFound`보다 원인 파악이 훨씬 빠르다. 즉, 테스트 본문에서는 시나리오를 간결하게 유지하면서도, 실패 시 관측 가능성은 오히려 더 높일 수 있다.

실무 적용 체크포인트는 아래와 같다.

1. 응답 파싱은 공통 유틸로 모아 테스트 본문에서 반복 제거
2. `path + typeRef` 형태를 표준화해 테스트 간 파싱 스타일 일관성 유지
3. 파싱 실패 로그에 GraphQL 에러 컨텍스트를 포함해 디버깅 시간 단축

이렇게 정리하면 테스트 코드는 “데이터를 어떻게 꺼냈는지”보다 “무엇을 보장해야 하는지”를 먼저 보여주게 되고, 리뷰어와 작성자 모두가 시나리오 중심으로 대화할 수 있다.

### 시간/외부연동 테스트 헬퍼: flaky를 줄이는 고정점 만들기

flaky 테스트의 대부분은 비즈니스 로직보다 **시간(now)과 외부 API 상태**에서 발생한다. 같은 코드를 실행해도 시점이 달라지거나 외부 응답이 흔들리면 테스트 결과가 달라지기 때문이다. 그래서 시간/외부연동은 개별 테스트에서 즉흥적으로 처리하기보다, 헬퍼를 통해 “고정점”을 만드는 방식이 필요하다.

아래와 같이 시간 고정은 라이브러리의 도움을 받아 `withConstantNow(...)` 패턴으로 일관되게 다룰 수 있다.

```kotlin
val now = OffsetDateTime.now().minusDays(100)
val thirtyOneDaysAgo = now.minusDays(31)

withConstantNow(thirtyOneDaysAgo) {
    testHelper.매칭_요청_생성(storeId = store.id)
}

withConstantNow(now) {
    matchingQuotationInquiryChannelClearScheduler.clearMatchingQuotationInquiryChannels()
}
```

이렇게 시간 축을 명시적으로 고정하면, “30일 경과” 같은 경계 조건 테스트가 실행 시점과 무관하게 재현된다. 즉, 자정/월말/서머타임 같은 우발 요인으로 인한 비결정성을 크게 줄일 수 있다.

외부 연동은 헬퍼로 계약을 고정해 테스트한다. 예를 들어 우리는 Sendbird 연동은 사전에 성공 응답을 고정하고, 실행 후 요청 본문/경로를 검증하는 방식으로 안정성을 확보했다.

```kotlin
fun Mockery.leaveSendbirdGroupChannel() {
    sendbirdApi.whenWithDefault(
        requestDefinition =
            HttpRequest.request()
                .withMethod(HttpMethod.PUT.name())
                .withPath("/v3/group_channels/.+/leave"),
    ).respond(
        HttpResponse.response().withStatusCode(200),
    )
}

fun Mockery.verifyLeaveSendbirdGroupChannel(
    channelUrl: String,
    sendbirdUserIds: List<String>,
) {
    // ... 생략 ...
    sendbirdApi.verifyWithDetailedError(
        HttpRequest.request()
            .withMethod(HttpMethod.PUT.name())
            .withPath("/v3/group_channels/$channelUrl/leave")
            .withBody(json(body)),
    )
}
```

실제 테스트 본문은 아래처럼 짧게 유지되며, 외부 API 성공/검증 로직은 헬퍼에 위임된다.

```kotlin
mockery.leaveSendbirdGroupChannel()

withConstantNow(now) {
    matchingQuotationInquiryChannelClearScheduler.clearMatchingQuotationInquiryChannels()
}

mockery.verifyLeaveSendbirdGroupChannel(
    channelUrl = quotationRequestedVendor.matchingQuotationInquiryChannelUrl!!,
    sendbirdUserIds = store.managersSendbirdIds + listOf(vendorAccount.sendbirdId!!),
)
```

실무 적용 체크포인트는 세 가지이다.

1. 시간 경계가 있는 테스트는 반드시 `withConstantNow`로 고정해 실행 시점 의존성을 제거한다.
2. 외부 API는 “실패하지 않게”가 아니라 “요청 계약이 맞는지”를 헬퍼로 검증한다.
3. 본문은 시나리오에 집중하고, 시간/외부연동 보일러플레이트는 공통 헬퍼로 이동한다.

이 원칙을 지키면 flaky를 줄이는 동시에, 실패 원인도 “시간 계산 문제인지/외부 요청 계약 문제인지”로 빠르게 분리할 수 있어 운영 대응 속도까지 함께 개선된다.

### 테스트 공통 베이스/확장 함수: 팀 규칙을 코드로 강제하기

테스트 품질을 문서로만 합의하면 시간이 지나며 쉽게 흐려진다. 반대로 공통 베이스 클래스와 확장 함수로 규칙을 코드에 녹여두면, 새 테스트를 작성할 때도 팀 규칙이 기본값으로 적용된다. 즉, “리뷰에서 잡는 방식”보다 “애초에 그렇게 작성되게 만드는 방식”이 장기적으로 더 강하다.

우리는 아래와 같이 테스트 타입별 베이스를 분리해 기본 행동을 강제하고 있다. 특히 베이스에 공통 설정을 모아두면, 개별 테스트는 같은 어노테이션/확장 설정을 반복하지 않아도 팀 규칙이 자동 적용된다.

```kotlin
@Import(FunctionalTestConfig::class, TestDatabaseConfiguration::class, TestRedisConfiguration::class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ContextConfiguration(initializers = [MockServerPropertyOverrideContextInitializer::class])
abstract class FunctionalTestBase : FunSpec() {
    override fun extensions() = listOf(SpringExtension, MockeryExtension)
}
```

이렇게 해두면 Functional 테스트 클래스는 매번 Spring/Mockery 설정을 다시 선언할 필요 없이 `: FunctionalTestBase()` 상속만으로 동일한 실행 규칙을 따르게 된다. 또한 데이터베이스 설정이나 여러 외부 리소스 연동을 하나하나 해줄 필요없이 단순히 베이크 클래스 상속만으로 해결할 수 있기 때문에 중복코드를 상당히 줄일 수 있다.

Unit 테스트에서도 같은 방식으로 공통 규칙을 강제하는 것을 볼 수 있다.

```kotlin
abstract class UnitTestBase : FunSpec() {
    override suspend fun beforeEach(testCase: TestCase) {
        clearAllMocks()
    }
}
```

Unit 테스트는 `UnitTestBase`를 상속하는 순간 매 테스트마다 mock 상태가 초기화된다. 그래서 테스트 간 상태 누수로 인한 간헐 실패를 기본적으로 차단할 수 있다.

```kotlin
class AIChatHistoryEventHandlerTest : UnitTestBase() {
    // ...
}
```

Repository 테스트도 아래와 같이 통합테스트를 위한 설정과 유틸리티 함수를 제공하여 테스트 규칙을 강제하고 중복코드를 줄이는 것을 볼 수 있다.

```kotlin
@DataJpaTest(showSql = false)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@AutoConfigureJdbc
@Import(
    TestDatabaseConfiguration::class,
    KotlinJdslAutoConfiguration::class,
    JdslExecutorSupport::class,
    JdslConfig::class,
    AdvisoryLockRepository::class,
)
abstract class RepositoryTestBase : FunSpec() {
    override fun extensions() = listOf(SpringExtension)
}

fun TestEntityManager.persistAll(vararg entities: Any): List<Any> =
    entities.map {
        when (it) {
            is Iterable<*> -> it.filterNotNull().map { element -> persistAll(element) }
            else -> persist(it)
        }
    }
```

결국 공통 베이스/확장 함수의 목적은 “코드 줄이기” 자체가 아니라, **팀 규칙(초기화, 권한 검증, 데이터 준비, 실패 관측성)을 테스트 작성 시점에 자동으로 적용**하는 데 있다. 이 구조를 갖추면 리뷰는 스타일 교정이 아니라 시나리오 완성도 검토에 더 많은 시간을 쓸 수 있다.

### 유틸리티 도입 체크리스트: 추상화가 의도를 숨기지 않게 하기

유틸리티는 잘 쓰면 테스트 속도를 높이지만, 잘못 쓰면 테스트 본문에서 시나리오를 지워버린다. 그래서 중요한 것은 “얼마나 많이 공통화했는가”가 아니라, **공통화 이후에도 테스트 의도가 한눈에 보이는가**이다. 아래 체크리스트는 유틸리티 도입 시 의도 숨김을 막기 위한 최소 기준이다.

#### 1) 본문에서 핵심 입력/기대 결과가 보이는가?

Factory/fixture를 쓰더라도 테스트 본문에는 시나리오를 결정하는 값이 남아 있어야 한다.

```kotlin
val data = fixture<OrderSheetSubmittedData>().copy(interpretationId = history.id)
```

이처럼 핵심 변수(`interpretationId`)가 본문에 드러나면, 유틸리티를 쓰면서도 “무엇을 검증하는 테스트인지”를 바로 파악할 수 있다.

#### 2) 반복되는 권한/에러 검증은 헬퍼로 묶되, 검증 포인트(path)는 본문에 남겼는가?

```kotlin
testDeniedRoles(
    allowRoles = setOf(Constants.STORE_ADMIN, Constants.STORE_MANAGER)
) { label, token ->
    test("$label 권한으로 견적 이미지 업로드 url을 조회하면 PERMISSION_DENIED 오류를 반환한다.") {
        val actual = clientBuilder.token(token).build().executeQuery(query, variables).errors
        actual shouldPermissionDeniedWith listOf("quotationImageUploadUrls")
    }
}
```

`testDeniedRoles`로 반복은 줄이고, `quotationImageUploadUrls` 같은 핵심 검증 지점은 본문에서 명시해 의도를 유지하는 방식이 좋다.

#### 3) 파싱 유틸은 보일러플레이트를 숨기되, 도메인 시나리오는 숨기지 않았는가?

```kotlin
val actual =
    clientBuilder.token(testHelper.점주_토큰_생성(manager.id)).build()
        .executeQuery(query)
        .fetchAsObject("terms", typeRef<List<TermsField>>())
```

`fetchAsObject`는 파싱 반복 코드를 줄여주지만, 어떤 필드를 어떤 타입으로 검증하는지는 테스트 본문에서 그대로 읽혀야 한다.

#### 4) 시간/외부연동 헬퍼는 “실행 성공”보다 “계약 검증”을 남기고 있는가?

```kotlin
mockery.leaveSendbirdGroupChannel()

withConstantNow(now) { 
    matchingQuotationInquiryChannelClearScheduler.clearMatchingQuotationInquiryChannels() 
}

mockery.verifyLeaveSendbirdGroupChannel(channelUrl = channelUrl, sendbirdUserIds = sendbirdUserIds)
```

`withConstantNow`와 `Mockery`를 쓰는 목적은 테스트를 통과시키는 것이 아니라, 시간 경계와 외부 요청 계약을 재현 가능하게 검증하는 데 있다.

#### 5) 공통 베이스/확장 함수가 팀 규칙을 자동 적용하는가?

```kotlin
override suspend fun beforeEach(testCase: TestCase) {
    clearAllMocks()
}
```

```kotlin
fun TestEntityManager.persistAll(vararg entities: Any): List<Any> = ...
```

이처럼 “매번 지켜야 하는 규칙”은 베이스/확장 함수로 강제해야 일관성이 유지된다.

---

요약하면, 유틸리티 도입의 목표는 코드량 절감이 아니라 **의도 보존 + 규칙 자동화 + 재현성 강화**이다. 유틸리티를 추가할 때마다 “본문이 더 짧아졌는가?”보다 “시나리오가 더 잘 보이는가?”를 먼저 확인하면, 추상화가 테스트 품질을 해치지 않고 팀 생산성을 안정적으로 높여준다.

## 마무리: 테스트를 ‘지속 가능한 시스템’으로 만드는 방법

이 글에서 강조하고 싶었던 핵심은 결국 하나로 모인다. 테스트는 “많이 쓰는 기술”이 아니라 **계속 유지할 수 있는 시스템**이어야 한다는 점이다. 실무에서 정말 힘든 지점은 테스트를 쓰느냐/안 쓰느냐가 아니라, 시간이 지나도 **지속 가능한 방식으로 쓰고 있느냐**에 더 가깝다고 생각한다. 읽기 어려운 테스트, 목적이 불분명한 테스트, 느려서 잘 돌지 않는 테스트는 결국 팀의 습관에서 멀어지기 마련이기 때문이다.

그래서 이 글의 흐름을 “스타일 → 전략 → 유틸리티”로 잡았다. 스타일은 **읽히는 테스트**를 만들고, 전략은 **어디를 먼저 잠글지**를 정리해 주며, 유틸은 **반복을 줄여 테스트를 일상 업무로 만드는 장치**가 된다. 세 가지가 함께 맞물릴 때 테스트는 ‘개발 속도를 늦추는 비용’이 아니라 ‘개발 속도를 지키는 안전장치’가 된다고 본다.

결국 테스트의 품질은 개별 테크닉보다 **팀이 유지 가능한 구조를 갖췄는가**에 의해 결정된다고 본다. 문서의 규칙보다 더 중요한 것은 그 규칙이 코드로 자동 적용되는가이다.

### 지금 바로 시작할 수 있는 작은 변화

- 테스트 이름 규칙을 팀 합의로 고정한다.
- 권한/에러 검증을 공통 assertion으로 묶는다.
- 시간 의존성은 `withConstantNow` 같은 패턴으로 통일한다.

이 세 가지 중 하나만 적용해도 테스트의 읽기성, 리뷰 속도, 회귀 탐지력이 눈에 띄게 좋아진다. 테스트는 한 번에 바뀌지 않지만, **작은 기준이 쌓이면 팀의 속도는 확실히 달라진다**고 생각한다.

마지막으로, 소개한 테크닉들은 정답이 아니라 참고용 도구라는 것을 말해두고 싶다. 하지만 이 도구들이 모이면, 실무에서 더 **효율적이고 유지 가능한 테스트**를 쓰는 데 분명한 도움이 된다. 이 글을 읽는 분들 각자의 팀 상황에 맞게 선택하고 조합해서, 테스트가 “해야 하는 일”이 아니라 “계속할 수 있는 일”이 되길 바란다.
