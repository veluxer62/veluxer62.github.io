---
layout: single
title: "JPA Application Level Isolation"
categories:
  - EXPLANATION
tags:
  - jpa
  - isolation level

toc: true
toc_sticky: true
excerpt: JPA에서 동작하는 Isolation 레벨에 대해서 알아보자.
---

## Intro

DBMS마다 각각 다른 기본 격리수준을 제공한다. Mysql, Maria 데이터베이스는 기본적으로 `REPEATABLE READ`이고, Oracle, MSSql은 `READ COMMITTED`이다. Aurora의 경우는 Writer 인스턴스는 `REPEATABLE READ` Reader 인스턴스는 `READ COMMITTED`이다.

Spring에서 JPA를 이용하여 Taransaction을 처리할때 `@Transactional`어노테이션을 사용하는데, 이때 기본 격리 수준은 데이터베이스의 격리수준을 따라간다고 한다.JPA는 영속성 컨텍스트라는 매커니즘을 사용한다. 영속성 컨텍스트의 캐싱은 영속성 모델에 대한 조회 시 DB를 직접 조회하는 것이 아니라 캐싱된 데이터를 조회하는 것이다. 

만약에 데이터베이스의 격리수준이 `REPEATABLE READ`인 경우에는 Transaction이 시작되고 종료되는 동안 SELECT한 값이 동일할 것이기 때문에 JPA의 영속성 컨텍스트가 조회값을 캐싱해도 문제가 되지 않는다. 하지만 데이터베이스 격리수준이 `READ COMMITTED`라면 Transaction이 시작되고 종료되는 동안 SELECT한 값이 변경될 수 있을 것이라 예상하지만 JPA의 영속성 컨텍스트의 캐싱 메커니즘으로 인해 `REPEATABLE READ`로 동작하지 않을까?

## Set Database Config

> 테스트는 Maria DB를 이용하였다.

테스트를 위해 단순한 테이블을 생성하고 데이터를 추가했다.

```sql
CREATE TABLE `test-db`.board (
	id BIGINT UNSIGNED auto_increment NOT NULL,
	title varchar(100) NOT NULL,
	`view` INT UNSIGNED DEFAULT 0 NOT NULL,
	CONSTRAINT board_PK PRIMARY KEY (id)
)
ENGINE=InnoDB
DEFAULT CHARSET=latin1
COLLATE=latin1_swedish_ci;

INSERT INTO board (title, `view`) VALUES('board-1', 0);
INSERT INTO board (title, `view`) VALUES('board-2', 0);
INSERT INTO board (title, `view`) VALUES('board-3', 0);
```

Maria DB의 기본 격리 수준은 `REPEATABLE READ`이기 때문에 `READ COMMITTED`로 변경하자.

```sql
set tx_isolation = 'READ-COMMITTED';
commit;
```

자동 커밋을 끈다.

```sql
set autocommit = false;
```

## DB Testing

### Session 1

```sql
begin;

# before session 2 committed
# view = 0
select view from board b where id = 1;

# after session 2 committed
# view = 3
select view from board b where id = 1;

commit;
```

### Session 2

```sql
begin;

UPDATE board set `view` = 3 where id = 1;

commit;
```

보는 바와 같이 DB에서 격리수준을 `READ COMMITTED`으로 변경하면 예상한 바와 같이 동작하는 것을 볼 수 있다.

## JPA Testing

### JPA

![jpa test snapshot](/assets/images/posts/jpa-application-level-isolation/jpa-test-snapshot.png)

### Session 2

```sql
begin;

UPDATE board set `view` = 3 where id = 1;

commit;
```

캡쳐에서 보듯이 다른 세션에서 커밋한 내용이 Transaction 내에서 변경되지 않고 동일한 값으로 조회 되는 것을 볼 수 있다. 아래와 같이 SQL로그를 보아도 SELECT 쿼리가 하나만 실행된 것을 볼 수 있다. 즉, JPA의 영속성 컨텍스트의 캐싱 매커니즘으로 인해 두번째 조회 쿼리는 실제 DB에서 조회하는 것이 아닌 캐싱된 영속성 컨텍스트에서 데이터를 조회하는 것이다.

![test log](/assets/images/posts/jpa-application-level-isolation/jpa-test-log.png)

## Wrap Up

위 테스트를 통해서 JPA를 이용하여 관계형 데이터베이스를 다룰 때 DB의 격리 수준과 다르게 동작할 수 있음을 확인할 수 있었다. 아마 영속성 모델에 대한 깊은 이해를 가지고 있었다면 이 부분은 당연히 알고 있지 않았을까. 기본 격리 수준이 `REPEATABLE READ`을 사용하는 DB가 아니라 다른 격리수준을 사용하는 DB나 격리수준을 변경해서 사용할 때 JPA를 사용한다면 이런 메커니즘은 숙지하고 사용하면 좋을 것 같다.

> [https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Reference.html#AuroraMySQL.Reference.IsolationLevels](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Reference.html#AuroraMySQL.Reference.IsolationLevels){: target="\_blank" }
>
> [https://mariadb.com/kb/en/set-transaction/](https://mariadb.com/kb/en/set-transaction/){: target="\_blank" }
>
> [https://docs.microsoft.com/en-us/sql/t-sql/statements/set-transaction-isolation-level-transact-sql?view=sql-server-ver15](https://docs.microsoft.com/en-us/sql/t-sql/statements/set-transaction-isolation-level-transact-sql?view=sql-server-ver15){: target="\_blank" }
>
> [https://docs.oracle.com/database/121/ADFNS/adfns_sqlproc.htm#ADFNS99871](https://docs.oracle.com/database/121/ADFNS/adfns_sqlproc.htm#ADFNS99871){: target="\_blank" }
>
> [https://docs.spring.io/spring-data/jpa/docs/2.3.2.RELEASE/reference/html/#transactions](https://docs.spring.io/spring-data/jpa/docs/2.3.2.RELEASE/reference/html/#transactions){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction](https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction){: target="\_blank" }
>
> [https://ict-nroo.tistory.com/130](https://ict-nroo.tistory.com/130){: target="\_blank" }
