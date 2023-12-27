---
layout: single
title: "2021년 9월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2021-september-dev-log/
---

9월은 코로나 백신 1차 접종으로 이틀간 보건휴가 사용과 추석연휴가 겹치면서 빠르게 지나갔던 한달이었던 것 같다.

6월이후로 꾸준히 운동을 한 덕분에(?) 몸에 근육이 많이 생긴것 같아 뿌듯하다. 하지만 특별히 식이를 하지 않으니 역시 살이 빠지진 않는다. 10월에 결혼식이 있는데 좀 빼야겠다.

추석 이후로 저녁에 1시간 씩 독서와 알고리즘 공부를 하기로 마음 먹었다. 첫주는 열심히 했는데 업무가 좀 몰리면서 최근에는 많이 빼먹었다. 다시 마음을 다잡고 해야겠다!

내 블로그에도 작성한 글을 기반으로 사내 블로그에 [도메인 주도 개발 전환 이야기](https://spoqa.github.io/2021/09/13/domain-driven-development-transition-story.html){: target="\_blank" }를 업로드 하였다.

최근에 제품 개발을 위한 JIRA 사용 가이드라는 글도 사내 위키에 작성해서 공유하였는데 이 글도 잘 정리해서 사내 블로그에 업로드 해야겠다.

아래는 9월 동안 정리한 이슈 내용들이다.

## The Clean Code Blog

최근에 개발관련 글을 찾다가 Robert C. Martin이 운영중인 [The Clean Code Blog](https://blog.cleancoder.com/uncle-bob/2021/03/06/ifElseSwitch.html){: target="\_blank" }를 발견하게 되었다. 최근까지고 블로그 글을 작성하고 있는걸로 보아 여전히 운영하고 있는 듯 하다. 클린코드, 클린아키텍쳐의 저자이기도한 Robert C. Martin의 생각들을 엿볼 수 있는 주옥같은 게시글들이 많이 있어 시간날때마다 방문에서 읽어봐야 겠다.

## The Open-Closed Principle

SOLID에서 SRP 만큼이나 쉬운것 같으면서도 막상 코드 작성 시 어려운 원칙 중 하나가 OCP라고 생각한다. OCP관련해서 글을 찾아보면 많은 글들을 볼 수 잇는데 PDF로 자세히 적혀있는 글을 발견해서 적어둔다.

[https://web.archive.org/web/20060822033314/http://www.objectmentor.com/resources/articles/ocp.pdf](https://web.archive.org/web/20060822033314/http://www.objectmentor.com/resources/articles/ocp.pdf){: target="\_blank" }

## Code With Me

올해 8월 젯브레인에서 Code With Me 대규모 업데이트를 하였다는 소식을 전해왔다.

[https://blog.jetbrains.com/ko/blog/2021/08/04/code-with-me-2021-2/](https://blog.jetbrains.com/ko/blog/2021/08/04/code-with-me-2021-2/){: target="\_blank" }

최근 코로나 여파로 재택근무가 많아지면서 원격 페어프로그래밍에 대한 필요성이 많이 부각되고 있는 것 같다. 이전에 VSCode에서 라이브쉐어를 통해 페어프로그래밍을 진행해본적 있었는데 젯브레인제품에도 이러한 기능이 없었던게 아쉬웠었는데, Code With Me를 통해 편리하게 원격 페어프로그래밍을 진행할 수 있게 되어서 좋다. 다만 한글 사용 시 너무 깜빡이는 현상은 어서 개선되면 좋을 것 같다.

## JOIN LATERAL

코드리뷰를 하면서 동료분께서 JOIN LATERAL을 사용하시는것을 보고 Postgres의 JOIN LATERAL을 알게되었다.

JOIN RATERAL은 for each와 유사한 것으로 설명되곤하는데 join시 앞서 조회한 값들의 row를 참고할 수 있다는 것이 join과 다른 큰 특징이다. 말로 설명하는것보다 예제를 보여주는 게 좀더 이해가 쉬울 것 같아서 아래에 예를 들어 보겠다. 해당 예제는 [PostgreSQL's Powerful New Join Type: LATERAL](https://heap.io/blog/postgresqls-powerful-new-join-type-lateral){: target="\_blank" }에서 가져온 것이다.

테스트할 테이블을 생성하자.

```sql
-- create event table
CREATE TABLE event (
  user_id BIGINT,
  event_id BIGINT,
  time BIGINT NOT NULL,
  data JSON NOT NULL,
  PRIMARY KEY (user_id, event_id)
)
```

먼저 JOIN RATERAL을 사용하여 홈페이지를 방문한 사용자와 홈페이지 방문 후 2주이내 신용카드를 사용한 사용자를 조회하는 쿼리를 작성해보자.

```sql
SELECT
  sum(view_homepage) AS viewed_homepage,
  sum(use_demo) AS use_demo
FROM (
  -- Get the first time each user viewed the homepage.
  SELECT
    user_id,
    1 AS view_homepage,
    min(time) AS view_homepage_time
  FROM event
  WHERE
    data->>'type' = 'view_homepage'
  GROUP BY user_id
) e1 LEFT join (
  -- For each row, get the first time the user_id did the use_demo
  -- event, if one exists within one week of view_homepage_time.
  SELECT
    user_id,
    1 AS use_demo,
    time AS use_demo_time
  FROM event
  WHERE
    user_id = e1.user_id AND
    data->>'type' = 'use_demo' AND
    time BETWEEN view_homepage_time AND (view_homepage_time + 1000*60*60*24*7)
  ORDER BY time
  LIMIT 1
) e2 ON true
```

만약 JOIN RATERAL을 사용하지 않는다면 e2의 값을 어떻게 조회할 수 있을까? 만약 `LIMIT 1`이 없었다면 일반적인 Join으로 해결 할 수 있었겠지만 `LIMIT 1`때문에 많은 트릭을 써서 복잡한 쿼리가 만들어질 것이다.

## Zapier

신규 입사자분의 부트캠프 때 슬렉 봇을 만드는 과제를 진행하였다. 슬렉봇의 서버를 어디다 둘까 고민하다가 [Zapier](http://penpaperandplot.com/2019/08/22/ppp-blog-zapier/){: target="\_blank" }를 사용한 사례가 있길래 Zapier를 이용하여 봇을 만들어보기로 하였다. 비용문제만 없다면 간단한 기능을 만들어서 써보기에는 나쁘지 않아보인다.

## chakra-ui

[chakra-ui](https://chakra-ui.com/){: target="\_blank" } Bootstrap, Ant-design과 같은 컴포넌트 라이브러리이다.

## reMarkable 2

회사 동료분이 [reMarkable 2](https://remarkable.com/store/remarkable-2){: target="\_blank" }를 추천해주셨다. 테블릿인데 필기감이 종이와 아주 흡사해 사용감이 정말 좋다고 한다. 돈 많이 벌면 사봐야지...

## Coroutines @RestController

Kotlin의 Coroutines를 이용하여 컨트롤러를 만들때 suspend function을 사용하면 Non blocking API를 만들 수 있다. functional Endpoint를 이용하여 API를 만든다면 `coRoute`를 사용할 수 있고 Java Reactor를 사용한다면 Mono나 Flux객체를 반환값으로 사용하면 된다.

[https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#controllers](https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#controllers){: target="\_blank" }

## Transactional

WebFlux를 사용할 때 `@Transaction`을 사용하면 예상한대로 동작하지 않는다. 그럼 Transaction을 동작하게 하려면 어떻게 하면 좋을까??
Kotlin의 Coroutine을 사용한다면 `TransactionalOperator.executeAndAwait`를 사용하면 된다.

```kotlin
class PersonRepository(private val operator: TransactionalOperator) {

    suspend fun initDatabase() = operator.executeAndAwait {
        insertPerson1()
        insertPerson2()
    }

    private suspend fun insertPerson1() {
        // INSERT SQL statement
    }

    private suspend fun insertPerson2() {
        // INSERT SQL statement
    }
}
```

Reactor를 사용한다면 아래와 같이 사용할 수 있다.

```java
TransactionalOperator rxtx = TransactionalOperator.create(tm);

Mono<Void> outsideTransaction = db.execute()
  .sql("INSERT INTO person (name, age) VALUES('Jack', 31)")
  .then();

Mono<Void> insideTransaction = rxtx.execute(txStatus -> {
  return db.execute()
    .sql("INSERT INTO person (name, age) VALUES('Joe', 34)")
    .fetch().rowsUpdated()
    .then(db.execute()
      .sql("INSERT INTO contacts (name) VALUES('Joe Black')")
      .then());
  }).then();

Mono<Void> completion = outsideTransaction.then(insideTransaction);
```

## Spring validator interface

Spring의 [ConstraintValidator](https://www.baeldung.com/spring-mvc-custom-validator){: target="\_blank" }를 이용하면 커스텀한 validator를 만들 수 있다.

문자열 관련한 validator들을 만들어두고 사용하면 유용하게 사용할 수 있을 듯 하다.

```java
@Documented
@Constraint(validatedBy = {PhoneNumberValidator.class})
@Target({METHOD, FIELD, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
public @interface PhoneNumber {

	String message() default "{com.spoqa.homework.phoneNumber}";
	Class<?>[] groups() default {};
	Class<? extends Payload>[] payload() default {};
}

public class PhoneNumberValidator implements ConstraintValidator<PhoneNumber, String> {
	@Override
	public boolean isValid(String value, ConstraintValidatorContext context) {
		if (value == null) {
			return true;
		}

		return value.matches("\\+\\d+");
	}
}
```
