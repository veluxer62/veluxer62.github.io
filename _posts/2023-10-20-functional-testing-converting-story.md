---
layout: single
title: "기능 테스트 전환 이야기"
categories:
  - REFERENCE
tags:
  - 테스트
  - 기능테스트
  - kotest
  - mockserver
toc: true
toc_sticky: true
excerpt: "모두가 노력해 준 덕분에 올해 목표로 했던 기능 테스트로의 전환을 마무리할 수 있게 되었다. 그래서 그동안 우리가 통합 테스트에서 기능테스트로 왜 전환하게 되었는지, 전환하면서 어떤 이슈들을 겪었는지 소개해보고자 한다."
---

## Intro

이 글은 사내 블로그에 작성한 [기능 테스트 전환 이야기](https://spoqa.github.io/2023/10/20/functional-testing-converting-story.html){: target="\_blank" } 내용을 그대로 가져오면서 나의 블로그의 언어톤에 맞게 변경한 글이다.

2월에 작성한 [청구/수납 서비스 개발기](https://veluxer62.github.io/reference/bill-payment-development-story/){: target="\_blank" } 를 이후로 오랜만에 글을 작성하는 것 같다. 이번 글은 지난번과 같은 서비스에 대한 내용이 아닌 좀 더 기술적인 내용을 다루어 보려고 한다.

2022년에 [서버 언어 전환 이야기](https://veluxer62.github.io/reference/all-new-server/){: target="\_blank" } 글을 작성했었다. 당시 우리 백엔드 챕터에서는 언어 전환을 하면서 테스트 코드 작성에 대한 숙련도 문제, 데이터 초기화의 불편함 등과 같은 여러 이유로 인해 통합테스트를 선택하게 되었었고 이로 인해 아래와 같은 이슈들을 겪게 되었다.

![all-new-server-jira-issue](/assets/images/posts/functional-testing-converting-story/all-new-server-jira-issue.png "JPA 이슈")

그동안 우리 백엔드 챕터는 서비스를 발전 시켜가면서 수많은 테스트 코드를 작성하였고 자연스레 테스트 코드 작성에 대한 숙련도를 높일 수 있게 되었다. 그래서 기능테스트 작성에 대한 난이도를 어느 정도 감수할 수 있다고 판단하게 되었다.

그래서 통합테스트에 대한 단점을 보완하고 좀 더 변경에 대한 안정감을 느끼며 서비스를 개발하기 위해 그동안 묵혀두었던(?) 기능 테스트로의 전환을 아래와 같이 2023년 목표로 설정하고 제품을 개발할 때 틈나는 대로 전환 작업을 진행해 왔다.

![2023-goal](/assets/images/posts/functional-testing-converting-story/2023-goal.png "2023년 백엔드 목표")

모두가 노력해 준 덕분에 올해 목표로 했던 기능 테스트로의 전환을 마무리할 수 있게 되었다. 그래서 그동안 우리가 통합 테스트에서 기능테스트로 왜 전환하게 되었는지, 전환하면서 어떤 이슈들을 겪었는지 소개해보고자 한다.

## 기능 테스트?

테스트 방법을 이야기할 때 [단위 테스트](https://en.wikipedia.org/wiki/Unit_testing){: target="\_blank" } 나 [통합 테스트](https://en.wikipedia.org/wiki/Integration_testing){: target="\_blank" } 는 수많은 서적이나 블로그 글에서 다루어질 만큼 개발자에게 익숙한 테스트 방법일 것으로 생각한다. 하지만 기능 테스트는 여러 매체에서 언급되지 않을 정도로 익숙한 용어는 아닌 것 같다. 그래서 먼저 이 글에서 전반적으로 이야기할 기능 테스트라는 단어를 먼저 정의하고 시작해 볼까 한다. 테스트는 여러 가지 의미로 해석된다. [E2E(End to End) 테스트](https://www.browserstack.com/guide/end-to-end-testing){: target="\_blank" } 로 보기도 하고 통합 테스트, 혹은 [시스템 테스트](https://en.wikipedia.org/wiki/System_testing){: target="\_blank" } 와 유사한 테스팅 기법으로 해석되기도 한다.

[위키피디아의 기능 테스트](https://en.wikipedia.org/wiki/Functional_testing){: target="\_blank" } 를 보면 아래와 같이 정의되어 있다.

> In software development, functional testing is a quality assurance (QA) process and a type of black-box testing that bases its test cases on the specifications of the software component under test. (소프트웨어 개발에서 기능 테스트는 품질 보증(QA) 프로세스이며 테스트 대상 소프트웨어 구성요소의 사양을 바탕으로 테스트 사례를 작성하는 일종의 블랙박스 테스트를 말한다)

특히, 아래 구문에서 기능 테스트가 E2E 테스트와 구분이 된다고 생각한다.

> Functional testing does not imply that you are testing a function (method) of your module or class. Functional testing tests a slice of functionality of the whole system. (기능 테스트는 모듈 또는 클래스의 기능(방법)을 테스트한다는 의미가 아니다. 기능 테스트는 전체 시스템의 일부 기능을 테스트한다)

E2E 테스트는 End to End 즉, 종단 간 테스트를 의미한다. 보통 실제 사용자 관점에서 테스트를 수행하기 때문에 브라우저 환경에서 테스트하는 [Selenium](https://www.selenium.dev/){: target="\_blank" }, [appium](https://appium.io/){: target="\_blank" } 과 같은 도구를 통해 테스트를 수행한다. 하지만 우리는 서버의 API 범위에 한정해서 테스트하므로 E2E 테스트라고 보기 어렵다고 생각했다.

한편, 누군가는 통합 테스트와 같은 의미이지 않은가 하고 이야기할 수 있을 것 같다. 통합 테스트가 단위 테스트를 거친 여러 모듈을 그룹화하여 테스트를 적용한다는 점과 단위 테스트와 시스템 테스트(혹은 E2E 테스트)의 사이에 진행한다는 점에서 같은 의미로 해석될 수 있다고 생각한다.

사실 통합 테스트를 좀 더 나은 테스트 방식으로 변경했다고 봐도 될 것 같긴 하다. 다만 동일한 통합 테스트라고 명명하기보다 좀 더 명시적으로 구분할 수 있는 테스트 방법을 표현할 수 있는 명칭이 필요하다고 생각했다. 그리고 일부 모듈들의 집합을 테스트할 수 있는 통합 테스트에 비해 API의 끝점(Endpoint)에서 블랙박스로 테스트한다는 부분이 좀 더 강조되는 기능 테스트가 우리가 전환하는 목적성에 잘 부합한다고 생각해서 기능 테스트라고 부르기로 하였다.


## 왜 기능 테스트로 전환하기로 하였나?

서버의 언어 전환을 진행한 이후로 백엔드 챕터에서는 모든 제품의 기능을 개발할 때 단위 테스트와 통합테스트를 작성하도록 하였다. 코드 리뷰를 할 때도 동료가 작성한 코드의 스타일 뿐만 아니라 누락된 테스트 케이스가 없는지 체크해 줌으로써 기능적인 결함이 없도록 모두가 노력하였고 그 덕인지 QA에서 발생하는 서버의 버그 비율은 10% 내외로 유지할 수 있게 되었다.

우리는 여기에 만족하지 않고 좀 더 버그 비율을 줄일 방법이 있을지 찾아보았다. 백엔드 전체 버그 이슈의 절반은 관리자와 관련된 이슈이고 그 나머지 중 1/3은 불명확한 요구사항으로 인해서, 나머지 1/3은 코드 오류 또는 테스트 케이스 누락, 나머지 1/3은 통합테스트에서 발견하지 못한 버그로 인해 발생한 것이었다.

불명확한 요구사항이나 개발자의 실수로 인한 버그는 백엔드 챕터 외적인 요소이거나 장기적으로 개발자의 실수를 줄일 방안을 찾아나가야 한다고 판단해서 제외하고 마지막 1/3인 통합테스트에서 발견되지 않은 버그로 인한 이슈에 집중하기로 하였다. 그럼 먼저 통합테스트로 인해 발견하지 못한 대표적인 사례들을 소개해 보면서 왜 통합 테스트에서 버그를 발견하지 못하게 되었는지 소개해 보겠다.

### 사례

#### Hibernate Lazy Loading

먼저 Hibernate의 Lazy Loading으로 인해 발생한 버그이다. 구현된 버그가 발생한 코드를 먼저 보고 이야기를 이어가 보겠다.

```kotlin
@Entity
class OrderableVendorProduct {
    // 생략

    @OneToMany(mappedBy = "product", fetch = FetchType.LAZY, cascade = [CascadeType.ALL], orphanRemoval = true)
    protected val singleProductBundle: MutableSet<OrderableVendorProductBundle> = mutableSetOf()
    val productBundle get() = singleProductBundle.firstOrNull()

    fun update(data: OrderableVendorProductUpdateData) {
        name = data.name
        unit = data.unit
        standard = data.standard
        erpCode = data.erpCode
        unitPrice = data.unitPrice
        isMarketPrice = data.isMarketPrice
    }

    fun syncBundle() {
        if (productBundle == null) {
            throw IllegalStateException("묶음품목이 없는 품목은 묶음을 동기화할 수 없습니다.")
        }

        val bundleProduct = productBundle!!.bundleProduct

        bundleProduct.name = this.name
        bundleProduct.standard = makeBundleStandard(this.standard, productBundle!!.unitCount, this.unit)
        bundleProduct.erpCode = this.erpCode
    }
}

@Entity
class OrderableVendorProductBundle(
    product: OrderableVendorProduct,
) {
    // 생략

    @ManyToOne(optional = false)
    @JoinColumn(name = "id", nullable = false, insertable = false, updatable = false)
    val product: OrderableVendorProduct = product
}

@Service
class OrderableVendorProductService {
    // 생략

    @Transactional
    fun update(orderableVendorProductId: UUID, data: OrderableVendorProductUpdateData): OrderableVendorProduct {
        // 1. 품목 조회
        val product = getOrderableVendorProductById(orderableVendorProductId)

        // 생략

        // 2. 품목 데이터 업데이트
        product.update(data)

        // 3. 묶음 품목 조회
        product.productBundle?.let {

            // 4. 묶음 품목이 존재하면 동기화
            product.syncBundle()
        }

        return product
    }
}
```

Entity로는 `품목`과 `묶음 품목`이 존재하고 양방향 연관관계를 가지고 있는 것을 볼 수 있다. 그리고 `OrderableVendorProductService#update` 함수를 보면 ID를 이용하여 `품목을 조회`하고 `품목 데이터를 업데이트`하는데 `묶음 품목이 존재`하면 `묶음 품목을 동기화`해 주는 것을 볼 수 있다.

만약 `품목`에 `묶음 품목`이 존재하면 `update` 함수가 기대했던 대로 동작할까? 예상하셨겠지만 아쉽게도 기대했던 대로 동작하지 않고 `품목`은 수정하기 전 상태 그대로 존재하게 된다. 왜일까?

이유는 Hibernate의 Lazy Loding과 영속 메커니즘 때문이다. `update` 함수가 수행되는 순서대로 차근차근 살펴보겠다.

1. 품목 ID를 이용해서 품목을 조회한다.
2. 조회한 품목을 수정한다.
3. 조회한 품목의 묶음 품목을 조회한다. `묶음 품목`의 `FetchType`이 `Lazy`이기 때문에 호출 시점에 묶음 품목을 조회한다. 이때 묶음 품목의 연관관계인 `품목`도 함께 조회된다.
4. 품목 정보를 동기화한다. `syncBundle` 함수를 보면 묶음 품목의 `품목` 정보를 조회하여 묶음 품목정보의 데이터를 업데이트해 준다는 것을 볼 수 있다.

문제는 3번에서 발생한다. `묶음 품목`에서 조회한 `품목` 정보의 ID는 1번에서 조회한 `품목`의 ID와 동일하다. (양방향 연관관계를 가졌으니까!) 그러다 보니 Hibernate의 영속 메커니즘에 따라 캐시 된 Entity 정보를 조회하게 되고 2번에서 수정한 품목 정보는 3번에서 조회한 품목 정보에 의해 덮어씌워지게 되면서 수정한 데이터가 반영되지 않게 되는 것이다.

눈으로 직접 확인해 보기 위해 로그를 출력해 보았다.

```kotlin
println("======== 11111111")
println("======== ${product.orderableVendorCatalogProduct}")
product.update(data)
println("======== 22222222")
println("======== ${product.orderableVendorCatalogProduct}")
product.productBundle?.let {
    println("======== 33333333")
    println("======== ${product.orderableVendorCatalogProduct}")
    product.syncBundle()
    println("======== 44444444")
    println("======== ${product.orderableVendorCatalogProduct}")
}
```

```
======== 11111111
======== com.spoqa.cart.domain.orderableVendor.OrderableVendorCatalogProduct@9671a783
======== 22222222
======== null
Hibernate: select s1_0.id,s1_0.bundle_product_id,s1_0.created_at,s1_0.unit_count from orderable_vendor_product_bundle s1_0 where s1_0.id=?
Hibernate: select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.id in(?,?)
======== 33333333
======== com.spoqa.cart.domain.orderableVendor.OrderableVendorCatalogProduct@9671a783
======== 44444444
======== com.spoqa.cart.domain.orderableVendor.OrderableVendorCatalogProduct@9671a783
```

2번 로그를 보면 분명 `orderableVendorCatalogProduct`를 `null`로 업데이트하였음에도 불구하고 3번 로그에서 다시 조회됨을 볼 수 있다.


#### Hibernate Flushing

Hibernate와 관련해서 좀 더 단순한 다른 이슈를 더 보겠다. 이번에는 Hibernate의 Flushing 메커니즘과 관련한 이슈이다.

```kotlin
@Entity
class Reconciliation(
    transactionDate: LocalDate,
) {
    @Column(nullable = false, unique = true)
    val transactionDate: LocalDate = transactionDate

    // 생략
}

@Service
class ReconciliationWriter {
    // 생략

    @Transactional
    fun write(results: List<PaymentTransactionComparisonResult>): Reconciliation {
        val transactionDate = getTransactionDate(results.first())

        // 1. 거래일 기준 대사 삭제
        reconciliationRepository.deleteByTransactionDate(transactionDate)

        val reconciliation = createReconciliation(transactionDate, results)

        // 2. 대사 저장
        return reconciliationRepository.save(reconciliation)
    }
}
```

위 코드를 보면 코드상으로는 문제가 없어 보인다. 거래일 기준으로 대사 정보를 삭제하고 해당 거래일 기준으로 대사를 다시 생성한 후 저장한다. 기존 데이터를 삭제하고 새로운 데이터를 생성하기 때문에 거래일이 유일 제약조건이 걸려있어도 문제없어 보인다. 통합테스트에서도 정상적으로 동작한다. 하지만 실제로 서버를 실행해서 기능을 수행해 보면 아래와 같이 오류가 발생한다.

```
58144180 [scheduling-1] ERROR:
ERROR: duplicate key value violates unique constraint "reconciliation_transaction_date_uk"
 Detail: Key (transaction_date)=(2023-02-01) already exists.
```

데이터를 삭제하고 저장했는데 왜 유일키 제약조건 위반 오류가 발생할까? 이유는 Hibernate의 Flushing 메커니즘 때문이다.

Hibernate의 [AbstractFlushingEventListener#performExecutions](https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/event/internal/AbstractFlushingEventListener.html#performExecutions(org.hibernate.event.spi.EventSource)){: target="\_blank" } 의 동작 방식을 보면 아래와 같이 적혀있다.

> Execute all SQL (and second-level cache updates) in a special order so that foreign-key constraints cannot be violated:
> 1. Inserts, in the order they were performed
> 2. Updates
> 3. Deletion of collection elements
> 4. Insertion of collection elements
> 5. Deletes, in the order they were performed

그래서 위에 적힌 `ReconciliationWriter#write`함수는 `2. 대사 저장` 후 `1. 거래일 기준 대사 삭제`를 수행하는 순서로 실행되게 된다. 그래서 저장을 먼저 수행하고 데이터를 삭제하니 유일키 제약조건을 위반하는 것이다.

#### Transactional Event Listener

이번에는 Hibernate가 아닌 SpringFramework에서 제공하는 [TransactionalEventListener](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/event/TransactionalEventListener.html){: target="\_blank" } 로 인해 발생했던 이슈를 살펴보겠다.

아래 코드는 유통사 계정을 생성하는 Facade의 구현 코드이다.

```kotlin
@Service
class OrderableVendorAccountFacade {
    // 생략

    @Transactional
    fun createAccount(data: CreateOrderableVendorAccountFacadeData): OrderableVendorAccount {
        // 생략

        // 1. 유통사 계정을 생성
        val createdAccount = orderableVendorAccountService.createAccount(createOrderableVendorAccountData)

        // 2. 샌드버드 사용자를 생성
        val response = chatClient.createOrderableVendorAccountChatUser(createdAccount)

        // 3. 샌드버드 사용자 ID를 생성한 유통사 계정에 업데이트
        orderableVendorAccountService.updateSendbirdUserId(
            orderableVendorAccountId = createdAccount.id,
            newSendbirdId = response.userId.toString(),
        )

        return createdAccount
    }
}

@Service
class OrderableVendorAccountService {
    // 생략

    @Transactional
    fun createAccount(data: CreateOrderableVendorAccountData): OrderableVendorAccount {
        // 생략

        // 1.1. 유통사 계정을 생성
        return orderableVendorAccountRepository.save(account)
            .also {
                // 1.2. 유통사 계정 생성 이벤트를 발행
                eventPublisher.publishEvent(OrderableVendorAccountCreatedEvent(account.id))
            }
    }
}

@Component
class ChatEventHandler {
    // 생략

    @TransactionalEventListener
    fun handle(event: OrderableVendorAccountCreatedEvent) {
        // 생략

        // 1.2.1. 유통사 계정 생성 이벤트를 받아 유통사의 모든 채팅방에 계정을 초대
        chatClient.inviteOrderableVendorAllChannels(account)
    }

    
}

@Component
class ChatClient {
    // 생략

    fun inviteOrderableVendorAllChannels(account: OrderableVendorAccount) {
        account.orderableVendor.orderChannels.forEach {
            // 1.2.1.1. 모든 주문채널에 계정을 초대
            queueMessageSender.inviteSendbirdChannelUser(
                SendbirdInviteChannelUserQueuePayload(
                    channelUrl = it,
                    payload = SendbirdGroupChannelInvitePayload(
                        userIds = listOf(account.sendbirdId!!),
                    ),
                ),
            )
        }

        queueMessageSender.inviteSendbirdChannelUser(
            // 1.2.1.2. 문의 채널에 계정을 초대
            SendbirdInviteChannelUserQueuePayload(
                channelUrl = "INQUIRY_${account.orderableVendor.id}",
                payload = SendbirdGroupChannelInvitePayload(
                    userIds = listOf(account.sendbirdId!!),
                ),
            ),
        )
    }
}
```

클래스가 분리되어 있다 보니 코드의 실행 순서를 따라가기 힘들 거라 생각되어 위에 적힌 계정 생성 기능의 코드 순서를 아래와 같이 정리해 보았다.

1. 유통사 계정을 생성 (1.)
2. 유통사 계정을 생성 (1.1.)
3. 유통사 계정 생성 이벤트를 발행 (1.2.)
4. 유통사 계정 생성 이벤트를 받아 유통사의 모든 채팅방에 계정을 초대 (1.2.1.)
5. 모든 주문 채널에 계정을 초대 (1.2.1.1.)
6. 문의 채널에 계정을 초대 (1.2.1.2.)
7. 샌드버드 사용자를 생성 (2.)
8. 샌드버드 사용자 ID를 생성한 유통사 계정에 업데이트 (3.)

한날 어느 개발자가 유통사 계정을 생성하는 코드를 리펙터링하는 도중 `ChatEventHandler#handle` 함수에 기재된 `@TransactionalEventListener`를 모종의 이유로 `@EventListener`로 변경하게 되었다. 코드를 변경하고 나니 5~6번 항목인 채팅방에 계정을 초대하는 부분에서 오류가 발생하였다.

이유를 살펴보니 아래와 같이 사용자 계정의 `sendbirdId`가 null 값이라 발생한 오류였다.

```kotlin
SendbirdGroupChannelInvitePayload(
    userIds = listOf(account.sendbirdId!!), // <--- NullPointerException 발생
)
```

구현 코드를 변경한 게 아니고 단순히 `@TransactionalEventListener`에서 `@EventListener`로 변경했음에도 불구하고 왜 이런 오류가 발생하였을까?

코드의 구현 순서가 아닌 실제 동작하는 순서를 다시 한번 적어보겠다.

1. 유통사 계정을 생성 (1.)
2. 유통사 계정을 생성 (1.1.)
3. 유통사 계정 생성 이벤트를 발행 (1.2.)
4. 샌드버드 사용자를 생성 (2.)
5. 샌드버드 사용자 ID를 생성한 유통사 계정에 업데이트 (3.)
6. 유통사 계정 생성 이벤트를 받아 유통사의 모든 채팅방에 계정을 초대 (1.2.1.)
7. 모든 주문채널에 계정을 초대 (1.2.1.1.)
8. 문의 채널에 계정을 초대 (1.2.1.2.)

앞서 소개한 순서와 조금 다른 부분을 볼 수 있다. 3번 항목인 `유통사 계정을 생성한 이벤트를 발행`한 후 곧바로 이벤트를 소비하는 것이 아닌 7번 항목이었던 `샌드버드 사용자 생성`과 8번 항목이었던 `샌드버드 사용자 ID를 생성한 유통사 계정에 업데이트`하는 항목을 먼저 실행한 후 유통사 계정 생성 이벤트를 소비한다는 것을 알 수 있다.

원인은 바로 `@TransactionalEventListener`의 동작 방식 때문이다. [TransactionalEventListener 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/event/TransactionalEventListener.html){: target="\_blank" } 를 보면 아래와 같은 내용을 볼 수 있다.

> ... If a transaction is running, the event is handled according to its TransactionPhase.

즉 이벤트 리스너의 함수에 `@TransactionalEventListener`를 정의하면 해당 리스너의 함수는 정해진 Transaction 단계에 따라 수행되며 위 예시 코드에서는 `OrderableVendorAccountFacade#createAccount` 함수에 `@Transactional`이 선언되어 있으므로 `createAccount` 함수가 끝난 시점에 `ChatEventHandler#handle`함수가 실행되는 것이다.

그래서 `ChatEventHandler#handle` 함수의 어노테이션을 `@EventListener`로 바꿔주면 `샌드버드 사용자를 생성`하고 `샌드버드 사용자 ID를 생성한 유통사 계정에 업데이트`하는 로직이 실행되기 전에 이벤트가 소비되므로 유통사 계정의 샌드버드 ID를 조회하는 `account.sendbirdId!!` 코드에서 `NullPointerException`이 발생하게 되는 것이다.

### 원인

통합테스트에서 검출되지 못한 버그로 인한 이슈는 위에서 소개한 사례 말고도 많이 있다. 대부분 Hiberante + Transaction 또는 Client의 Mocking으로 인해 발생한 이슈들로 모을 수 있었다.

통합테스트를 위해 우리는 테스트 간의 데이터를 손쉽게 격리하고 Mock Bean들을 원활하게 생성하기 위해 아래와 같이 테스트를 위한 서버를 `MOCK` 모드로 실행하고 Transaction 내에서 실행되도록 설정해 두었다.

```kotlin
@SpringBootTest
@Import(
    TestDatabaseConfiguration::class,
    Fixture::class,
    ApplicationEventPublisherSpyConfiguration::class,
)
@Transactional
@AutoConfigureMockMvc
abstract class IntegrationTestBase : BehaviorSpec() {
    // 생략

    @MockkBean
    protected lateinit var s3BucketClient: S3BucketClient

    @MockkBean
    protected lateinit var chatClient: ChatClient
}
```

그러다 보니 Transaction 밖에서의 Hibernate의 실행 동작을 테스트 환경에서 온전히 재현하기 어려웠다. 그리고 모든 모듈에 대한 단위테스트가 되어있다 보니 통합테스트를 좀 더 편하게 하기 위해 API들을 추상화한 Client 클래스들을 Mocking 하였다. 이로 인해 위 사례와 같이 실제 동작에서 발견할 수 있는 버그를 발견하지 못하는 사례가 생기게 된 것이다.

그래서 기능 테스트에서는 더 이상 `MOCK` 모드로 실행하는 것이 아닌 내장된 서버를 이용하여 테스트를 실행할 수 있도록 하고 MockBean 들을 모두 제거하여 테스트를 수행할 수 있도록 아래와 같이 베이스 클래스를 설정하였다.

```kotlin
@Import(FunctionalTestConfig::class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
abstract class FunctionalTestBase : FunSpec() {
    override fun extensions() = listOf(SpringExtension)

    private lateinit var mockServer: ClientAndServer

    override suspend fun beforeSpec(spec: Spec) {
        val configuration = Configuration.configuration().logLevel(Level.WARN)
        mockServer = ClientAndServer.startClientAndServer(
            configuration,
            listOf(
                SENDBIRD_API_PORT,
                SLACK_API_PORT,
                DATA_GOV_API_PORT,
                NCLOUD_SENS_API_PORT,
                NICEPAY_WEB_API_PORT,
                NICEPAY_DATA_API_PORT,
                KAKAO_API_PORT,
            ),
        )
    }

    override suspend fun afterSpec(spec: Spec) {
        mockServer.stop()
    }

    // 생략
}
```

## 어떻게 전환 작업을 하였나?

앞선 사례를 통해 통합테스트 환경에서 발견하지 못한 버그들을 살펴보았다. 그럼 이제 본격적으로 기능 테스트로의 전환 이야기로 넘어가 보겠다. 테스트 코드를 보여주는 것은 통합 테스트에서나 기능 테스트에서나 큰 틀에서는 차이가 없을 것이므로 어떠한 전략과 방법을 사용하여 기능 테스트를 수행하였는지 이야기해 보는 게 좋을 것 같다.

### 기능 테스트 전환 가이드

언어 전환 프로젝트 때와 유사하게 기능 테스트로의 전환 작업은 긴 호흡을 가지고 진행해야 할 프로젝트였다. 더욱이 언어 전환 프로젝트와 같이 제품 개발 프로젝트를 멈추고 진행하는 방식이 아니었기에 제품의 신규 개발 프로젝트와 병행해서 진행해야 했고 우선순위에 의해 일정이 종종 미뤄질 수 있었기에 일정을 정하기도 어려웠다. 또한 구성원들이 테스트에 대한 이해도가 높아졌다고 하더라도 사람마다 이해도가 다르고 생소한 테스트 방식에 대한 어려움이 있을 수 있다고 생각했다.

그래서 프로젝트 기간이 길어지더라도 기능테스트에 대한 작업 방법을 가이드하고 코드의 일관성을 유지하기 위해서 가이드 문서를 작성하여 구성원들에게 공유하였다.

![functional-testing-guide](/assets/images/posts/functional-testing-converting-story/functional-testing-guide.png)

### 테스트 케이스

아래 그림은 [이펙티브 소프트웨어 테스팅](https://product.kyobobook.co.kr/detail/S000201055864){: target="\_blank" } 에서 소개된 테스트 피라미드이다. 피라미드 상단으로 올라갈수록 복잡도는 올라가지만, 현실성이 높아짐을 볼 수 있다.

![test-pyramid](/assets/images/posts/functional-testing-converting-story/test-pyramid.jpg)

서버 언어를 전환하며 채택했던 테스트 전략에서와 같이 저희는 복잡한 비즈니스 요구사항에 대한 다양한 케이스들은 단위 테스트에서 모두 다루고 주요하거나 단위 테스트에서 발견하기 힘들다고 판단되는 케이스에 대해서 기능테스트를 작성하는 방식을 도입하였다.

즉, 보다 단순하게 테스트할 수 있는 단위 테스트에서 대부분의 비즈니스 로직을 테스트하고 실제 환경과 가장 유사하지만 테스트하기에 복잡한 기능 테스트에서는 전체적인 기능이 잘 수행되는지 혹은 단위 테스트만으로 불안하다고 판단되는 부분을 확인하기 위한 테스트 코드를 작성하였다.

아래는 정산 데이터를 생성하는 기능에 대한 단위 테스트와 기능 테스트의 테스트 케이스이다. 단위 테스트는 각 모듈별로 여러 테스트 케이스가 존재하는 것을 볼 수 있지만 기능 테스트는 주요 기능에 대한 테스트 케이스만 존재하는 것을 볼 수 있다.

#### 정산 데이터 생성 단위 테스트들 중 테스트 케이스 일부

![ettlement-unit-testing](/assets/images/posts/functional-testing-converting-story/settlement-unit-testing.png)

![calculator-unit-testing](/assets/images/posts/functional-testing-converting-story/calculator-unit-testing.png)

#### 주문서 생성 기능 테스트 케이스

![settlement-functional-testing](/assets/images/posts/functional-testing-converting-story/settlement-functional-testing.png)


### MockServer

통합 테스트에서 기능 테스트로 전환하는 여러 이유 중 하나가 바로 Client를 Mocking 함으로써 발견하지 못한 버그의 존재였다. 그래서 기능 테스트에서는 최대한 외부 API를 그대로 사용하고자 하였다. AWS와 같이 우리가 관리하는 외부 API는 테스트를 위한 인프라를 구성해 둘 수 있었지만 그렇지 못한 외부 API도 존재하였다.

그렇다고 이전과 같이 Client를 그대로 Mocking 하는 것은 좋지 않다고 생각했다. 그래서 외부 API를 최대한 유사하게 사용하는 환경을 구성할 수 있는 [MockServer](https://www.mock-server.com/){: target="\_blank" } 를 활용하기로 하였다. 기능 테스트에서 MockServer를 사용하는 것에 대해 다소 논란이 있을 수 있겠지만 우리는 MockServer를 사용하게 되면서 아래와 같이 실제 외부 API를 사용할 때 발생할 수 있는 문제점을 해결할 수 있다는 부분이 더 매력적으로 다가와 MockServer를 채택하게 되었다.

- 만약 외부 API가 다운된다면 테스트를 할 수 없음
- 외부 리소스를 생성하거나 수정하는 경우 외부 API로 검증하지 못하는 상황이 있음
- 외부 API에 데이터를 전달하기 위한 사전 조건이 너무 방대한 경우 혹은 불가능한 경우
- 외부 API에 테스트 데이터를 함부로 넣으면 안 되는 경우

사실 MockServer를 사용함으로써 [블랙박스 테스트](https://en.wikipedia.org/wiki/Black-box_testing){: target="\_blank" } 의 장점을 많이 상쇄시킨다는 부분이 마음에 걸렸다. 하지만 여러 고민을 해본 결과 블랙박스 테스트의 장점을 상쇄시키는 것이 외부 API를 복잡하게 사용함으로써 기능 테스트 코드 작성에 어려움을 겪는 것보다 낫다고 판단해서 결국 MockServer를 사용하기로 하였다. 다만, 테스트 코드에서 Mocking 부분을 최대한 추상화된 함수로 사용함으로써 테스트 코드를 복잡하지 않게 사용하도록 노력하였다.

한편, 왜 많은 Mock 서버 라이브러리 중 MockServer를 선택하였는지 궁금할 수 있겠다는 생각이 들었는데, MockServer를 선택한 이유는 단순히 우리가 이미 사용 중인 테스트 프레임워크인 [Kotest에서 확장 기능](https://kotest.io/docs/extensions/mockserver.html){: target="\_blank" } 을 제공해 주었기 때문이다. 또한 우리는 Kotest의 가이드 문서와 같이 코드를 작성하지는 않고 최대한 기능 테스트 코드에서 Mocking에 대한 내용을 숨기기 위해 Base 클래스에서 `MockServer` 및 `MockClient`를 생성하고 Helper 클래스를 통해 Mocking 코드를 최대한 추상화하여 사용하였다.

```kotlin
@Import(FunctionalTestConfig::class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
abstract class FunctionalTestBase : FunSpec() {
    override fun extensions() = listOf(SpringExtension)

    private lateinit var mockServer: ClientAndServer

    override suspend fun beforeSpec(spec: Spec) {
        val configuration = Configuration.configuration().logLevel(Level.WARN)
        mockServer = ClientAndServer.startClientAndServer(
            configuration,
            listOf(
                SENDBIRD_API_PORT,
                SLACK_API_PORT,
                NCLOUD_SENS_API_PORT,
                // 생략
            ),
        )
    }

    // 생략
}

@TestConfiguration
class FunctionalTestConfig {
    // 생략

    @Bean
    fun sendbirdApi() = MockServerClient("localhost", SENDBIRD_API_PORT)

    @Bean
    fun slackApi() = MockServerClient("localhost", SLACK_API_PORT)

    @Bean
    fun ncloudSensApi() = MockServerClient("localhost", NCLOUD_SENS_API_PORT)

    @Bean
    fun mockery(
        sendbirdApi: MockServerClient,
        slackApi: MockServerClient,
        ncloudSensApi: MockServerClient,
        // 생략
    ): Mockery {
        return Mockery(
            sendbirdApi,
            slackApi,
            ncloudSensApi,
            // 생략
        )
    }
}

fun Mockery.createSendbirdUser() {
    sendbirdApi.whenWithDefault(
        HttpRequest.request()
            .withMethod(HttpMethod.POST.name())
            .withPath("/v3/users"),
    ).respond(
        template(
            HttpTemplate.TemplateType.MUSTACHE,
            """
            {
                "statusCode": 200,
                "body": {
                    "user_id": {% raw %}"{{#jsonPath}}$.user_id{{/jsonPath}}{{jsonPathResult}}"{% endraw %}
                }
            }
            """.trimIndent(),
        ),
    )
}

fun Mockery.verifyCreateSendbirdUser(userId: String) {
    sendbirdApi.verify(
        HttpRequest.request()
            .withMethod(HttpMethod.POST.name())
            .withPath("/v3/users")
            .withBody(
                json(
                    """
                    {
                      "user_id": "$userId"
                    }
                    """.trimIndent(),
                    MatchType.ONLY_MATCHING_FIELDS,
                ),
            ),
    )
}


test("매장과 유통사를 연결하면 샌드버드 계정이 생성된다.") {
    // Given
    // 생략

    mockery.createSendbirdUser()

    // When
    val actual = clientBuilder.token(testHelper.점주_토큰_생성(manager.id)).build()
        .executeQuery(mutation, variables)
        .extractValueAsObject(
            "connectOrderableVendor",
            typeRef<ConnectedOrderableVendor>(),
        )

    // Then
    val actualStore = testHelper.단일_매장_조회(store.id)
    actualStore.managersSendbirdIds shouldHaveSize 1

    val managerSendbirdId = actualStore.managersSendbirdIds.first()
    eventually(duration = 5.seconds) {
        mockery.verifyCreateSendbirdUser(managerSendbirdId)
    }

    // 생략
}
```

### Test Helper 클래스

앞에서도 언급하였지만, 테스트를 작성하다 보면 기능을 수행하기 위한 값이나 상태를 만들기 위한 코드들이 필요하다. 특히 기능 테스트에서는 단위 테스트에 비해 준비 코드들이 상당히 필요할 수 있다. 이러한 코드들을 모든 테스트에 하나하나 작성해 두기보다 Helper 클래스를 만들어서 사용하면 반복적인 코드를 상당히 줄일 수 있다. 그리고 의미 있는 함수명을 사용한다면 좀 더 읽기 쉬운 테스트 코드를 작성할 수 있기도 하다.

아래 코드를 보면 `testHelper`, `mockery`라는 변수로 사용되는 모듈을 볼 수 있을 것이다. 해당 모듈이 테스트를 위한 데이터를 생성해 주거나 검증을 위한 데이터를 가져오는 역할을 수행해준다.

```kotlin
test("유통사 토큰으로 대량 메시지를 발송을 호출하면 대량 메시지 발송 이력이 저장되고, 메시지가 발송된다.") {
    // Given
    val orderableVendor = testHelper.주문_가능한_거래처_생성()
    val orderableVendorAccount = testHelper.주문_가능한_거래처_계정_생성(orderableVendor.id)
    val store = testHelper.매장_생성()
    testHelper.점주_관리_매장_추가(store.id)
    testHelper.매장_주문_가능_유통사_연결(store.id, orderableVendor.id)

    val input = SendBulkChatInput(
        content = "content",
        imageUrl = "imageUrl",
        storeIds = listOf(store.id),
    )
    val variables = mapOf("input" to input)

    // 생략

    // When
    val actual = clientBuilder.token(testHelper.주문_가능한_거래처_토큰_생성(orderableVendorAccount.id)).build()
        .executeQuery(mutation, variables)
        .extractValueAsObject("sendBulkChat", typeRef<SendBulkChat>())

    // Then
    assertSoftly(actual.history!!) {
        it.content shouldBe "content"
        it.imageUrl shouldBe "imageUrl"
    }

    val expectedStore = testHelper.단일_매장_조회(store.id)
    val expectedGroupChannelUrl = expectedStore.getOrderChannelUrl(orderableVendor.id)

    eventually(5.seconds) {
        mockery.verifySendUserMessageToGroupChannel(
            groupChannelUrl = expectedGroupChannelUrl,
            message = "content",
            customType = "ORDERABLE_VENDOR_ANNOUNCEMENT",
            sendbirdId = orderableVendorAccount.sendbirdId!!,
        )
    }

    // 생략
}
```

만약 Helper 클래스가 없다면 위에서 사용한 `주문_가능한_거래처_생성`함수와 같은 코드를 매번 작성해 주어야 해 상당한 코드 중복이 발생할 것이다.

```kotlin
fun 주문_가능한_거래처_생성(
    // 생략
): OrderableVendor {
    val facade = context.getBean(OrderableVendorFacade::class.java)

    val data = OrderableVendorCreationData(
        name = name,
        deliverableDayOfWeek = deliverableDayOfWeek,
        erpConfiguration = ErpConfigurationCreateData(
            erpType = ErpType.NOT_SUPPORTED,
            billPaymentConfigurationCreateData = BillPaymentConfigurationCreateData(
                usable = false,
                accountNumber = null,
                bank = null,
                accountHolder = null,
                virtualAccountBank = null,
                depositedMethod = DepositMethod.BY_ORDERABLE_VENDOR,
                feeRules = listOf(),
            ),
            ecountErpConfigurationCreateData = EcountErpConfigurationCreateData(
                usable = false,
                companyCode = null,
                userId = null,
                apiCertKey = null,
                warehouseCode = null,
            ),
        ),
        businessInfo = OrderableVendorBusinessInfo(
            regNum = regNum,
            businessName = businessName,
            businessType = businessType,
            businessCondition = businessCondition,
            businessAddress = businessAddress,
            representative = representative,
            email = email,
            storageAddress = storageAddress,
            establishmentDate = establishmentDate,
        ),
        officials = officials,
        authenticationAttachments = authenticationAttachments,
        productInfo = OrderableVendorProductInfo(
            majorTradeStoreCategories = majorTradeStoreCategories,
            majorProductCategories = majorProductCategories,
            mainProducts = mainProducts,
        ),
        deliveryInfo = OrderableVendorDeliveryInfo(
            nextDayDeliveryDeadline = nextDayDeliveryDeadline,
            regions = deliveryRegions,
            preferredRegions = preferredDeliveryRegions,
            methods = deliveryMethods,
            deliveryTimeRange = deliveryTimeRange,
        ),
        paymentInfo = OrderableVendorPaymentInfo(
            availableMethods = availablePaymentMethods,
            intervals = paymentIntervals,
            minimumOrder = minimumOrder,
        ),
        inquiryInfo = OrderableVendorInquiryInfo(
            chatInquirable = chatInquirable,
            newStoreExtendable = newStoreExtendable,
            inqurableTime = inqurableTime,
            inqurableDayOfWeek = inqurableDayOfWeek,
        ),
        memo = memo,
        orderChannelWelcomeMessage = orderChannelWelcomeMessage,
        matchingEnabled = matchingEnabled,
    )
    return facade.createOrderableVendor(data)
}
```

### 테스트를 위한 Open EntityManager in View

기능 테스트를 수행하다 보면 리소스를 생성하는 API를 수행한 후 주어진 데이터로 리소스가 잘 생성되었는지 검증하기 위해 단언(Assertion) 시 데이터를 조회하게 된다. 조회 API가 구현되어 있다면 해당 API를 사용하면 가장 이상적이겠지만 조회 API가 구현되어 있지 않는 경우에는 어쩔 수 없이 Service나 Repository를 이용하여 Entity를 조회해야 하는 경우가 발생한다.

이때 검증을 위해 조회한 Entity의 연관관계를 조회하면 아래와 같은 오류를 만나게 되는 경우가 있다. (Hibernate에 한정된 이슈이다)

```kotlin
test("재무 담당자 권한과 거래일이 주어지면 대사 정보를 생성한다.") {
    // Given
    // 사전 데이터 생성

    // When
    clientBuilder.token(testHelper.재무_관리자_토큰_생성()).build()
        .executeQuery(mutation, variables)
        .extractValueAsObject(
            "reconcile",
            typeRef<Reconcile>(),
        )

    // Then
    val expected = testHelper.단일_대사_조회(transactionDate)
    assertSoftly(expected) {
        it.transactionDate shouldBe transactionDate
        it.estimatedSettlementDate shouldBe LocalDate.of(2023, 3, 7)
        it.state shouldBe ReconciliationState.SUCCESS
        it.totalTransactionCount shouldBe 2           // <------ 연관관계 조회 시 오류 발생
        it.preVendorSettlements shouldHaveSize 1
        it.totalTransactionAmount shouldBe 25000.toBigDecimal()
    }
}
```

```
failed to lazily initialize a collection of role: com.spoqa.cart.domain.reconciliation.Reconciliation._transactions: could not initialize proxy - no Session
org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: com.spoqa.cart.domain.reconciliation.Reconciliation._transactions: could not initialize proxy - no Session
	at org.hibernate.collection.spi.AbstractPersistentCollection.throwLazyInitializationException(AbstractPersistentCollection.java:635)
	at org.hibernate.collection.spi.AbstractPersistentCollection.withTemporarySessionIfNeeded(AbstractPersistentCollection.java:218)
	at org.hibernate.collection.spi.AbstractPersistentCollection.readSize(AbstractPersistentCollection.java:148)
	at org.hibernate.collection.spi.PersistentBag.size(PersistentBag.java:350)
```

이유는 Transaction 밖에서 Lazy 하게 연관관계를 조회하려고 하면서 발생한 오류이다. (좀 더 자세히 알고 싶다면 [Hibernate could not initialize proxy – no Session](https://www.baeldung.com/hibernate-initialize-proxy-exception){: target="\_blank" } 글을 보길 바란다) 테스트로 인해서 구현된 운영 코드를 바꿀 수 없으므로 테스트 코드에서 무언가 조치를 해야 할 필요가 있었다.

해당 이슈에 대한 해결 방법으로 생각해 낸 것이 바로 OSIV(Open Session in View)로 알려진 Spring의 [Open EntityManager in View](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#data.sql.jpa-and-spring-data.open-entity-manager-in-view){: target="\_blank" } 이다.

Spring으로 Web Application을 개발하는 개발자라면 OSIV(Open Sesison in View)에 대해 잘 알 것으로 생각한다. OSIV 패턴은 Spring MVC에서 `OpenEntityManagerInViewInterceptor`에 의해 적용되어야 한다. 해당 클래스를 참고해서 아래와 같이 `TestHelper` 클래스에서 Entity를 조회한 후 연관관계를 사용할 때 오류가 발생하지 않도록 조치하였다.

```kotlin
@Target(AnnotationTarget.CLASS)
annotation class OpenEntityManager

@Aspect
class OpenEntityManagerAspect : EntityManagerFactoryAccessor() {
    @Around("@within(com.spoqa.cart.fixture.OpenEntityManager).*(..)")
    fun openEntityManager(pjp: ProceedingJoinPoint): Any? {
        logger.debug { "Opening JPA EntityManager" }

        val emf: EntityManagerFactory = obtainEntityManagerFactory()

        try {
            val em: EntityManager = createEntityManager()
            val emHolder = EntityManagerHolder(em)
            TransactionSynchronizationManager.bindResource(emf, emHolder)
        } catch (ex: PersistenceException) {
            throw DataAccessResourceFailureException("Could not create JPA EntityManager", ex)
        }

        try {
            val result = pjp.proceed()

            if (result is PrimaryKeyEntity) {
                result.loadAssociations()
            }

            return result
        } finally {
            val emHolder = TransactionSynchronizationManager.unbindResource(emf) as EntityManagerHolder

            logger.debug { "Closing JPA EntityManager" }

            EntityManagerFactoryUtils.closeEntityManager(emHolder.entityManager)
        }
    }
}

private fun PrimaryKeyEntity.loadAssociations() {
    this::class.declaredMemberProperties.forEach {
        try {
            it.getter.call(this).toString()
        } catch (_: Exception) {
        }
    }
}

@TestComponent
@OpenEntityManager
class TestHelper {
    // 생략

    fun 단일_대사_조회(transactionDate: LocalDate): Reconciliation {
        val facade = context.getBean(ReconciliationFacade::class.java)
        return facade.reconcile(transactionDate)
    }
}
```

`OpenEntityManagerAspect `클래스를 보면 Spring의 AOP를 사용해서 `@OpenEntityManager` 어노테이션이 선언된 클래스의 모든 함수에 OSIV를 적용한다는 것을 볼 수 있다. 특히 `loadAssociations` 함수를 보면 조회한 Entity의 모든 연관관계를 조회한다는 것을 볼 수 있다. 그 이유는 테스트를 수행하는 함수에서는 Transaction이 실행 중이지 않기 때문에 `TestHelper`의 함수에서 모든 연관관계를 먼저 조회하여 테스트 함수에서 연관관계 조회 시 `LazyInitializationException`이 발생하지 않도록 하기 위함이었다.

이렇게 조치하면 `@Transactional`이 선언되어 있지 않은 테스트 코드에서도 운영 코드를 변경하지 않고 매번 Helper 클래스에서 연관관계를 명시적으로 조회하지 않고도 Entity의 연관관계를 손쉽게 조회할 수 있어 테스트 코드를 좀 더 손쉽게 작성할 수 있다는 장점이 있다.

### 기능 테스트에서의 단언(Assertion)

통합 테스트에서 기능 테스트로 전환 시 가장 걱정했던 부분이 바로 테스트 케이스 간의 데이터 공유로 인한 간섭이었다. 테스트 코드를 작성하는 데 하나의 테스트 케이스가 다른 테스트 케이스에 영향을 받지 않도록 하는 것이 이상적이지만 CI 환경에서 효율적이고 빠르게 테스트를 수행하기 위해서는 어쩔 수 없이 데이터베이스나 Message Queue와 같은 자원들은 공유할 수밖에 없었다.

그러다 보니 통합 테스트에서는 Transaction을 테스트마다 실행시켜서 테스트 종료 후 Rollback 하는 형태로 테스트간 간섭을 회피하였다. 기능 테스트에서는 테스트 케이스 함수에서 Transaction 사용으로 인한 문제점을 해결하려 하였기 때문에 통합 테스트의 방식을 사용할 수 없었다. 매 테스트 코드가 실행될 때마다 데이터베이스를 초기화해 주는 스크립트를 실행시켜 보자는 의견도 나왔었지만, 테스트가 실행될 때 준비 시간이 너무 늘어나는 이슈로 인해 해당 방법도 사용할 수 없었다.

결국, 기능 테스트에서는 데이터베이스나 Message Queue를 공유하되 각 테스트 케이스의 단언을 아래와 같이 다른 테스트 케이스에 영향을 받지 않게끔 작성하도록 하였다.

#### 통합 테스트 단언 예시

통합 테스트에서는 각 테스트마다 데이터가 격리되기 때문에 주문서를 생성한 후 전체 주문서를 조회해도 기대하는 주문서를 이용해서 단언을 수행할 수 있다.

```kotlin
test("유통사를 생성한다.") {
    // Given
    val orderableVendor = testHelper.주문_가능한_거래처_생성()
    
    // 생략

    // When
    val actual = clientBuilder.token(token).build()
            .executeQuery(mutation, variables)
            .extractValueAsObject("createOrderSheet.orderSheet", typeRef<OrderSheetField>())

    // Then
    val expected = testHelper.전체_주문서_조회()
    expected shouldHaveSize 1
    expected.first().id shouldBe actual.id
}
```

#### 기능 테스트 단언 예시

기능 테스트에서는 테스트마다 데이터가 격리되지 않기 때문에 각 테스트 케이스에서 생성한 유통사를 이용하여 기대하는 주문서를 조회한 후 단언을 수행하도록 하여 테스트의 거짓양성이 발생하지 않도록 하였다.

```kotlin
test("유통사를 생성한다.") {
    // Given
    val orderableVendor = testHelper.주문_가능한_거래처_생성()
    
    // 생략

    // When
    val actual = clientBuilder.token(token).build()
            .executeQuery(mutation, variables)
            .extractValueAsObject("createOrderSheet.orderSheet", typeRef<OrderSheetField>())

    // Then
    val expected = testHelper.유통사의_주문서_목록_조회(orderableVendor.id)
    expected shouldHaveSize 1
    expected.first().id shouldBe actual.id
}
```

### 비동기 코드 검증

테스트의 단언(Assertion)의 연장선으로 통합 테스트에서 기능 테스트로 전환 시 고민했던 부분이 비동기 코드의 검증이었다. 백엔드에서는 주문서 생성 시 슬렉 메시지 전송과 같은 주요 로직이 아닌 부가적인 로직을 처리할 때나 처리 효율성을 이유로 비동기적으로 서버 요청을 처리해야 할 때 `@Async`를 활용하고 있다. (`@Async`와 관련한 자세한 내용은 [Creating Asynchronous Methods](https://spring.io/guides/gs/async-method/){: target="\_blank" } 글을 참고하길 바란다)

통합 테스트에서는 Mocking을 이용하여 호출 여부만 판단하는 형태로 테스트 코드를 작성했었다. 그러다 보니 이벤트 처리에 대한 대부분의 코드가 대부분 단위테스트로만 검증되고 있었다. 그래서 비동기 코드가 최종적으로 어떻게 통합되어 수행되는지 검증하지 못한다는 단점이 있었다. 그래서 기능 테스트로 전환하면서 비동기적인 코드가 끝까지 실행되는지를 테스트하여 해당 기능에 대한 안정성을 좀 더 높이고자 하였다.

하지만 비동기적으로 실행되는 코드를 검증하는 것은 동기적으로 실행되는 코드보다 복잡할 수 있다. 우리는 그래서 아래와 같은 선택지에서 고민하였다.

#### Thread#sleep

다소 아름답지(?) 못한 방식이지만 가장 단순하게 시도해 볼 수 있는 코드이다.

```kotlin
test("주문서를 생성하면 주문 메시지를 발송한다.") {
    // Given
    val input = CreateOrderSheetInput()
    val variables = mapOf("input" to input)

    // When
    val actual = clientBuilder.token(token).build()
            .executeQuery(mutation, variables)
            .extractValueAsObject("createOrderSheet", typeRef<CreateOrderSheet>())

    // Then
    val expected = testHelper.유통사의_주문서_목록_조회(orderableVendor.id)
    expected shouldHaveSize 1
    expected.first().id shouldBe actual.id

    Thread.sleep(1000)
    mockery.verifySendUserMessageToGroupChannel()
}
```

위 코드대로라면 비동기 코드가 언제 실행되든지 간에 테스트는 최소 1초 이상 실행될 것이다. 거기다 만약 이벤트가 1초 이상 걸린다면 테스트는 실패하게 될 것이다. 테스트 코드를 작성하는 중에 제대로 테스트 코드를 작성하고 있는지 확인하기 위해 임시로 코드를 넣어볼 순 있겠지만 `Thread#sleep`을 그대로 사용하는 것은 좋아 보이지 않는다.

#### SyncTaskExecutor

다음 방법으로는 [SyncTaskExecutor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/SyncTaskExecutor.html){: target="\_blank" } 를 활용하는 것이다. `SyncTaskExecutor`는 `@Async`로 선언된 코드를 동기적으로 수행되도록 해준다. 문서에도 쓰여있다시피 주로 테스트를 위해 사용된다.

테스트 설정에서 아래와 같이 코드를 작성하면 사용할 수 있다.

```kotlin
@TestConfiguration
@EnableAsync
class AsyncConfig : AsyncConfigurer {
    override fun getAsyncExecutor(): Executor {
        return SyncTaskExecutor()
    }
}
```

이렇게 설정하면 이제는 더이상 `Thread.sleep(1000)`과 같은 코드를 넣지 않고도 비동기 기능을 검증할 수 있게 된다.

```kotlin
test("주문서를 생성하면 주문 메시지를 발송한다.") {
    // Given
    val input = CreateOrderSheetInput()
    val variables = mapOf("input" to input)

    // When
    val actual = clientBuilder.token(token).build()
            .executeQuery(mutation, variables)
            .extractValueAsObject("createOrderSheet", typeRef<CreateOrderSheet>())

    // Then
    val expected = testHelper.유통사의_주문서_목록_조회(orderableVendor.id)
    expected shouldHaveSize 1
    expected.first().id shouldBe actual.id

    mockery.verifySendUserMessageToGroupChannel()
}
```

하지만 해당 설정에도 문제점이 존재하였다. 우리는 부가적인 로직(예를 들어 주문서 생성 알림을 위한 메시지 전송)에서 발생하는 오류가 주요 로직(예를 들어 주문서 생성)에 영향을 미치지 않았으면 하였다. 부가적인 로직을 비동기 함수로 처리하게 되면 이러한 요구사항을 충족시킬 수 있었다. 그래서 만약 `sendUserMessageToGroupChannel` 함수에서 오류가 발생하더라도 주문서 생성은 문제없이 동작하는 것이다. 그러나 `SyncTaskExecutor`를 사용하면 이러한 요구사항을 충족시키지 못한다. 부가적인 로직에서 발생한 오류가 전파되어 주요 로직에도 영향을 미치기 때문이다.

운영환경과 테스트환경에 차이가 있는 것은 어느 정도 불가피하다지만 이러한 주요 요구사항을 충족하지 못하는 부분은 중대하다고 판단해서 `SyncTaskExecutor`를 사용하지 않기로 하였다.

#### eventually

결국 우리는 Kotest에서 제공하는 [Eventually](https://kotest.io/docs/assertions/eventually.html){: target="\_blank" } 를 사용하기로 하였다. `Eventually`는 실제 환경과 동일하게 테스트 환경을 구성함과 동시에 `Thread.sleep(1000)`을 사용하지 않도록 하는 가장 손쉬운 방법을 제공해 주었다.

`Eventually`는 제한된 시간 내에 기대하는 비동기 코드가 실행되는지 가장 짧은 시간 내에 알려준다. 그래서 `Thread.sleep(1000)`을 사용했을 때처럼 무조건 지정된 시간을 기다리지도 않고 비동기 코드 단언을 위한 복잡한 코드를 작성하지도 않아도 된다.

```kotlin
test("주문서를 생성하면 주문 메시지를 발송한다.") {
    // Given
    val input = CreateOrderSheetInput()
    val variables = mapOf("input" to input)

    // When
    val actual = clientBuilder.token(token).build()
            .executeQuery(mutation, variables)
            .extractValueAsObject("createOrderSheet", typeRef<CreateOrderSheet>())

    // Then
    val expected = testHelper.유통사의_주문서_목록_조회(orderableVendor.id)
    expected shouldHaveSize 1
    expected.first().id shouldBe actual.id
    
    eventually(5.seconds) {
        mockery.verifySendUserMessageToGroupChannel()
    }
}
```

## 전환 시 이슈는 없었나?

앞서 말했지만, 처음부터 기능 테스트로 테스트 코드를 작성하지 않고 통합 테스트로 테스트 코드를 작성할 만큼 기능 테스트에 대한 난이도에 대한 우려가 있었다. 아니나 다를까 기능테스트를 전환하면서 수많은 이슈를 겪게 되었다. 모두 소개하면 좋겠지만 내용이 너무 길어질 수 있으므로 우리가 겪었던 대표적인 이슈들을 소개하고 어떻게 해결하였는지 이야기해 보겠다.

### Flaky tests

우리는 CI(Continuous Integration) 도구로 [CircleCI](https://circleci.com)를 사용한다. CircleCI에서는 Insights라는 기능을 통해 테스트 케이스가 간헐적으로 실패하는 테스트를 알려준다. ([CircleCI 문서의 Flaky tests](https://circleci.com/docs/insights-tests/#flaky-tests){: target="\_blank" } 를 참고하길 바란다)

기능테스트로 전환하면서 CI에서 Flaky tests의 빈도가 증가하였다. 앞서 말한 바와 같이 기능 테스트에서는 테스트간 상태 격리가 되지 않아 개발자의 실수 탓에 간헐적으로 실패가 발생하는 것이었다. `기능 테스트에서의 단언` 부분에서 말했다시피 최대한 테스트 간에 영향을 받지 않게끔 코드를 작성함에도 간헐적인 테스트의 거짓양성이 발생할 수 있는 것은 어쩔 수 없다고 생각한다. 그래서 저희는 완벽하게 간헐적인 테스트 실패를 막으려 하기보다 아래와 같이 CI의 Flaky tests 리포트를 자주 모니터링하면서 최대한 불안정한 테스트를 줄이도록 노력하고 있다.

![flaky-tests-report](/assets/images/posts/functional-testing-converting-story/flaky-tests-report.png)

### MockServer Response Template

테스트 코드를 작성할 때 테스트를 어렵게 하는 요인 중 하나가 바로 현재시간, Random 데이터 생성 등 테스트 대상 내에서 생성하는 무작위 값이 존재할 때이다. 좀 더 테스트하기 쉬운 코드를 작성하기 위해서 단위 테스트에서는 의존주입, 매개변수로 추출 등과 같은 기술을 이용하지만, 기능테스트에서는 이마저도 활용할 수 없는 경우가 많다.

그래서 기능테스트에서는 검증하기 힘든 값들은 이미 단위 테스트에서 잘 검증했다는 가정하에 과감히 생략하기도 한다. 하지만 이마저도 주요 로직에 포함되거나 모듈 간에 통합하는 부분에서 테스트가 필요한 경우 생략하지 못하는 상황이 존재한다. 우리는 MockServer를 활용하면서 해당 이슈들을 겪었었다. 그중 대표적인 사례만 소개하고 넘어가 보겠다.

아래 코드는 매장과 유통사를 연결하는 기능에 대한 테스트이다.

```kotlin
test("관리자 권한으로 유통사와 매장을 연결하면 매장과 유통사가 연결된다.") {
    // Given
    val orderableVendor = testHelper.주문_가능한_거래처_생성()
    val store = testHelper.매장_생성()
    val input = ConnectStoreOrderableVendorInput()
    val variables = mapOf("input" to input)
    
    mockery.getSendbirdUser()
    
    // 생략

    // When
    val actual = clientBuilder.token(testHelper.관리자_토큰_생성()).build()
        .executeQuery(mutation, variables)
        .extractValueAsObject(
            "connectStoreOrderableVendor",
            typeRef<ConnectedStoreOrderableVendor>(),
        )

    // Then

    // 생략

    eventually(5.seconds) {
        mockery.verifyCreateSendbirdUser(createdSendbirdId)
    }
}
```

`Mockery#getSendbirdUser` 함수를 보면 아래와 같이 외부 API를 Mocking하고 있다.

```kotlin
fun Mockery.getSendbirdUser() {
    sendbirdApi.whenWithDefault(
        HttpRequest.request()
            .withMethod(HttpMethod.GET.name)
            .withPath("/v3/users/.+"),
    ).respond(
        HttpResponse.response()
            .withStatusCode(200)
            .withBody(
                json(
                    """
                    {
                      "user_id": "${generateId()}",
                      "nickname": "${generateString()}"
                    }
                    """.trimIndent(),
                ),
            ),
    )
}
```

`user_id`를 랜덤한 값으로 Mocking 하게 되면서 테스트가 실패하거나 검증하지 못하는 상황이 발생하였다. 그래서 우리는 랜덤한 값을 생성하여 전송하더라도 운영 코드 변경 없이 테스트가 잘 수행되도록 할 방법을 모색하기 시작하였다.

첫 번째는 `Mockery#getSendbirdUser` 함수에 매개변수를 추가하는 방법이 논의되었다.

```kotlin
fun Mockery.getSendbirdUser(
    userId: String,
) {
    sendbirdApi.whenWithDefault(
        HttpRequest.request()
            .withMethod(HttpMethod.GET.name)
            .withPath("/v3/users/.+"),
    ).respond(
        HttpResponse.response()
            .withStatusCode(200)
            .withBody(
                json(
                    """
                    {
                      "user_id": "$userId",
                      "nickname": "${generateString()}"
                    }
                    """.trimIndent(),
                ),
            ),
    )
}
```

가장 직관적이고 손쉽게 문제를 해결할 방법 같아 보이지만 API 서버 내부에서 `userId`를 랜덤하게 생성하는 경우에는 해당 값을 매개변수로 전달할 수 없기 때문에 해결 방법으로 사용할 수 없었다.

두 번째 방법으로 [MockServer의 Response Template](https://www.mock-server.com/mock_server/response_templates.html){: target="\_blank" } 을 활용하는 방법을 모색하였다. Response Template에는 여러 가지 포맷들이 있다. 그중 우리는 [Mustache Response Templates](https://www.mock-server.com/mock_server/response_templates.html#mustache_templates){: target="\_blank" } 을 사용하였다.

```kotlin
fun Mockery.getSendbirdUser() {
    sendbirdApi.whenWithDefault(
        HttpRequest.request()
            .withMethod(HttpMethod.GET.name())
            .withPath("/v3/users/{userId}")
            .withPathParameters(
                param("userId", ".+"),
            ),
    ).respond(
        template(
            HttpTemplate.TemplateType.MUSTACHE,
            """
            {
                "statusCode": 200,
                "body": {
                  "user_id": "{% raw %}{{ request.pathParameters.userId.0 }}{% endraw %}",
                  "nickname": "${generateId()}"
                }
            }
            """.trimIndent(),
        ),
    )
}
```

위 코드를 보면 `{% raw %}{{ request.pathParameters.userId.0 }}{% endraw %}`부분이 보인다. 해당 값은 `withPath("/v3/users/{userId}")`의 `userId` 값을 그대로 반환하도록 하는 문법이다. 이를 통해 위에서 작성한 테스트가 실패하지 않고 올바르게 검증되도록 할 수 있었고 결국 우리는 두 번째 방법을 사용하기로 하였다.


### Message Throttling

[청구/수납 서비스 개발기](https://spoqa.github.io/2023/02/24/bill-payment-development-story.html){: target="\_blank" } 에서 Message Throttling과 관련한 이야기를 다루었었다. [Bucket4j](https://github.com/bucket4j/bucket4j){: target="\_blank" } 의 Message Throttling을 사용하기 위해서는 데이터베이스와 Message Queue가 필요하다. 앞서 말한 바와 같이 기능테스트에서는 데이터베이스와 Message Queue를 공유해서 사용하고 있다. 이에 따라 Throttling 된 메시지를 전송할 때 `eventually`를 사용해 검증하지만 메시지 전송 단언(Assertion)이 되지 않는 이슈가 발생하였다.

특이한 점은 단일 테스트 케이스를 실행해서 테스트하는 경우에는 성공하지만, 전체 테스트 케이스를 실행시키면 실패한다는 것이었다. 원인은 바로 Message Throttling이 문제였었다.

하나의 테스트 케이스만 실행하는 경우 Throttling 대기열에 쌓여있는 메시지가 없으므로 `eventually`로 검증 시 제한된 시간 내 잘 검증이 되는 것을 볼 수 있다. 하지만 전체 테스트를 실행하는 경우 Throttling 대기열에 다수의 테스트 케이스 메시지들이 쌓이게 되고 Throttling 되어 소비되기 때문에 `eventually`로 검증을 시도하더라도 제한된 시간 내 단언에 성공하지 못해 테스트가 실패하는 경우가 발생한 것이다. (해당 이슈를 발견하기까지 상당히 애를 먹었다 ^^;;)

생각해 보면 Message Throttling을 구현한 이유가 외부 API의 제약조건 때문이었다. 하지만 이러한 외부요인을 회피하기 위해 우리는 MockServer를 활용하고 있으므로 Message를 Throttling 할 필요가 없다. 그래서 우리는 아래와 같이 Throttling이 걸려있는 Bucket 설정을 모두 Throttling이 걸리지 않도록 변경하여 문제를 해결했다.

#### 운영 환경 설정

```kotlin
@Configuration
class Bucket4jConfig {
    @Bean
    fun bucketProxyManager(dataSource: DataSource): PostgreSQLadvisoryLockBasedProxyManager<Long> {
        return PostgreSQLadvisoryLockBasedProxyManager<Long, Long>(SQLProxyConfiguration.builder().build(dataSource))
    }

    @Bean
    fun sendbirdUserMessageBucket(bucketProxyManager: PostgreSQLadvisoryLockBasedProxyManager<Long>): BucketProxy {
        val key = 1003L
        val bucketConfiguration = BucketConfiguration.builder()
            .addLimit(Bandwidth.simple(5, Duration.ofSeconds(1))) // 1초에 5회로 Throttling
            .build()
        return bucketProxyManager.builder().build(key, bucketConfiguration)
    }

    // 생략
}
```

#### 테스트 환경 설정

```kotlin
@TestConfiguration
class Bucket4jConfig {
    private val unlimited: Bandwidth = Bandwidth.simple(Long.MAX_VALUE, Duration.ofNanos(Long.MAX_VALUE))

    @Bean
    fun bucketProxyManager(dataSource: DataSource): PostgreSQLadvisoryLockBasedProxyManager<Long> {
        return PostgreSQLadvisoryLockBasedProxyManager<Long, Long>(SQLProxyConfiguration.builder().build(dataSource))
    }

    @Bean
    fun sendbirdUserMessageBucket(bucketProxyManager: PostgreSQLadvisoryLockBasedProxyManager<Long>): BucketProxy {
        val key = 1003L
        val bucketConfiguration = BucketConfiguration.builder()
            .addLimit(unlimited) // Throttling 설정하지 않음
            .build()
        return bucketProxyManager.builder().build(key, bucketConfiguration)
    }

    // 생략
}
```


## 마무리

지금까지 기능 테스트로 전환 시 사용했던 여러 가지 방법들을 알아보았다.

상황을 보다 이해가 잘되도록 하려다 보니 다소 세세한 부분까지 다루게 된 것 같다. 기능 테스트를 전환할 때 어떠한 애로사항들이 있는지, 해당 애로사항들을 어떻게 풀어나갈 수 있는지와 같은 넓은 관점에서 바라봐 주면 좋을 것 같다.

모두의 노력 덕분에 우리는 올해 통합 테스트 코드를 모두 제거하고 온전히 기능 테스트로만 전체 기능을 테스트할 수 있게 되었다. 다만, 여전히 기능테스트에 대한 개선할 점들은 많이 있어 보인다 ^^;

![move-integration-testing-pr](/assets/images/posts/functional-testing-converting-story/remove-integration-testing-pr.png)

모쪼록 이번 이야기가 여러분의 테스트 코드 작성에 조금이나마 도움이 되었길 바라며, 기능 테스트를 작성하면서 다시 또 재미있는 이야깃거리가 있다면 소개해 드리는 글로 찾아보도록 하겠다.
