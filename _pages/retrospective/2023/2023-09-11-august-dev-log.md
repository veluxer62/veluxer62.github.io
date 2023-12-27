---
layout: single
title: "2023년 8월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2023-august-dev-log/
---

8월은 제품의 방향성을 바꾸는 큰 프로젝트로 바쁜 한달이 되었다. 이전에 명세표를 통한 비용관리 서비스에서 주문톡 서비스로 변경한 것과 같이 피봇팅을 하는 것이 아니라 주문톡 서비스에 이전에는 제공하지 않는 새로운 시스템을 도입하는 형태로의 변화였다. 제품에 여러가지 기능들이 개발되고 있지만 아직 이렇다할 와우 포인트가 없어서 방향성에 대한 고민들이 많은 시기이다. 이번 프로젝트가 잘 성공해서 연말에 목표를 향해 열심히 달려가는 한해가 되었으면 좋겠다.

<br/>

올해 목표중 하나였던 자전거 국토종주 그랜드슬램 달성을 위해 종주 계획을 짜고 있다. 더운 시기가 지나고 추워지기까지 시간이 그렇게 넉넉하지 않기 때문에 4개의 종주 코스를 다 가보려면 주말마다 부지런히 다녀야할 것 같다. 특히 출발지가 문경, 목포, 대진, 제주도와 같이 지방이기 때문에 버스시간을 맞춰서 일정을 짜려니 하루에 많게는 200KM정도도 달려하는 빡센 일정으로 코스를 짤 수 밖에 없었다. 조금 무리는 하겠지만 그래도 올해안에 꼭 모든 코스를 다 돌아서 그랜드 슬램을 달성해야겠다!

<br/>

아래는 8월동안 정리한 이슈 내용들이다.

## SonarQube 3.x.x 이슈

Gradle 8로 업그레이드 하면 소나큐브 3.x.x 버전이 실행이 되지 않는다. 이슈를 찾아보니 Gradle 8버전 부터는 4.x.x 버전으로 사용해야 한다고 한다.

출처: [https://community.sonarsource.com/t/sonarqube-plugin-for-gradle-8/83936](https://community.sonarsource.com/t/sonarqube-plugin-for-gradle-8/83936){: target="\_blank" }

## QueryDSL nullable 가격 정렬

RDBMS에서 nullable한 숫자형 타입을 정렬하고자 할 때 null인 값들을 마지막에 오도록 해야하는 요구사항이 있을 수 있다. 그런경우 QueryDSL에서 사용할 수 있는 함수가 바로 `nullsLast`이다.

```kotlin
from(foo)
    .orderBy(
        foo.price.asc().nullsLast(),
    )
```

다만 여기에서 만약 가격은 정렬하지 않고 `null` 값만 마지막으로 오도록 정렬하고 싶다면 어떻게 해야할까? 바로 아래와 같이 case 문을 사용해야한다.

```kotlin
from(foo)
    .orderBy(
        promotionPriceOrder(),
    )

private fun promotionPriceOrder(): OrderSpecifier<Int> {
    val cases = CaseBuilder()
        .`when`(foo.price.isNull).then(2)
        .otherwise(1)
    return OrderSpecifier(Order.ASC, cases)
}
```

## 시스템 종료될때 비동기 잡이 Graceful하게 종료되도록 설정하는 방법

스프링에서 `@Async`를 이용하여 비동기 잡을 실행할때 서버를 종료하게되면 어떻게 될까? 만약 Graceful 설정을 해두지 않는다면 해당 작업은 원치않는 타이밍에 종료될 수 있다. 트랜잭션을 설정해두었다면 DB에 대한 원자성은 유지될 수 있을지 모르지만 만약 외부 시스템을 활용함으로 인해 원자성을 유지할 수 없는 경우에는 난처한 상황이 펼쳐질 수 있다. 이러한 상황을 방지하기 위해 아래와 같이 비동기 잡도 Graceful하게 설정할 수 있다.

```kotlin
@Configuration
@EnableAsync
class AsyncConfig : AsyncConfigurer {
    override fun getAsyncExecutor(): Executor {
        return ThreadPoolTaskExecutor().apply {
            corePoolSize = 10
            maxPoolSize = 10
            queueCapacity = 1000
            setWaitForTasksToCompleteOnShutdown(true)
            initialize()
        }
    }
}
```

추가로 만약 queue 사이즈가 모자라면 자동으로 thread를 생성하도록 실패전략을 가져가는 설정을 해줄 수도 있다.

```kotlin
@Configuration
@EnableAsync
class AsyncConfig : AsyncConfigurer {
    override fun getAsyncExecutor(): Executor {
        return ThreadPoolTaskExecutor().apply {
            corePoolSize = 10
            maxPoolSize = 10
            queueCapacity = 1000
            setRejectedExecutionHandler(ThreadPoolExecutor.CallerRunsPolicy())
            setWaitForTasksToCompleteOnShutdown(true)
            initialize()
        }
    }
}
```

## TestConfiguration 일반 상식

`TestConfiguration`은 경로와 파일명이 동일하면 별도로 import를 해주지 않아도 된다. 하지만 개인적으로는 import를 명시적으로 해주는게 좋다고 생각하는데 운영코드의 패키지가 이동하게되었을때 해당 테스트 파일도 기억하고 이동해주지 않으면 원치않게 테스트가 실행되지 않을 수 있다는 점과 패키지와 파일명이 동일해야 한다는 제약조건으로 인해 유연성이 떨어진다는 점 때문이다.

물론 import를 별도로 해줄때의 단점도 있는데, `Bean` 오버라이딩을 해주어야 한다는 점과 운영코드의 설정과 테스트 설정 파일이 일치하지 않아 이슈 추적이 다소 번거롭다는 점이 있다.

각자의 상황에 맞게 잘 선택해서 사용하자.

## 테스트에서의 메시지 스로틀링

센드버드나 슬렉과 같이 API의 호출 제한이 있는 경우 부득이하게 메시지 스로틀링을 구현해야할 수도 있다. 우리는 [bucket4j](https://github.com/bucket4j/bucket4j){: target="\_blank" }를 사용하는데 아래와 같이 스로틀링 설정이 되어있다.

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
            .addLimit(Bandwidth.simple(5, Duration.ofSeconds(1)))
            .build()
        return bucketProxyManager.builder().build(key, bucketConfiguration)
    }
}
```

위와 같이 설정하면 메시지를 1초에 최대 5번까지만 지연발송하도록 할 수 있다.

```kotlin
fun handle(message: SendbirdUpdateUserQueuePayload) {
    updateSendbirdUserBucket.asBlocking().consume(1)

    sendbirdApi.updateUser(
        sendbirdId = message.sendbirdId,
        sendbirdUpdateUserPayload = message.payload,
    )
}
```

이번에 기능테스트로 전환작업하면서 테스트가 원치않게 실패하는 거짓양성을 겪게 되었는데 바로 이러한 메시지 스로틀링으로 인해 발생한 것이었다. 

테스트를 작성하다보면 동일한 함수라도 다수의 테스트 케이스를 작성해야 하는 경우가 많고 단위 테스트가 아닌 기능테스트에서는 사전데이터를 만들기 위해 위의 예시와 같이 메시지 스로틀링이 걸려있는 함수를 여러번 호출해야하는 경우가 종종 있다. 이때 메시지가 스로틀링이 걸려있기 때문에 비동기잡의 경우 원하는 타이밍에 작업이 소비되지 않아 테스트가 실패하는 상황이 발생하게 된다.

그렇다면 이러한 실패를 겪지 않으려면 어떻게 조치하면 좋을까? 테스트 환경에 따라 다르겠지만 만약 여러분이 [MockServer](https://www.mock-server.com/){: target="\_blank" }를 사용하고 있다면 간단하게 해결할 수 있다. 바로 스로틀링을 제거해버리면 된다. 어차피 스로틀링이 필요한 이유는 외부 API의 제약조건이기 때문에 이러한 제약조건이 없는 MockServer에서는 아래와 같이 스로틀링을 설정해주지 않아도 된다.

```kotlin
@Bean
fun sendbirdUserMessageBucket(bucketProxyManager: PostgreSQLadvisoryLockBasedProxyManager<Long>): BucketProxy {
    val key = 1003L
    val bucketConfiguration = BucketConfiguration.builder()
        .addLimit(Bandwidth.simple(Long.MAX_VALUE, Duration.ofNanos(Long.MAX_VALUE)))
        .build()
    return bucketProxyManager.builder().build(key, bucketConfiguration)
}
```

## MockServer Response Template

[MockServer](https://www.mock-server.com/){: target="\_blank" }를 이용하여 테스트할때 응답을 mocking 하는 경우 구현된 코드에 따라 테스트하기 까다로운 경우가 있다. 예를 들어 응답하는 값에 따라 분기를 해야한다던가, 응답 ID를 활용한다던가 하는 경우 말이다. 이럴때 아래와 같이 랜덤한 값을 반환하게 만들어버리면 테스트에서 검증을 하기 힘든경우가 발생한다. 반환값을 이용하여 단언을 해야하는데 값이 랜덤이라 단언이 힘들기 때문이다.

```kotlin
fun Mockery.getSendbirdOrderGroupChannel() {
    sendbirdApi.whenWithDefault(
        requestDefinition = HttpRequest.request()
            .withMethod(HttpMethod.GET.name())
            .withPath("/v3/group_channels/.+"),
    ).respond(
        HttpResponse.response()
            .withStatusCode(200)
            .withBody(
                json(
                    """
                    {
                      "channel_url": "${generateId()}",
                      "name": "${generateString()}",
                      "data": ""
                    }
                    """.trimIndent(),
                ),
            ),
    )
}
```

이러한 문제를 해결하기 위해서는 여러가지 방법이 있다.

첫번째는 함수의 매개변수로 넘기는 방법이다.

```kotlin
fun Mockery.getSendbirdOrderGroupChannel(
    channelUrl: String,
    name: String,
) {
    sendbirdApi.whenWithDefault(
        requestDefinition = HttpRequest.request()
            .withMethod(HttpMethod.GET.name())
            .withPath("/v3/group_channels/.+"),
    ).respond(
        HttpResponse.response()
            .withStatusCode(200)
            .withBody(
                json(
                    """
                    {
                      "channel_url": "${channelUrl}",
                      "name": "${name}",
                      "data": ""
                    }
                    """.trimIndent(),
                ),
            ),
    )
}
```

이게 가장 직관적이고 손쉬운 방법이다. 하지만 상황에 따라서는 시스템 내부에서 랜덤한 값을 생성해서 넘긴다던지 하는 이유로 매개변수를 이용한 Mocking을 할 수 없는 경우가 있다.

두번째 방법은 요청한 매개변수를 그대로 반환하는 방법이다.

Mock Server에서는 [Response Template](https://www.mock-server.com/mock_server/response_templates.html){: target="\_blank" }이라는 기능을 통해서 요청한 값에 대한 참조값을 반환값으로 지정할 수 있도록 해준다. 아래 코드는 Mustache 문법을 이용한 예시이다.

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

이렇게 하면 요청한 Path variable 인 `userId`를 반환값으로 활용할 수 있다.

## Query DSL fetch join

이번 이슈는 내가 Query DSL를 제대로 이해하지 못하고 사용한데서 발생한 이슈이다.

아래와 같이 fetch join을 사용했음에도 불구하고 N+1 쿼리가 실행되는 현상을 겪게 되었다.

```kotlin
val query = from(orderableVendorCatalogProduct)
    .join(orderableVendorProduct)
    .on(orderableVendorCatalogProduct.id.eq(orderableVendorProduct.orderableVendorCatalogProduct.id))
    .fetchJoin()
    .where(filter.getExpression())
    .orderBy(
        promotionPriceOrder(),
        orderableVendorCatalogProduct.createdAt.desc(),
    )
```

```
select count(o1_0.id) from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id=?
select count(o1_0.id) from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id='018a2752-9505-24a4-8f27-68c5096b9da0';
Hibernate: select o1_0.id,o1_0.created_at,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id=? order by case when (o1_0.promotion_price is null) then ? else 1 end asc,o1_0.created_at desc offset ? rows fetch first ? rows only
2023-08-24T20:33:59.014+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839014 | took 3ms | statement | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF
select o1_0.id,o1_0.created_at,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id=? order by case when (o1_0.promotion_price is null) then ? else 1 end asc,o1_0.created_at desc offset ? rows fetch first ? rows only
select o1_0.id,o1_0.created_at,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id='018a2752-9505-24a4-8f27-68c5096b9da0' order by case when (o1_0.promotion_price is null) then 2 else 1 end asc,o1_0.created_at desc offset 0 rows fetch first 10 rows only;
Hibernate: select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.orderable_vendor_catalog_product_id=?
2023-08-24T20:33:59.020+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839020 | took 3ms | statement | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF
select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.orderable_vendor_catalog_product_id=?
select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.orderable_vendor_catalog_product_id='018a2752-96b1-f170-d263-c2a588c22a20';
Hibernate: select o1_0.id,o1_0.authentication_attachments,o1_0.business_address_id,o1_0.business_name,o1_0.created_at,o1_0.can_deliver_friday,o1_0.can_deliver_monday,o1_0.can_deliver_saturday,o1_0.can_deliver_sunday,o1_0.can_deliver_thursday,o1_0.can_deliver_tuesday,o1_0.can_deliver_wednesday,o1_0.email,o1_0.can_inquiry_friday,o1_0.can_inquiry_monday,o1_0.can_inquiry_saturday,o1_0.can_inquiry_sunday,o1_0.can_inquiry_thursday,o1_0.can_inquiry_tuesday,o1_0.can_inquiry_wednesday,o1_0.inquiryable_time,o1_0.invitation_code,o1_0.meta_orderable_vendor_id,o1_0.name,o1_0.reg_num,o1_0.representative,o1_0.updated_at from orderable_vendor o1_0 where o1_0.id=?
2023-08-24T20:33:59.023+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839023 | took 3ms | statement | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF
select o1_0.id,o1_0.authentication_attachments,o1_0.business_address_id,o1_0.business_name,o1_0.created_at,o1_0.can_deliver_friday,o1_0.can_deliver_monday,o1_0.can_deliver_saturday,o1_0.can_deliver_sunday,o1_0.can_deliver_thursday,o1_0.can_deliver_tuesday,o1_0.can_deliver_wednesday,o1_0.email,o1_0.can_inquiry_friday,o1_0.can_inquiry_monday,o1_0.can_inquiry_saturday,o1_0.can_inquiry_sunday,o1_0.can_inquiry_thursday,o1_0.can_inquiry_tuesday,o1_0.can_inquiry_wednesday,o1_0.inquiryable_time,o1_0.invitation_code,o1_0.meta_orderable_vendor_id,o1_0.name,o1_0.reg_num,o1_0.representative,o1_0.updated_at from orderable_vendor o1_0 where o1_0.id=?
select o1_0.id,o1_0.authentication_attachments,o1_0.business_address_id,o1_0.business_name,o1_0.created_at,o1_0.can_deliver_friday,o1_0.can_deliver_monday,o1_0.can_deliver_saturday,o1_0.can_deliver_sunday,o1_0.can_deliver_thursday,o1_0.can_deliver_tuesday,o1_0.can_deliver_wednesday,o1_0.email,o1_0.can_inquiry_friday,o1_0.can_inquiry_monday,o1_0.can_inquiry_saturday,o1_0.can_inquiry_sunday,o1_0.can_inquiry_thursday,o1_0.can_inquiry_tuesday,o1_0.can_inquiry_wednesday,o1_0.inquiryable_time,o1_0.invitation_code,o1_0.meta_orderable_vendor_id,o1_0.name,o1_0.reg_num,o1_0.representative,o1_0.updated_at from orderable_vendor o1_0 where o1_0.id='018a2752-9505-24a4-8f27-68c5096b9da0';
Hibernate: select m1_0.id,m1_0.advertisement_enabled,m1_0.certification_comment,m1_0.detail_images,m1_0.introduction,m1_0.introduction_summary,m1_0.major_products,m1_0.representative_image,m1_0.available_payment_methods,m1_0.business_conditions,m1_0.business_types,m1_0.chat_inquirable,m1_0.delivery_methods,m1_0.delivery_regions,m1_0.delivery_time_range,m1_0.establishment_date,m1_0.main_products,m1_0.major_product_categories,m1_0.major_trade_store_categories,m1_0.matching_enabled,m1_0.memo,m1_0.minimum_order,m1_0.new_store_extendable,m1_0.next_day_delivery_deadline,m1_0.officials,m1_0.payment_intervals,m1_0.preferred_delivery_regions,m1_0.storage_address_id from meta_orderable_vendor m1_0 where m1_0.id=?
2023-08-24T20:33:59.027+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839027 | took 2ms | statement | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF
select m1_0.id,m1_0.advertisement_enabled,m1_0.certification_comment,m1_0.detail_images,m1_0.introduction,m1_0.introduction_summary,m1_0.major_products,m1_0.representative_image,m1_0.available_payment_methods,m1_0.business_conditions,m1_0.business_types,m1_0.chat_inquirable,m1_0.delivery_methods,m1_0.delivery_regions,m1_0.delivery_time_range,m1_0.establishment_date,m1_0.main_products,m1_0.major_product_categories,m1_0.major_trade_store_categories,m1_0.matching_enabled,m1_0.memo,m1_0.minimum_order,m1_0.new_store_extendable,m1_0.next_day_delivery_deadline,m1_0.officials,m1_0.payment_intervals,m1_0.preferred_delivery_regions,m1_0.storage_address_id from meta_orderable_vendor m1_0 where m1_0.id=?
select m1_0.id,m1_0.advertisement_enabled,m1_0.certification_comment,m1_0.detail_images,m1_0.introduction,m1_0.introduction_summary,m1_0.major_products,m1_0.representative_image,m1_0.available_payment_methods,m1_0.business_conditions,m1_0.business_types,m1_0.chat_inquirable,m1_0.delivery_methods,m1_0.delivery_regions,m1_0.delivery_time_range,m1_0.establishment_date,m1_0.main_products,m1_0.major_product_categories,m1_0.major_trade_store_categories,m1_0.matching_enabled,m1_0.memo,m1_0.minimum_order,m1_0.new_store_extendable,m1_0.next_day_delivery_deadline,m1_0.officials,m1_0.payment_intervals,m1_0.preferred_delivery_regions,m1_0.storage_address_id from meta_orderable_vendor m1_0 where m1_0.id='018a2752-9505-24a4-8f27-68c5096b9da2';
Hibernate: select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 left join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o1_0.id=?
2023-08-24T20:33:59.031+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839031 | took 2ms | statement | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF
select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 left join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o1_0.id=?
select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 left join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o1_0.id='018a2752-96b1-f170-d263-c2a588c22a20';
Hibernate: select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.orderable_vendor_catalog_product_id=?
2023-08-24T20:33:59.034+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839034 | took 2ms | statement | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF
select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.orderable_vendor_catalog_product_id=?
select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.orderable_vendor_catalog_product_id='018a2752-9706-7d18-cb9e-5de7203ea54d';
Hibernate: select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 left join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o1_0.id=?
2023-08-24T20:33:59.037+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839037 | took 2ms | statement | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF
select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 left join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o1_0.id=?
select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 left join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o1_0.id='018a2752-9706-7d18-cb9e-5de7203ea54d';
2023-08-24T20:33:59.042+09:00  INFO 89912 --- [  XNIO-1 task-2] p6spy                                    : #1692876839042 | took 2ms | commit | connection 11| url jdbc:postgresql://localhost:51023/test_database?loggerLevel=OFF

;
2023-08-24T20:33:59.087+09:00  INFO 89912 --- [pool-1-thread-1] com.spoqa.cart.Constants 
```

원인을 찾아보니 fetch join은 아래와 같이 사용해야 정상 동작한다는 것을 알 수 있었다. 즉, on절을 명시적으로 적어주면 안된다.

```kotlin
val query = from(orderableVendorCatalogProduct)
    .join(orderableVendorCatalogProduct.orderableVendorProduct, orderableVendorProduct)
    .fetchJoin()
    .where(filter.getExpression())
    .orderBy(
        promotionPriceOrder(),
        orderableVendorCatalogProduct.createdAt.desc(),
    )
```

```
select count(o1_0.id) from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id=?
select count(o1_0.id) from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id='018a2751-27f9-d26d-1488-10c0ccf031a8';
Hibernate: select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id=? order by case when (o1_0.promotion_price is null) then ? else 1 end asc,o1_0.created_at desc offset ? rows fetch first ? rows only
2023-08-24T20:32:25.527+09:00  INFO 89881 --- [  XNIO-1 task-2] p6spy                                    : #1692876745527 | took 5ms | statement | connection 11| url jdbc:postgresql://localhost:50933/test_database?loggerLevel=OFF
select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id=? order by case when (o1_0.promotion_price is null) then ? else 1 end asc,o1_0.created_at desc offset ? rows fetch first ? rows only
select o1_0.id,o1_0.created_at,o2_0.id,o2_0.created_at,o2_0.erp_code,o2_0.is_market_price,o2_0.name,o2_0.orderable_vendor_id,o2_0.standard,o2_0.unit,o2_0.unit_price,o2_0.updated_at,o2_0.vat_included,o1_0.promotion_price from orderable_vendor_catalog_product o1_0 join orderable_vendor_product o2_0 on o1_0.id=o2_0.orderable_vendor_catalog_product_id where o2_0.orderable_vendor_id='018a2751-27f9-d26d-1488-10c0ccf031a8' order by case when (o1_0.promotion_price is null) then 2 else 1 end asc,o1_0.created_at desc offset 0 rows fetch first 10 rows only;
Hibernate: select o1_0.id,o1_0.authentication_attachments,o1_0.business_address_id,o1_0.business_name,o1_0.created_at,o1_0.can_deliver_friday,o1_0.can_deliver_monday,o1_0.can_deliver_saturday,o1_0.can_deliver_sunday,o1_0.can_deliver_thursday,o1_0.can_deliver_tuesday,o1_0.can_deliver_wednesday,o1_0.email,o1_0.can_inquiry_friday,o1_0.can_inquiry_monday,o1_0.can_inquiry_saturday,o1_0.can_inquiry_sunday,o1_0.can_inquiry_thursday,o1_0.can_inquiry_tuesday,o1_0.can_inquiry_wednesday,o1_0.inquiryable_time,o1_0.invitation_code,o1_0.meta_orderable_vendor_id,o1_0.name,o1_0.reg_num,o1_0.representative,o1_0.updated_at from orderable_vendor o1_0 where o1_0.id=?
2023-08-24T20:32:25.532+09:00  INFO 89881 --- [  XNIO-1 task-2] p6spy                                    : #1692876745532 | took 4ms | statement | connection 11| url jdbc:postgresql://localhost:50933/test_database?loggerLevel=OFF
select o1_0.id,o1_0.authentication_attachments,o1_0.business_address_id,o1_0.business_name,o1_0.created_at,o1_0.can_deliver_friday,o1_0.can_deliver_monday,o1_0.can_deliver_saturday,o1_0.can_deliver_sunday,o1_0.can_deliver_thursday,o1_0.can_deliver_tuesday,o1_0.can_deliver_wednesday,o1_0.email,o1_0.can_inquiry_friday,o1_0.can_inquiry_monday,o1_0.can_inquiry_saturday,o1_0.can_inquiry_sunday,o1_0.can_inquiry_thursday,o1_0.can_inquiry_tuesday,o1_0.can_inquiry_wednesday,o1_0.inquiryable_time,o1_0.invitation_code,o1_0.meta_orderable_vendor_id,o1_0.name,o1_0.reg_num,o1_0.representative,o1_0.updated_at from orderable_vendor o1_0 where o1_0.id=?
select o1_0.id,o1_0.authentication_attachments,o1_0.business_address_id,o1_0.business_name,o1_0.created_at,o1_0.can_deliver_friday,o1_0.can_deliver_monday,o1_0.can_deliver_saturday,o1_0.can_deliver_sunday,o1_0.can_deliver_thursday,o1_0.can_deliver_tuesday,o1_0.can_deliver_wednesday,o1_0.email,o1_0.can_inquiry_friday,o1_0.can_inquiry_monday,o1_0.can_inquiry_saturday,o1_0.can_inquiry_sunday,o1_0.can_inquiry_thursday,o1_0.can_inquiry_tuesday,o1_0.can_inquiry_wednesday,o1_0.inquiryable_time,o1_0.invitation_code,o1_0.meta_orderable_vendor_id,o1_0.name,o1_0.reg_num,o1_0.representative,o1_0.updated_at from orderable_vendor o1_0 where o1_0.id='018a2751-27f9-d26d-1488-10c0ccf031a8';
Hibernate: select m1_0.id,m1_0.advertisement_enabled,m1_0.certification_comment,m1_0.detail_images,m1_0.introduction,m1_0.introduction_summary,m1_0.major_products,m1_0.representative_image,m1_0.available_payment_methods,m1_0.business_conditions,m1_0.business_types,m1_0.chat_inquirable,m1_0.delivery_methods,m1_0.delivery_regions,m1_0.delivery_time_range,m1_0.establishment_date,m1_0.main_products,m1_0.major_product_categories,m1_0.major_trade_store_categories,m1_0.matching_enabled,m1_0.memo,m1_0.minimum_order,m1_0.new_store_extendable,m1_0.next_day_delivery_deadline,m1_0.officials,m1_0.payment_intervals,m1_0.preferred_delivery_regions,m1_0.storage_address_id from meta_orderable_vendor m1_0 where m1_0.id=?
2023-08-24T20:32:25.536+09:00  INFO 89881 --- [  XNIO-1 task-2] p6spy                                    : #1692876745536 | took 2ms | statement | connection 11| url jdbc:postgresql://localhost:50933/test_database?loggerLevel=OFF
select m1_0.id,m1_0.advertisement_enabled,m1_0.certification_comment,m1_0.detail_images,m1_0.introduction,m1_0.introduction_summary,m1_0.major_products,m1_0.representative_image,m1_0.available_payment_methods,m1_0.business_conditions,m1_0.business_types,m1_0.chat_inquirable,m1_0.delivery_methods,m1_0.delivery_regions,m1_0.delivery_time_range,m1_0.establishment_date,m1_0.main_products,m1_0.major_product_categories,m1_0.major_trade_store_categories,m1_0.matching_enabled,m1_0.memo,m1_0.minimum_order,m1_0.new_store_extendable,m1_0.next_day_delivery_deadline,m1_0.officials,m1_0.payment_intervals,m1_0.preferred_delivery_regions,m1_0.storage_address_id from meta_orderable_vendor m1_0 where m1_0.id=?
select m1_0.id,m1_0.advertisement_enabled,m1_0.certification_comment,m1_0.detail_images,m1_0.introduction,m1_0.introduction_summary,m1_0.major_products,m1_0.representative_image,m1_0.available_payment_methods,m1_0.business_conditions,m1_0.business_types,m1_0.chat_inquirable,m1_0.delivery_methods,m1_0.delivery_regions,m1_0.delivery_time_range,m1_0.establishment_date,m1_0.main_products,m1_0.major_product_categories,m1_0.major_trade_store_categories,m1_0.matching_enabled,m1_0.memo,m1_0.minimum_order,m1_0.new_store_extendable,m1_0.next_day_delivery_deadline,m1_0.officials,m1_0.payment_intervals,m1_0.preferred_delivery_regions,m1_0.storage_address_id from meta_orderable_vendor m1_0 where m1_0.id='018a2751-27fa-cb6b-82ba-16f0e847382a';
2023-08-24T20:32:25.541+09:00  INFO 89881 --- [  XNIO-1 task-2] p6spy                                    : #1692876745541 | took 2ms | commit | connection 11| url jdbc:postgresql://localhost:50933/test_database?loggerLevel=OFF

;
2023-08-24T20:32:25.580+09:00  INFO 89881 --- [pool-1-thread-1] com.spoqa.cart.Constants   
```

## JPA One To One Lazy Loading

One-To-One을 사용하면 Lazy를 선언해도 Eager 로딩을 사용하는걸 볼 수 있다. [Hibernate Tips](https://thorben-janssen.com/hibernate-tip-lazy-loading-one-to-one/){: target="\_blank" }를 보면 MapsId 를 사용하면 해결할 수 있을 듯 한데 테스트 해봐야할거 같다.

## JPA 사용할때 주의할 사례

최근 프로젝트를 진행하면서 데이터를 수정요청했는데 수정이 되지 않는다는 이슈가 있었다. 원인이 되는 코드가 아래와 같았는데, 같은 Entity를 순환참조형태로 의존성을 가지는 형태를 가진 Entity가 문제였다.

```kotlin
@Entity
class OrderableVendorProduct {
    // ... 생략
    @OneToMany(mappedBy = "bundleProduct", fetch = FetchType.LAZY, cascade = [CascadeType.REMOVE], orphanRemoval = true)
    protected val singleProductUnit: MutableSet<OrderableVendorProductBundle> = mutableSetOf()
    val productUnit get() = singleProductUnit.firstOrNull()

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

        if (data.withCatalog) {
            setCatalog(promotionPrice = data.promotionPrice)
        } else {
            this.orderableVendorCatalogProduct = null
        }
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
class OrderableVendorProductBundle {
    // ... 생략
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "id", nullable = false, insertable = false, updatable = false)
    val product: OrderableVendorProduct = product

    @ManyToOne(fetch = FetchType.LAZY, optional = false, cascade = [CascadeType.ALL])
    @JoinColumn(name = "bundle_product_id", nullable = false)
    val bundleProduct: OrderableVendorProduct = bundleProduct
}
```

`update` 함수는 품목 정보를 수정하는 함수이고 `syncBundle`은 묶음품목을 동기화해주는 기능이다.

문제는 아래의 서비스함수에서 수정기능을 수행할 때 발생하였다.

```kotlin
 @Transactional
fun update(orderableVendorProductId: UUID, data: OrderableVendorProductUpdateData): OrderableVendorProduct {
    // ... 생략

    product.update(data)
    product.productBundle?.let {
        product.syncBundle()
    }
}
```

품목을 수정한 후에 묶음 품목이 존재하는 경우 동기화를 수행하도록 한다. 코드상으로는 아무 문제가 없어보인다. 하지만 실제로 동작시켜보면 update가 되지 않는 것을 볼 수 있다. 아래는 해당 함수의 로그를 확인하기 위한 테스트 코드이다.

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
2023-08-27T18:31:04.885+09:00  INFO 3409 --- [  XNIO-1 task-2] p6spy                                    : #1693128664885 | took 1ms | statement | connection 17| url jdbc:postgresql://localhost:60864/test_database?loggerLevel=OFF
select s1_0.id,s1_0.bundle_product_id,s1_0.created_at,s1_0.unit_count from orderable_vendor_product_bundle s1_0 where s1_0.id=?
select s1_0.id,s1_0.bundle_product_id,s1_0.created_at,s1_0.unit_count from orderable_vendor_product_bundle s1_0 where s1_0.id='018a3655-265f-caf4-1af4-bc7eab50e75b';
Hibernate: select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.id in(?,?)
2023-08-27T18:31:04.889+09:00  INFO 3409 --- [  XNIO-1 task-2] p6spy                                    : #1693128664889 | took 1ms | statement | connection 17| url jdbc:postgresql://localhost:60864/test_database?loggerLevel=OFF
select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.id in(?,?)
select o1_0.id,o1_0.created_at,o1_0.erp_code,o1_0.is_market_price,o1_0.name,o1_0.orderable_vendor_id,o1_0.orderable_vendor_catalog_product_id,o1_0.standard,o1_0.unit,o1_0.unit_price,o1_0.updated_at,o1_0.vat_included from orderable_vendor_product o1_0 where o1_0.id in('018a3655-2679-fdc3-f3c0-03e978ed1047','018a3655-265f-caf4-1af4-bc7eab50e75b');
======== 33333333
======== com.spoqa.cart.domain.orderableVendor.OrderableVendorCatalogProduct@9671a783
======== 44444444
======== com.spoqa.cart.domain.orderableVendor.OrderableVendorCatalogProduct@9671a783
```

로그를 보면 2번 로그에서 `orderableVendorCatalogProduct`가 `update` 함수가 실행된 후 `null`로 변경된것을 확인할 수 있다. 하지만 갑자기 3번 로그부터 `null`로 변경되었던 `orderableVendorCatalogProduct`가 다시 존재하는 것을 볼 수 있다.

이유를 찾아보니 `OrderableVendorProduct`Entity에서 `productBundle`를 조회하면 `OrderableVendorProductBundle`이 가진 `product`가 자동으로 로딩되게 되고 이때 ID값이 주체인 `OrderableVendorProduct`과 같기 때문에 캐싱한 Entity 값을 가지고 오게 된다. 그래서 `update`로 인해 수정된 값은 온데간데 없이 사라지고 최초 조회한 `OrderableVendorProduct`의 상태로 되돌아간 것이다.

그래서 `productBundle`을 먼저 불러온 후 수정하도록 구현코드를 변경하였다. 사실 이런코드가 마음에 들진 않지만 JPA의 메커니즘대로 동작하게 해야하니 어쩌겠는가...

```kotlin
if (product.productBundle == null) {
    product.update(data)
} else {
    product.update(data)
    product.syncBundle()
}
```

다행히도 이부분은 기능테스트로 전환하였기 때문에 발견할 수 있었고 쉽게 고칠 수 있게 되었다. 다만 이런 이슈는 JPA를 잘 이해하지 못하면 원인을 찾는게 여간 쉽지 않다. 일단 순환 참조하는 구조는 다시 생각해봐야겠다.
