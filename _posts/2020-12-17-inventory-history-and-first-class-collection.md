---
layout: single
title: "재고 이력관리와 일급 컬렉션"
categories:
  - EXPLANATION
tags:
  - entity
  - value object
  - domain driven development

toc: true
toc_sticky: true
excerpt: 이 글은 회사에서 재고 관리 서비스의 재고 이력관리 기능을 개발하면서 일급 컬렉션을 사용하게 되었는데, 일급 컬렉션을 어떻게 활용하였는지 소개하는 글이다.
---

## Intro

이 글은 회사에서 재고 관리 서비스의 재고 이력관리 기능을 개발하면서 일급 컬렉션을 사용하게 되었는데, 일급 컬렉션을 어떻게 활용하였는지 소개하는 글이다.

## 일급 컬렉션(First Class Collection)

일급 컬렉션이라는 용어는 [소트웍스 앤솔로지](http://www.yes24.com/Product/Goods/3290339?scode=032&OzSrank=1){: target="\_blank" }라는 책에서 소개된 용어이다. (~~책은 아직 안읽어 봤다.~~)

> 규칙 8: 일급 콜렉션 사용
> 이 규칙의 적용은 간단하다.
> 콜렉션을 포함한 클래스는 반드시 다른 멤버 변수가 없어야 한다.
> 각 콜렉션은 그 자체로 포장돼 있으므로 이제 콜렉션과 관련된 동작은 근거지가 마련된셈이다.
> 필터가 이 새 클래스의 일부가 됨을 알 수 있다.
> 필터는 또한 스스로 함수 객체가 될 수 있다.
> 또한 새 클래스는 두 그룹을 같이 묶는다든가 그룹의 각 원소에 규칙을 적용하는 등의 동작을 처리할 수 있다.
> 이는 인스턴스 변수에 대한 규칙의 확실한 확장이지만 그 자체를 위해서도 중요하다.
> 콜렉션은 실로 매우 유용한 원시 타입이다.
> 많은 동작이 있지만 후임 프로그래머나 유지보수 담당자에 의미적 의도나 단초는 거의 없다.

쉽게 말해서 Collection을 Wrapping한 클래스라고 보면 되는데, 가장 큰 특징이 Collection외 다른 변수가 존재하지 않아야 한다는 것이다.

## 코드를 보자 아니 코드 보기 전에 요구사항 부터 보자

### 이해관계자

현재 다니고 있는 회사에서 상품을 관리하는 역할을 가진 친구들은 MD 친구들과 물류 친구들이다.

MD 친구들은 상품을 소싱하고 관리하는게 주요 역할이다. 그래서 판매할 상품이 선정되면 선매입한 상품의 매타정보(가격, 옵션, 공급가, 옵션 등등)을 입력한 후 상품을 관리자 페이지에서 등록한다.

물류 친구들은 상품의 재고를 관리하는 것이 주요 역할이다. 실제로 창고에 가서 재고 실사를 하기도 하고 우리가 보유한 재고가 몇개인지 확인한 후 전산상에 데이터가 일치하는지 여부등을 체크한다.

### 요구사항

여기에서 재고 이력관리가 왜 필요한지에 대한 물류 친구들의 요구사항이 있는데 요구사항을 보면 아래와 같다.

> 재고 변경에 대한 이력을 확인할 수 있고 MD 친구들이 등록한 재고 수량이 물류 친구들이 검수한 후에 전산상에 재고로 반영되었으면 좋겠다. 그리고 재고 실사 후 재고를 변경할 수 있어야 한다.

위 요구사항의 주요 기능은 `이력관리`와 `재고 반영의 시점 관리`이다.

MD 친구들은 계약한 상품을 등록하는데 이때 원하는 재고 수량을 입력하게 된다. 이 재고 수량은 아직 우리가 관리하는 창고에 입고된 것이 아니기 때문에 재고로 잡을 수 없다. 그래서 재고 요청 상태로 두어야 한다. MD 친구들은 요청한 재고가 언제 입고되는지는 알지 못한다.

MD 친구들은 창고에서 입고된 재고를 확인할 수 있다. 그래서 MD 친구들이 상품 등록 시 요청한 재고 수량에 대해서 창고에 입고되었는지 확인한 후 입고 확인 처리를 해주어야 한다. 그리고 매월 재고 실사를 통해서 재고가 적거나 많은 것을 확인할 수 있고 이를 전산상의 재고와 비교하여 보정한다.

### 모델링

위 요구사항을 토대로 이력을 표현하기 위한 Collection을 만들면 좋겠다고 생각했다. 하지만 List, Set 등과 같은 Collection은 위 요구사항에 맞는 행위를 표현하기가 쉽지 않았다.

예를 들어 이력은 시간 순서를 가져야 하고 현재 재고 이력이 입고 확인을 받아야 하는 상태인지, 계산된 재고 수가 몇 개인지를 표현하려면 Collection이 가지고 있는 기본 함수들을 조합해서 사용해야 한다. 하지만 이렇게 하면 클라이언트 코드(대부분 서비스 객체일 것이다)가 복잡해지고 만약 다른 클라이언트가 이력을 사용한다면 중복되는 코드가 작성되어야 할 것이다. 기능이 복잡해지면 아마 다르게 동작하는 코드가 작성될 수도 있다. 그래서 나는 일급 컬랙션이 이 요구사항에 잘 맞는 모델이라 생각해서 선택해 보았다.

### 참고

재고 변경 이력은 아래 클래스명을 보면 알 수 있겠지만 수동(?)으로 재고를 변경한 경우에만 이력을 쌓도록 하였다. 즉 관리자에서 변경한 것만 이력으로 쌓지 주문에 의한 재고 차감 이력은 기록하지 않는다는 것이다. 주문에 의한 재고 차감까지 기록한다면 재고 변경 이력은 엄청나게 많아질 것이고 사용자가 재고 변경에 대한 이력관리를 더욱 어렵게 할 것이라 판단했다.

## 이제 진짜 코드를 보자

```kotlin
class InventoryManualChangeHistories private constructor(
    private val histories: MutableList<InventoryManualChangeHistory> // 1)
) {
    companion object {
				// 2)
        fun create(data: InventoryManualChangeHistoryCreationDto): InventoryManualChangeHistories {
            val history = InventoryManualChangeHistory(
                state = REQUEST_WAREHOUSING,
                timestamp = ZonedDateTime.now(),
                deltaQuantity = data.quantity,
                currentQuantity = 0,
                memo = "최초 입고 요청",
                registrant = data.userId,
                warehousingRequestDate = data.warehousingRequestDate
            )
            return InventoryManualChangeHistories(mutableListOf(history))
        }

				// 3)
        fun of(histories: List<InventoryManualChangeHistory>): InventoryManualChangeHistories {
						// TODO 최초 이력이 입고요청인지 확인하는 체크 로직 추가 필요
            return InventoryManualChangeHistories(histories.sortedBy { it.timestamp }.toMutableList())
        }
    }

		// 4)
    fun getHistories(): List<InventoryManualChangeHistory> {
        return histories.toList()
    }

		// 5)
    fun getStock(): Int {
        return histories
            .lastOrNull { it.state != REQUEST_WAREHOUSING }
            ?.let {
                if (it.state == DEDUCTED) {
                    it.currentQuantity - it.deltaQuantity
                } else {
                    it.currentQuantity + it.deltaQuantity
                }
            }
            ?: 0
    }

		// 6)
    fun requestWarehousing(data: WarehousingRequestDto) {
        val history = InventoryManualChangeHistory(
            state = REQUEST_WAREHOUSING,
            timestamp = ZonedDateTime.now(),
            deltaQuantity = data.requestDeltaStock,
            currentQuantity = data.currentStock,
            memo = data.memo,
            registrant = data.userId,
            warehousingRequestDate = data.warehousingRequestDate
        )
        histories.add(history)
    }

		// 7)
    fun confirm(data: AdditionInventoryHistoryDto) {
        if (isConfirmed()) {
            throw DomainConstraintException("입고 요청이 있는 이력만 입고 확인이 가능합니다.")
        }

        val history = InventoryManualChangeHistory(
            state = WAREHOUSED,
            timestamp = ZonedDateTime.now(),
            deltaQuantity = data.deltaStock,
            currentQuantity = data.currentStock,
            memo = data.memo,
            registrant = data.userId
        )
        histories.add(history)
    }

    private fun isConfirmed() =
        histories.indexOfLast { it.state == WAREHOUSED } > histories.indexOfLast { it.state == REQUEST_WAREHOUSING }

		// 8)
    fun warehouse(data: AdditionInventoryHistoryDto) {
        val history = InventoryManualChangeHistory(
            state = WAREHOUSED,
            timestamp = ZonedDateTime.now(),
            deltaQuantity = data.deltaStock,
            currentQuantity = data.currentStock,
            memo = data.memo,
            registrant = data.userId
        )
        histories.add(history)
    }

		// 9)
    fun deduct(data: AdditionInventoryHistoryDto) {
        if (data.deltaStock > data.currentStock) {
            throw DomainConstraintException("차감할 재고는 보유수량보다 많을 수 없습니다.")
        }

        val history = InventoryManualChangeHistory(
            state = DEDUCTED,
            timestamp = ZonedDateTime.now(),
            deltaQuantity = data.deltaStock,
            currentQuantity = data.currentStock,
            memo = data.memo,
            registrant = data.userId
        )
        histories.add(history)
    }
}
```

1. 일급 컬랙션의 특징은 변수가 컬랙션만 존재한다는 것이다. 여기서 history는 재고 변경에 대한 이력이고 private 생성자로 외부에서 생성자 접근을 하지 못하도록 하였다. 그 이유는 아래 2, 3번을 보면 알 수 있다. (Kotlin을 모르시는 분들을 위해서 Kotlin은 List가 기본적으로 불변이다. MutableList는 변경가능한 Collection이라고 생각하면 된다)
2. 생성 함수는 최초 이력을 생성할때 사용한다. 이력은 최초 생성 시 입고 요청으로부터 시작해야 하는데 생성 함수는 처음 Collection을 만들어주는 역할을 한다.  최초 재고이력 생성시 사용한다.
3. of 함수는 기존에 존재하던 이력을 토대로 객체를 생성하는 함수이다. DB에서 이력 목록을 가져와서 생성할 때 사용하기 위한 용도인데 이때 입력된 이력이 정렬이 되어있지 않을 수 있으므로 정렬하여 이력을 생성하도록 한다. 왜냐하면 이 객체는 이력의 순서가 중요하기 때문이다. 기존 재고이력이 존재하는 경우 생성자로 사용한다.
4. 이력 조회 함수는 객체 내 컬렉션을 반환하는 함수이다. 반환 시 불변 객체로 반환한다. 모든 재고 이력을 조회하는 용도로 사용한다.
5. 재고 조회 함수는 현재 이력에서 가져올 수 있는 재고의 수를 반환한다. 주로 관리자에서 재고 변경 시 이력의 추가와 함께 상품의 재고에 반영하기 위한 계산된 재고 수를 가져오기 위한 용도로 사용된다.
6. 입고 요청 함수는 입고요청 이력을 추가하는 커맨드 함수이다. MD 친구가 관리자에서 추가 입고 요청에 대한 용도로 사용된다.
7. 입고 확인 함수는 입고요청이 존재하는 경우 입고 확인 이력을 추가하는 함수이다.  물류 친구가 관리자에서 입고요청에 대한 확인에 대한 용도로 사용된다.
8. 추가 입고 함수는 물류 담당자가 창고 재고 실사 후 추가 재고가 발견되어 관리자에서 재고를 추가입고 시키는 용도로 사용한다.
9. 재고 차감 함수는 물류 담당자가 창고 재고 실사 후 누락된 재고수를 관리자에서 적용하기 위한 용도로 사용한다.

## 이점

### 명확한 역할

처음에는 도메인 객체로 만들어볼까도 생각했었다. 하지만 이력 목록을 제외한 나머지 추가적인 속성이 불필요했다. 오히려 추가 속성이 있으므로 객체의 역할이 모호해질것 같아 보였다. 그래서 일급객체로 사용하는게 좀더 명확한 역할을 부여하고 의미있는 이름을 부여할 수 있었다.

### 비지니스에 종속적인 자료구조

일반적인 Collection은 범용적으로 사용하는 함수만 제공한다. `filter`, `map`, `sum`... 등등

하지만 일급 컬랙션을 사용함으로써 비지니스 요구사항을 구현한 비지니스 로직을 해당 클래스 내부로 끌어들일 수 있다. 이렇게 함으로써 우리는 서비스 객체가 비대해 지는 것을 방지할 수 있고 재사용 가능성을 높일 수 있다.

### 손쉬운 테스트 코드 작성

서비스 클래스와 일반 클래스 중 어느 클래스가 테스트하기 쉬운지 묻는다면 당연히 일반 클래스가 테스트하기 쉬울 것이라 말할 것이다. 이력관리에 대한 비지니스 로직을 서비스 클래스에 두지 않고 일급 컬랙션을 만들어 사용한다면 테스트하기가 용이하다.

아래는 [Kotest](https://kotest.io/){: target="\_blank" }로 이력 목록 일급 객체에 대한 테스트 코드이다.

```kotlin
class InventoryManualChangeHistoriesTest : FreeSpec() {
    private lateinit var sut: InventoryManualChangeHistories

    override fun beforeTest(testCase: TestCase) {
        mockkStatic(ZonedDateTime::class)
        every { ZonedDateTime.now() } returns ZonedDateTime
            .of(2020, 1, 1, 0, 0, 0, 0, ZoneId.systemDefault())

        val creation = InventoryManualChangeHistoryCreationDto(
            itemId = 1,
            barcode = "A",
            quantity = 10,
            warehousingRequestDate = ZonedDateTime
                .of(2020, 1, 10, 0, 0, 0, 0, ZoneId.systemDefault()),
            userId = "k@101.inc"
        )
        sut = InventoryManualChangeHistories.create(creation)
    }

    override fun afterTest(testCase: TestCase, result: TestResult) {
        clearAllMocks()
    }

    init {
        "생성 함수는 데이터를 입력받아 입고요청 이력을 생성하고 0인 재고를 가진다" {
            // Given
            val data = InventoryManualChangeHistoryCreationDto(
                itemId = 1,
                barcode = "A",
                quantity = 10,
                warehousingRequestDate = ZonedDateTime
                    .of(2020, 1, 10, 0, 0, 0, 0, ZoneId.systemDefault()),
                userId = "k@101.inc"
            )

            // When
            val actual = InventoryManualChangeHistories.create(data)

            // Then
            val expected = InventoryManualChangeHistory(
                state = REQUEST_WAREHOUSING,
                timestamp = ZonedDateTime.now(),
                deltaQuantity = 10,
                currentQuantity = 0,
                memo = "최초 입고 요청",
                registrant = "k@101.inc",
                warehousingRequestDate = ZonedDateTime
                    .of(2020, 1, 10, 0, 0, 0, 0, ZoneId.systemDefault())
            )
            assertSoftly {
                actual.getHistories() shouldContainAll listOf(expected)
                actual.getStock() shouldBe 0
            }
        }

        "of 함수는 이력목록을 입력받아 정렬된 이력목록을 생성한다" {
            // Given
            val histories = listOf(
                InventoryManualChangeHistory(
                    state = WAREHOUSED,
                    timestamp = ZonedDateTime
                        .of(2020, 1, 10, 0, 0, 1, 0, ZoneId.systemDefault()),
                    deltaQuantity = 10,
                    currentQuantity = 0,
                    memo = "입고 확인",
                    registrant = "k@101.inc"
                ),
                InventoryManualChangeHistory(
                    state = REQUEST_WAREHOUSING,
                    timestamp = ZonedDateTime
                        .of(2020, 1, 10, 0, 0, 0, 0, ZoneId.systemDefault()),
                    deltaQuantity = 10,
                    currentQuantity = 0,
                    memo = "최초 입고 요청",
                    registrant = "k@101.inc"
                )
            )

            // When
            val actual = InventoryManualChangeHistories.of(histories)

            // Then
            val expected = listOf(
                InventoryManualChangeHistory(
                    state = REQUEST_WAREHOUSING,
                    timestamp = ZonedDateTime
                        .of(2020, 1, 10, 0, 0, 0, 0, ZoneId.systemDefault()),
                    deltaQuantity = 10,
                    currentQuantity = 0,
                    memo = "최초 입고 요청",
                    registrant = "k@101.inc"
                ),
                InventoryManualChangeHistory(
                    state = WAREHOUSED,
                    timestamp = ZonedDateTime
                        .of(2020, 1, 10, 0, 0, 1, 0, ZoneId.systemDefault()),
                    deltaQuantity = 10,
                    currentQuantity = 0,
                    memo = "입고 확인",
                    registrant = "k@101.inc"
                )
            )
            actual.getHistories() shouldBe expected
        }

        "입고 요청 함수는 요청 데이터를 입력받아 입고요청 이력을 생성하고 재고를 추가하지 않는다" {
            // Given
            val data = WarehousingRequestDto(
                requestDeltaStock = 10,
                currentStock = 0,
                warehousingRequestDate = ZonedDateTime
                    .of(2020, 1, 11, 0, 0, 0, 0, ZoneId.systemDefault()),
                memo = "추가 입고 요청",
                userId = "k@101.inc"
            )

            // When
            sut.requestWarehousing(data)

            // Then
            sut.getHistories().last() shouldBe InventoryManualChangeHistory(
                state = REQUEST_WAREHOUSING,
                timestamp = ZonedDateTime.now(),
                deltaQuantity = 10,
                currentQuantity = 0,
                memo = "추가 입고 요청",
                registrant = "k@101.inc",
                warehousingRequestDate = ZonedDateTime
                    .of(2020, 1, 11, 0, 0, 0, 0, ZoneId.systemDefault()),
            )
            sut.getStock() shouldBe 0
        }

        "입고 확인 함수는" - {
            "데이터를 입력받아 재고 증가 이력을 생성하고 재고를 확정한다" {
                // Given
                val data = AdditionInventoryHistoryDto(
                    deltaStock = 10,
                    currentStock = 0,
                    memo = "입고 확인",
                    userId = "k@101.inc"
                )

                // When
                sut.confirm(data)

                // Then
                sut.getHistories().last() shouldBe InventoryManualChangeHistory(
                    state = WAREHOUSED,
                    timestamp = ZonedDateTime.now(),
                    deltaQuantity = 10,
                    currentQuantity = 0,
                    memo = "입고 확인",
                    registrant = "k@101.inc"
                )
                sut.getStock() shouldBe 10
            }

            "입고 요청이 없는 이력은 입고 확인을 할 수 없다" {
                // Given
                val data = AdditionInventoryHistoryDto(
                    deltaStock = 10,
                    currentStock = 0,
                    memo = "입고 확인",
                    userId = "k@101.inc"
                )
                sut.confirm(data)

                // Expected
                val actual = assertThrows<DomainConstraintException> {
                    sut.confirm(data)
                }

                actual.message shouldBe "입고 요청이 있는 이력만 입고 확인이 가능합니다."
            }
        }

        "재고 추가 함수는 데이터를 입력받아 재고 증가 이력을 생성하고 기존 재고를 증가 시킨다" {
            // Given
            sut.confirm(
                AdditionInventoryHistoryDto(
                    deltaStock = 10,
                    currentStock = 0,
                    memo = "입고 확인",
                    userId = "k@101.inc"
                )
            )

            val data = AdditionInventoryHistoryDto(
                deltaStock = 5,
                currentStock = 10,
                memo = "재고 증가",
                userId = "k@101.inc"
            )

            // When
            sut.warehouse(data)

            // Then
            sut.getHistories().last() shouldBe InventoryManualChangeHistory(
                state = WAREHOUSED,
                timestamp = ZonedDateTime.now(),
                deltaQuantity = 5,
                currentQuantity = 10,
                memo = "재고 증가",
                registrant = "k@101.inc"
            )
            sut.getStock() shouldBe 15
        }

        "재고 차감 함수는" - {
            "데이터를 입력받아 재고 차감 이력을 생성하고 기존 재고를 차감 시킨다" {
                // Given
                sut.confirm(
                    AdditionInventoryHistoryDto(
                        deltaStock = 10,
                        currentStock = 0,
                        memo = "입고 확인",
                        userId = "k@101.inc"
                    )
                )

                val data = AdditionInventoryHistoryDto(
                    deltaStock = 5,
                    currentStock = 10,
                    memo = "재고 차감",
                    userId = "k@101.inc"
                )

                // When
                sut.deduct(data)

                // Then
                sut.getHistories().last() shouldBe InventoryManualChangeHistory(
                    state = DEDUCTED,
                    timestamp = ZonedDateTime.now(),
                    deltaQuantity = 5,
                    currentQuantity = 10,
                    memo = "재고 차감",
                    registrant = "k@101.inc"
                )
                sut.getStock() shouldBe 5
            }

            "보유한 재고보다 많은 재고를 차감할 수 없다" {
                // Given
                sut.confirm(
                    AdditionInventoryHistoryDto(
                        deltaStock = 10,
                        currentStock = 0,
                        memo = "입고 확인",
                        userId = "k@101.inc"
                    )
                )

                val data = AdditionInventoryHistoryDto(
                    deltaStock = 15,
                    currentStock = 10,
                    memo = "재고 차감",
                    userId = "k@101.inc"
                )

                // Expected
                val actual = assertThrows<DomainConstraintException> {
                    sut.deduct(data)
                }
                actual.message shouldBe "차감할 재고는 보유수량보다 많을 수 없습니다."
            }
        }
    }
}
```

### 그외

그 외에도 블로그 글을 보면 상태와 행위를 한곳에서 관리할 수 있다라는 장점과 불변성을 보장해준다는 장점 등등을 볼 수 있다. 사실 상태와 행위를 한곳에서 관리한다는 점은 일급 컬랙션만이 가진 장점은 아니라고 생각이 든다. 그리고 불변성에 대한 보장은 Java가 아닌 Kotlin을 사용해서 개발한다면 해결할 수 있는 부분이라 이부분도 두드러진 장점이라고 말하기 어려운 부분이 있는 것 같다.

## Wrap Up

이력관리와 같은 도메인은 어디서든 앞에서 말한 재고 이력관리 말고도 어디서든 사용될 수 있는 도메인이다. 단순히 List나 Map과 같은 자료구조만 사용하지 말고 일급 컬랙션을 사용해서 도메인을 코드를 작성하는 연습을 해보면 좋을 것 같다.

> [https://rok93.tistory.com/m/81](https://rok93.tistory.com/m/81)
