---
layout: single
title: "The review of sku service development"
categories:
  - EXPLANATION
tags:
  - DDD
  - kotlin
  - Event Driven
  - JPA

toc: true
toc_sticky: true
excerpt: 이 글은 내가 회사에서 SKU 기능을 개발하면서 고민했던 내용들을 동료들에게 공유하면서 정리했던 내용을 옮긴 글이다. 기능 개발을 맡게 되면서 평소 해보고 싶었던 것들을 도입해 보았고 그것을 해보면서 많은 것들을 배우고 느낄 수 있었다. 아직 갈길이 멀지만 개인적으로는 좋은 경험이었고 앞으로도 꾸준히 더 좋은 서비스 개발을 위해서 노력해 가야겠다. 그럼 본 내용을 시작해 보자.
---

## Intro

이 글은 내가 회사에서 SKU 기능을 개발하면서 고민했던 내용들을 동료들에게 공유하면서 정리했던 내용을 옮긴 글이다. 기능 개발을 맡게 되면서 평소 해보고 싶었던 것들을 도입해 보았고 그것을 해보면서 많은 것들을 배우고 느낄 수 있었다. 아직 갈길이 멀지만 개인적으로는 좋은 경험이었고 앞으로도 꾸준히 더 좋은 서비스 개발을 위해서 노력해 가야겠다. 그럼 본 내용을 시작해 보자.

## SKU 서비스란?

SKU 서비스란 자사에서 제공하는 SKU(실물, 클래스, 구독권 등등)들의 메타정보 제공 및 재고 관리 기능을 담당하는 서비스를 말한다. 흔히 물류에서 말하는 SKU와는 조금 다른 개념이지만 편의상 SKU라고 부르고 있다.

## 주요 도메인

### PACK

이형상품을 표현해주는 도메인으로 SKU를 묶어주는 역할을 한다. 예를 들면 아래와 같다.

> "연필"이라는 별도의 옵션이 없는 상품이 있다고 하면 연필은 하나의 SKU를 가진 Pack으로 구성된다.

> "아이패드"라는 여러개의 옵션이 있는 (색상, 용량) 상품이 있다고 하면 아이패드는 여러개의 SKU를 가진 Pack으로 구성된다.

```kotlin
class Pack(
    val id: Long = DEFAULT_ID,
    private var title: String,
    private var skus: List<Sku>,
    private var shown: Boolean,
    private var deleted: Boolean = false,
    private var audit: Audit
)
```

### SKU

자사 서비스에서 제공하는 재고를 가진 상품의 최소단위를 나타내는 도메인을 말한다. 자사 서비스에는 실물인 키트 뿐만 아니라 클래스, 구독권, 코칭권 등 무형의 상품도 SKU로 사용된다. 현재는 실물인 키트만 개발된 상태이다.

```kotlin
class Kit(
    val id: Long = DEFAULT_ID,
    val packId: Long = DEFAULT_ID,
    private var name: String,
    private var barcode: String,
    private var thumbnail: String,
    private var price: Money,
    private var taxType: TaxType,
    private var inventory: Inventory,
    private var settlement: Settlement,
    private var shelfLife: ShelfLife,
    private var shippings: List<Shipping>,
    private var options: List<Option>,
    private var tags: List<Tag>,
    private var audit: Audit,
    private var shown: Boolean,
    private var deleted: Boolean = false,
    private var sort: Int
)
```

## 개발 시 고민했던 것들?

### Domain Model과 Persistence Model의 분리

도메인 모델과 영속성 모델을 같은 모델로 사용하는 경우가 많다.

도메인 모델의 목적은 이해관계자의 요구사항을 나타낸 것이고 영속성 모델은 데이터 영속화를 위해 존재한다. 그래서 도메인 모델은 요구사항을 잘 표현하는 것에 집중해야 하고 영속성 모델은 데이터를 잘 저장하기 위해 집중해야 한다. 그렇기 때문에 두 모델의 역할은 다르고 분리되어야 한다고 생각한다.

두 모델은 거의 동일한 속성을 가질 수도 있고 그렇기 때문에 두 모델을 합쳐서 하나의 모델로써 사용해도 무방하다. 하지만 두 모델이 다르다는 것을 알고 합쳐서 사용하는 것과 모르고 합쳐서 사용하는 것은 다르기 때문에 이 부분을 알고 함께 사용했으면 좋겠다.

아래는 내가 이러한 생각을 가지게 해준 글이다.

[Just Stop It! The Domain Model Is Not The Persistence Model](https://blog.sapiensworks.com/post/2012/04/07/Just-Stop-It!-The-Domain-Model-Is-Not-The-Persistence-Model.aspx){: target="\_blank" }

도메인 모델을 분리한 가장 큰 이유는 추상화였다.

SKU 도메인에서 가진 속성 중 **정산정보**(Settlement)라는 속성이 있다. 정산정보는 2가지 유형이 있는데 하나는 **고정정산**이고 다른하나는 **비율정산**이다. **고정정산**은 공급가가 고정되어 있어 나중에 정산 시 공급가와 판매금액을 이용해 정산을 하는 것이고, **비율정산**은 정산비율이 주어져서 이 비율을 통해 공급가를 결정해 정산을 하는 방식이다.

이러한 요구사항을 아래와 같이 도메인 모델로 표현하였다.

```kotlin
// 정산정보
interface Settlement {
    val id: Long
    val store: Store
}
```

```kotlin
// 고정정산
class FixedSettlement(
    override val id: Long = DEFAULT_ID,
    val supplyPrice: Money,
    val taxType: TaxType,
    override val store: Store
) : Settlement
```

```kotlin
// 비율정산
class RatioSettlement(
    override val id: Long = DEFAULT_ID,
    val ratio: Float,
    val taxType: TaxType,
    override val store: Store
) : Settlement
```

영속성 모델로 표현하면 아래와 같다

```kotlin
@Entity(name = "settlements")
data class SettlementEntity(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private var id: Long = DEFAULT_ID,

    @Column(length = 100, nullable = false)
    @Enumerated(EnumType.STRING)
    private var type: SettlementType,

    @Embedded
    @AttributeOverrides(
        AttributeOverride(name = "amount", column = Column(name = "supply_price_amount")),
        AttributeOverride(name = "currency", column = Column(name = "supply_price_currency"))
    )
    private var supplyPrice: MoneyEmbed? = null,

    @Column(columnDefinition = "float")
    private var ratio: Float? = null,

    @Column(length = 100, nullable = false)
    @Enumerated(EnumType.STRING)
    private var taxType: TaxType,

    @Embedded
    private var store: StoreEmbed
)
```

코드에서 볼 수 있듯이 영속성 모델에서(~~JPA 에서~~) 인터페이스를 이용하여 Entity를 정의할 수 없으므로 타입(type)이라는 속성이 있는 것을 볼 수 있다. 그리고 고정정산과 비율정산에서 필요로 하는 속성이 다르므로 공급가와 비율이 nullable 한것도 볼 수 있다.

만약 도메인 모델을 사용하지 않고 영속성 모델을 그대로 사용해 로직을 구현한다면 수많은 조건문이 비지니스 코드에 녹아 들어야 할 것이다. 하지만 위와 같이 도메인 모델을 추상화 한다면 각 클래스별로 행위를 분리해서 정의할 수 있으므로 조건문을 많이 줄일 수 있다. 만약 또다른 정산유형이 추가된다면 단순히 클래스만 추가하면 되니 확장성도 용이하다.

### 의존성의 방향성

![onion-architecture](/assets/images/posts/review-of-sku-service-development/onion-architecture.png)

어플리케이션의 패키지 구조를 잡으면서 위 그림과 같이 의존성 방향성을 가지도록 노력을 했다.

위에서 정의한 도메인 모델은 가장 안쪽에 위치한다. 그래서 도메인 모델의 코드는 어떠한 어플리케이션의 코드(Spring, DB 등)와도 의존하지 않도록 구현했다. 그렇기 때문에 만약 어플리케이션을 구현하기 위한 프레임워크가 변경되더라도(~~그럴일이 거의 없겠지만~~) 우리는 도메인 모델의 코드만 떼어내어 다른 프레임워크로 갈아끼울 수 있을 것이다.

여기서 위에 도메인 모델과 영속성 모델을 분리한 이유가 또 나오는데, 영속성 모델은 도메인 모델 레이어에 위치하지 않고 인프라 어플리케이션 레이어에 위치한다. 왜냐하면 영속성 모델은 JPA에 의존하기 때문이다.

위 내용을 바탕으로 패키지 구조를 아래와 같이 구성해 보았다. Spring에서 멀티 모듈을 지원해서 domain 패키지를 아예 분리해서 물리적으로 도메인 코드가 어플리케이션 코드를 사용하지 못하게 막을 수도 있었지만 그부분은 추후에 고민해 보기로 했다.

![package-structure](/assets/images/posts/review-of-sku-service-development/package-structure.png)

## DDD

~~DDD 알못이지만~~ "도메인 모델이 가진 행위를 풍부하게 하고 서비스 객체는 도메인의 행위를 사용하게 한다"라는 관점에서 비지니스 로직을 구현하려고 해보았다.

![service-domain-uml](/assets/images/posts/review-of-sku-service-development/service-domain-uml.jpg)

아래코드와 같디 요구사항을 담고 있는 비지니스 로직은 도메인 클래스가 담고 있고 서비스 객체는 도메인 클래스에게 명령을 내리는 방식으로 구현했다.

```kotlin
// SKU 도메인
class Kit {
	
	// ...

	// 재고 변경
	fun changStock(changeQuantity: Int, userId: String) {
	    val changedQuantity = inventory.quantity + changeQuantity
	
	    if (changedQuantity < 0) {
	        throw OutOfStockException()
	    }
	
	    inventory = inventory.copy(quantity = changedQuantity)
	    audit = audit.copy(
	        modifier = userId,
	        updatedAt = ZonedDateTime.now()
	    )
	}
}
```

```kotlin
// 서비스
@Service
class SimpleSkuService {

	// ...

	// 재고 변경 요청
	@Transactional(timeout = 3)
  override fun changeStock(data: SkuChangeStockCommand) {
      val entity = skuRepository.findByIdForUpdate(data.id)
          .orElseThrow { NotFoundException.withId(data.id) }

      val sku = entity.toDomain()
      sku.changStock(data.quantity, data.userId)

      val event = SkuStockChangedEvent(sku)
      publisher.publishEvent(event)
  }
}
```

## Event Driven

위 코드에서 보았겠지만 서비스 코드에서 도메인에 특정 행위를 요청 한 후 이벤트를 발생시키는 것을 볼 수 있다.

이벤트를 발생 시키면 해당 이벤트를 구독하고 있는 리스너가 이벤트를 수신하여 데이터를 영속화 하도록 구현하였다.

```kotlin
@Component
class SkuEventListener {
	
	// ...

	@EventListener
  fun handle(event: SkuStockChangedEvent) {
      val entity = event.sku.toEntity()
      skuRepository.save(entity)
  }
}
```

이렇게 구현하였을 때 가질 수 있는 장점은 아래와 같다.

- 서비스간의 느슨한 결합

    이벤트를 발행하는 서비스 코드에서는 어떤 수신자가 있는지 알 필요가 없고 수신자도 수신되는 메시지에만 의존하므로 각 서비스간에는 결합도가 느슨하다라는 장점을 가진다. 결합도가 낮다는 것은 코드 변경에 대한 부작용을 줄일 수 있다는 것이다.

- 확장성

    만약 위 코드에서처럼 데이터를 영속화 하는것 외에 재고가 변경되었을 때 특정 사용자에게 이메일을 발송해야 한다라는 기능이 추가되면 어떻게 하면 될까? 아래와 같이 단순히 리스너를 추가하고 그에 맞는 코드만 작성하면 된다. 서비스 코드는 전혀 변경하지 않아도 되므로 변경에 대한 리스크를 가지지 않고 새로운 기능을 추가할 수 있다.

    ```kotlin
    @EventListener
    fun handle(event: SkuStockChangedEvent) {
        messageService.sendMessage(sku.getStock())
    }
    ```

단점 또한 존재한다

- 디버깅의 어려움

    결합도가 낮으니 코드를 쫓아 디버깅을 할 때 어려움이 있다. 다행히 IDEA에서 리스너를 추적해 주므로 어느정도 이 문제를 해결할 수는 있다.

- 복잡성

    트랜젝션 관리라던지 실패에 대한 처리 등등 복잡성이 증가할 수 있다. 이부분은 장점과 함께 충분히 고민해보고 선택해야 할것이라 생각한다.

## Test

개발된 SKU 서비스는 TDD로 개발하였으며 현재는 유닛테스트로만 작성되어 있다. API에 대한 통합 테스트는 추가할 예정이다.

![jacoco-report](/assets/images/posts/review-of-sku-service-development/jacoco-report.png)

> 단에 커버리지가 낮은 이유는 QueryDSL에서 자동으로 생성한 클래스 때문이고 해당 클래스는 테스트 커버리지에서 제외하였다.

계획하고 있는 배포 전략이 코드가 master에 병합되면 프로덕션에 자동으로 배포되도록 하여 짧은 주기로 코드가 병합되고 어플리케이션에 반영되도록 하는 것이다. 이를 위해서는 자동화된 테스트 코드는 필 수 인데 테스트 코드를 작성하도록 강제하는 방법이 필요해서 테스트 커버리지 도구인 [Jacoco](https://www.jacoco.org/){: target="\_blank" }를 사용하였다.

커버리지 제한은 라인 커버리지는 80%, 브런치 커버리지는 70%으로 두었다. 브런치 커버리지를 70%으로 둔 이유는 간단한 switch문을 작성해서 테스트를 하였을 때 70% 수준으로 나오기 때문이다.

```kotlin
tasks.jacocoTestCoverageVerification {
    violationRules {
        rule {
            element = "CLASS"

            limit {
                counter = "BRANCH"
                value = "COVEREDRATIO"
                minimum = 0.70.toBigDecimal()
            }

            limit {
                counter = "LINE"
                value = "COVEREDRATIO"
                minimum = 0.80.toBigDecimal()
            }

            excludes = listOf(
                "*.ApplicationKt",
                "*.Application",
                "*.infrastructure.config.*",
                "*.infrastructure.persistence.entity.Q*",
                "*.infrastructure.util.InlineFunctionsKt.*"
            )
        }
    }
}
```

또한 Test 코드 작성 시 BDD 방식으로 작성하였다. 최대한 테스트 실행 결과만 보더라도 시스템이 어떤 역할을 하는지 보여주는 것이 목표였다.

![test-result](/assets/images/posts/review-of-sku-service-development/test-result.png)

JUnit으로 작성하면 BDD코드를 작성하는 것이 번거롭기 때문에 [Kotest](https://github.com/kotest/kotest){: target="\_blank" }라는 라이브러리를 사용해서 테스트 코드를 작성하였다.

```kotlin
"changeStock 함수는" - {
    val sut = domainFixture<Kit>().copy(inventory = Inventory(100, 10))
    val userId = fixture<String>()
    val prevUpdatedAt = sut.getAudit().updatedAt

    "변경할 수가 주어지면" - {
        val changeQuantity = -10
        sut.changStock(changeQuantity, userId)
        "올바른 재고수량으로 변경한다" {
            sut.getInventory().quantity shouldBe 90
            sut.getAudit().modifier shouldBe userId
            sut.getAudit().updatedAt shouldNotBe prevUpdatedAt
        }
    }

    "재고보다 큰 변경할 수가 주어지면" - {
        val changeQuantity = -110
        "OutOfStockException 을 던진다" {
            shouldThrow<OutOfStockException> {
                sut.changStock(changeQuantity, userId)
            }
        }
    }
}
```

## Code Convention

코드 컨벤션을 지키면서 개발하면 가독성을 상당히 높일 수 있다. 이러한 이유로 프로젝트 내에서 누가 작성해도 Kotlin의 공식 컨벤션을 지키도록 하고 싶었고, [editorconfig](https://editorconfig.org/){: target="\_blank" }와 [ktlint](https://github.com/pinterest/ktlint){: target="\_blank" }를 이용해서 컨벤션을 지키도록 강제하였다.

editorconfig는 IDEA에서 컨벤션을 설정할 수 있도록 해주는 도구이며 Intellij 뿐만 아니라 수많은 IDEA에서 설정지원을 하고 있다.

ktlint는 ESLint 처럼 Kotlin의 컨벤션을 체크해주는 도구이다.

## Review

- 개발 하면서 익숙하지 않은 방식으로 개발하다보니 단순한 기능임에도 상당한 시간을 할애했다.
- 도메인 모델과 영속성 모델을 분리하다 보니 코드가 상당히 많아 졌다. DTO까지 더하면 클래스 개수가 적진 않다.
- 섣부른 추상화 = 야근
- SKU는 단순하지 않았다...
- 앞으로 보완해야 할 것들이 아직 좀 남아 있다.
- 마이그레이션 작업과 아폴로 서버에서 SKU 서비스로 교체 작업 계획을 해야 한다.

> [https://www.jacoco.org/](https://www.jacoco.org/){: target="\_blank" }
>
> [https://editorconfig.org/](https://editorconfig.org/){: target="\_blank" }
>
> [https://github.com/pinterest/ktlint](https://github.com/pinterest/ktlint){: target="\_blank" }
>
