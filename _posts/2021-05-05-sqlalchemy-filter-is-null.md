---
layout: single
title: "SQLAlchemy의 filter에서 is None/is not None은 왜 동작하지 않을까?"
categories:
  - EXPLANATION
tags:
  - Python
  - SQLAlchemy

toc: true
toc_sticky: true
excerpt: 이 글은 Python 초보인 내가 SQLAlchemy를 (~~제대로 알지 못하고~~)사용하면서 겪었던 이슈에 대해 원인을 찾아보고 왜 SQLAlchemy가 그렇게 동작하게끔 구현되었는지, 앞으로 사용할 때 어떤 주의를 기울여야 하는지를 정리한 글이다.
---

## Intro

이 글은 Python 초보인 내가 [SQLAlchemy](https://www.sqlalchemy.org/){: target="\_blank" }를 (~~제대로 알지 못하고~~)사용하면서 겪었던 이슈에 대해 원인을 찾아보고 왜 SQLAlchemy가 그렇게 동작하게끔 구현되었는지, 앞으로 사용할 때 어떤 주의를 기울여야 하는지를 정리한 글이다.

## 모델 소개

이 글을 설명하기 위해서 Model을 먼저 소개해야 좀더 설명을 매끄럽게 할 수 있을 것 같다. 아래에 간단한 `Employee`이라는 모델이 있다.

`Employee`는 식별자와 이름, 그리고 정규직 전환일 속성을 가지고 있다. 그리고 정규직 전환일의 존재여부에 따라 정규직인지 비정규직인지를 판단할 수 있다. 정규직/비정규직 여부를 판단하는 함수는 `@hybrid_property`와 `@expression`을 이용하여 표현하였다. [Hybrid attribue 참고](https://docs.sqlalchemy.org/en/13/orm/extensions/hybrid.html){: target="\_blank" }

```python
from sqlalchemy import Column, DateTime
from sqlalchemy.ext.hybrid import hybrid_property
from sqlalchemy.sql.expression import null
from sqlalchemy_utils import UUIDType


class Employee(object):
    id = Column(UUIDType, primary_key=True)
    name = Column(String, nullable=False)
    full_time_transition_at = Column(DateTime(timezone=True))

    @hybrid_property
    def is_full_time(self):
        return self.full_time_transition_at is not None

    @is_full_time.expression
    def is_full_time(cls):
        return cls.full_time_transition_at != None

    __tablename__ = 'employees'
```

## 사건의 발단

### 첫번째 이슈

문제의 시작은 `expression`을 위와같이 작성하지 않고 아래와 같이 작성한 것이었다.

```python
@is_full_time.expression
def is_full_time(cls):
    return cls.full_time_transition_at is not None
```

<br/>

그리고 정규직 근로자 목록을 조회하기 위해 `filter`함수를 사용하여 `where`절을 사용하였다. [Query API 참고](https://docs.sqlalchemy.org/en/13/orm/query.html#sqlalchemy.orm.query.Query.filter){: target="\_blank" }

```python
employees = session.query(Employee)\
    .filter(Employee.is_full_time).all()
```

<br/>

하지만 실행되는 쿼리를 보니 내가 예상했던 쿼리가 아니었다.

예상했던 쿼리

```sql
select
    employees.id,
    employees.name,
    employees.full_time_transition_at
from employees
where employees.full_time_transition_at is not null
```

<br/>

실제로 동작한 쿼리

```sql
select
    employees.id,
    employees.name,
    employees.full_time_transition_at
from employees
where true
```

<br/>

이유를 찾아보니 SQLAlchemy에서는 `magic method`를 사용하니 제대로 동작하게 하려면 `!=`을 사용해야 한다는 것을 알게 되었다. [stackoverflow 참고](https://stackoverflow.com/questions/5602918/select-null-values-in-sqlalchemy/5632224){: target="\_blank" }

그래서 `expression`을 아래와 같이 변경하였고

```python
@is_full_time.expression
def is_full_time(cls):
    return cls.full_time_transition_at != None
```

<br/>

다시 아래 코드를 실행해보니

```python
employees = session.query(Employee)\
    .filter(Employee.is_full_time).all()
```

<br/>

쿼리가 예상한대로 정상적으로 동작하였다.

```sql
select
    employees.id,
    employees.name,
    employees.full_time_transition_at
from employees
where employees.full_time_transition_at is not null
```

<br/>

이렇게 마무리 지어도 되겠지만 무언가 마음에 들지 않는 부분이 있었다. 그건 바로 아래 그림처럼 Pycharm에서 warning으로 표시되는 노란색 줄이었다.

![warning-line](/assets/images/posts/sqlalchemy-filter-is-null/warning-line.png)

<br/>

`Comparison with None performed with equality operators` 즉, None은 `equals`연산자를 사용하지 말고 `is`연산자를 사용하라는 경고 문구 였다. `is`연산자는 앞에서 봤듯이 사용할 수 없으니 다른 방법을 찾아야만 했다. 그래서 찾은 방법이 SQLAlchemy에서 제공하는 [operaters](https://docs.sqlalchemy.org/en/13/core/sqlelement.html?highlight=operations#sqlalchemy.sql.operators.ColumnOperators.isnot){: target="\_blank" }와 [expression](https://docs.sqlalchemy.org/en/13/core/sqlelement.html#sqlalchemy.sql.expression.null){: target="\_blank" }을 이용하는 것이었다.

```python
@is_full_time.expression
def is_full_time(cls):
    return cls.full_time_transition_at.isnot(null())
```

<br/>

하지만 `datetime` 타입의 속성에 `isnot`함수를 사용해도 여전히 warning이 표시된다. 왜냐하면 `datetime` 타입의 속성은 `isnot`함수를 제공해주지 않기 때문이다.

![warning-line2](/assets/images/posts/sqlalchemy-filter-is-null/warning-line-2.png)

<br/>

결론은 둘중에 맘에 드는 방식을 사용하자는 것이다. 개인적으로는 단순하면서 깔끔한 `equals` 연산자를 사용한 방법이 마음에 든다.

### 두번째 이슈

두번째 이슈는 첫번째 이슈를 해결하고 나서 새로운 요구사항을 구현하기 위해 새로운 쿼리를 작성하면서 발생했다. 요구사항은 바로 비정규직 근로자를 조회하는 것이었다. 그래서 이전에 사용한 코드에서 논리값을 뒤집는 `not`연산자를 사용해 보았다.

```python
employees = session.query(Employee)\
    .filter(not Employee.is_full_time).all()
```

<br/>

하지만 쿼리는 또다시 내가 예상한대로 동작하지 않았다.

예상했던 쿼리

```sql
select
    employees.id,
    employees.name,
    employees.full_time_transition_at
from employees
where employees.full_time_transition_at is null
```

<br/>

실제 동작 쿼리

```sql
select
    employees.id,
    employees.name,
    employees.full_time_transition_at
from employees
where false
```

~~`is`연산자가 동작하지 않는걸 보고 예상했으면 좋았겠지만 그러지 못한건 아쉬운 부분이다.~~

그러면 비정규직 근로자를 조회하려면 어떻게하면 좋을까?? 먼저 `is_full_time`함수와 반대되는 `is_not_full_time`함수를 새롭게 만드는 것이다. 솔직히 이방법도 나쁘지 않은 방법이라 생각되었다. 근로자 모델이 비정규직 여부를 조회할 일도 많을 것이라 예상되기 때문이다.

하지만 처음 `not`을 사용했던 것처럼 `is_full_time`의 반대 연산자를 통해서 조회하는 방법을 찾고 싶었다. 그러다 찾은 방법이 바로 `invert`(`~`) 연산자를 사용하는 것이었다. `invert`연산자는 비트연산자중 하나로 bit단위로 not연산을 한다. `invert`, `inv`, `~`중에 선택에서 사용하면 된다. 개인의 취향이겠지만 `~`연산자가 간편하고 깔끔해 보이긴 한데, 처음 `~`연산자에 대해서 잘 모르는 상태에서 봤을때에는 해당 쿼리가 어떤 의미를 가지는지 찾아봐야 했다.

```python
employees = session.query(Employee)\
    .filter(invert(Employee.is_full_time)).all()
```

```python
employees = session.query(Employee)\
    .filter(inv(Employee.is_full_time)).all()
```

```python
employees = session.query(Employee)\
    .filter(~Employee.is_full_time).all()
```

<br/>

예상대로 쿼리가 잘 동작함을 볼 수 있다.

```sql
select
    employees.id,
    employees.name,
    employees.full_time_transition_at
from employees
where employees.full_time_transition_at is null
```

## SQLAlchemy의 query.filter

앞에서 말한 사례를 보면 알 수 있겠지만 SQLAlchemy의 `filter`함수에 사용되는 인자인 `Employee.is_full_time`의 반환 값이 `hybrid_propertyProxy`를 반환함을 알 수 있다. 다른부분은 생략하고 `expression`을 보면 `employees.full_time_transition_at IS NOT NULL`값을 가지고 있는 것을 볼 수 있다.

![debugger-is-full-time](/assets/images/posts/sqlalchemy-filter-is-null/debugger-is-full-time.png)

<br/>

`Employee.full_time_transition_at != None`은 어떨까?? 다른 반환타입이지만 `expression`을 보면 `employees.full_time_transition_at IS NOT NULL`값을 가지고 있는 것을 볼 수 있다.

![debugger-full-time-transition-at-not-equals-null](/assets/images/posts/sqlalchemy-filter-is-null/debugger-full-time-transition-at-not-equals-null.png)

<br/>

`~Employee.is_full_time`을 보아도 마찬가지로 `BinaryExpression`을 반환하면서 `employees.full_time_transition_at IS NULL`을 가지고 있음을 볼 수 있다.

![debugger-invert-is-full-time](/assets/images/posts/sqlalchemy-filter-is-null/debugger-invert-is-full-time.png)

<br/>

하지만 `Employee.full_time_transition_at is not None`을 보면 반환타입이 `bool`임을 볼 수 있다.

![debugger-full-time-transition-at-is-not-null](/assets/images/posts/sqlalchemy-filter-is-null/debugger-full-time-transition-at-is-not-null.png)

<br/>

`not Employee.is_full_time`도 마찬가지로 `bool`을 반환한다.

![debugger-not-is-full-time](/assets/images/posts/sqlalchemy-filter-is-null/debugger-not-is-full-time.png)

<br/>

SQLAlchemy의 `filter`함수를 들여다 보면 아래와 같은데

```python
@_generative(_no_statement_condition, _no_limit_offset)
def filter(self, *criterion):
    r"""Apply the given filtering criterion to a copy
    of this :class:`_query.Query`, using SQL expressions.

    e.g.::

        session.query(MyClass).filter(MyClass.name == 'some name')

    Multiple criteria may be specified as comma separated; the effect
    is that they will be joined together using the :func:`.and_`
    function::

        session.query(MyClass).\
            filter(MyClass.name == 'some name', MyClass.id > 5)

    The criterion is any SQL expression object applicable to the
    WHERE clause of a select.   String expressions are coerced
    into SQL expression constructs via the :func:`_expression.text`
    construct.

    .. seealso::

        :meth:`_query.Query.filter_by` - filter on keyword expressions.

    """
    for criterion in list(criterion):
        criterion = expression._expression_literal_as_text(criterion)

        criterion = self._adapt_clause(criterion, True, True)

        if self._criterion is not None:
            self._criterion = self._criterion & criterion
        else:
            self._criterion = criterion
```

expression을 이용하여 SQL의 Where절을 만드는 것을 볼 수 있다. 그래서 `Employee.full_time_transition_at != None`, `~Employee.is_full_time`는 예상한대로 쿼리가 생성되고 `Employee.full_time_transition_at is not None`, `not Employee.is_full_time`는 쿼리가 예상한대로 생성되지 않는 것이다.

### Magic Method

매직 메소드(Magic Method)는 특별 메소드(Special Method), 던더 메소드(Dunder Method)라고도 불리는데, 파이썬의 객체에서 정의된 조금 특별한 함수들을 말한다. 주로 `__`로 시작해서 `__`로 끝나는 이름을 가지며 파이썬에서 사용되는 빌트인 타입 함수를 동작하게 하거나 연산자의 동작을 재정의할 때 사용한다. 이외에도 아래와 같은 기능에 대해서 지원하기도 한다. [Fluent Python 참고](http://www.kyobobook.co.kr/product/detailViewEng.laf?ejkGb=BNT&mallGb=ENG&barcode=9781491946008){: target="\_blank" }

- 반복
- 컬렉션
- 속성 접근
- 연산자 오버로딩
- 함수 호출
- 객체 생성 및 제거
- 문자열 표현
- 블록 및 컨텍스트 관리

### SQLAlchemy operators

갑자기 웬 `Magic Method`에 대한 설명을 할까 생각할 수도 있겠지만 `Employee.full_time_transition_at != None`과 `~Employee.is_full_time`이 어떻게 SQL문의 Where절을 만들 수 있는지를 이해하려면 앞선 설명이 먼저 필요하다고 생각해서 적어 보았다.

먼저 `Employee.full_time_transition_at != None`를 통해서 어떻게 쿼리문을 생성할 수 있는지 살펴보자. SQLAlchemy의 `ColumnOperators`를 들여다보면 `__eq__`와 `__ne__`가 정의되어 있는 것을 볼 수 있다.

```python
def __eq__(self, other):
    """Implement the ``==`` operator.

    In a column context, produces the clause ``a = b``.
    If the target is ``None``, produces ``a IS NULL``.

    """
    return self.operate(eq, other)

def __ne__(self, other):
    """Implement the ``!=`` operator.

    In a column context, produces the clause ``a != b``.
    If the target is ``None``, produces ``a IS NOT NULL``.

    """
    return self.operate(ne, other)
```

<br/>

`~Employee.is_full_time`도 살펴보자. `Operators`를 들여다보면 `__invert__`가 정의되어 있는 것을 볼 수 있다.

```python
def __invert__(self):
    """Implement the ``~`` operator.

    When used with SQL expressions, results in a
    NOT operation, equivalent to
    :func:`_expression.not_`, that is::

        ~a

    is equivalent to::

        from sqlalchemy import not_
        not_(a)

    """
    return self.operate(inv)
```

<br/>

반환값이 `self.operate`인 것을 볼 수 있다. `self.operate`의 반환 타입은 `BinaryExpression`이다. 이것이 바로 `==`, `!=`, `~`가 동작하는 이유인 것이다. 만약 해당 함수가 재정의 되어있지 않았다면 `Employee.full_time_transition_at != None`의 반환값은 `bool`이었을 것이고 쿼리문은 예상했던 대로 동작하지 않았을 것이다.

그럼 `is`나 `not`은 왜 동작하지 않을까? 다시 `operators.py`를 들여다 보자.

```python
def is_(self, other):
    """Implement the ``IS`` operator.

    Normally, ``IS`` is generated automatically when comparing to a
    value of ``None``, which resolves to ``NULL``.  However, explicit
    usage of ``IS`` may be desirable if comparing to boolean values
    on certain platforms.

    .. seealso:: :meth:`.ColumnOperators.isnot`

    """
    return self.operate(is_, other)

def isnot(self, other):
    """Implement the ``IS NOT`` operator.

    Normally, ``IS NOT`` is generated automatically when comparing to a
    value of ``None``, which resolves to ``NULL``.  However, explicit
    usage of ``IS NOT`` may be desirable if comparing to boolean values
    on certain platforms.

    .. seealso:: :meth:`.ColumnOperators.is_`

    """
    return self.operate(isnot, other)
```

`is`나 `not`은 Python의 Special Operator로 따로 Magic Method로 재정의하지 않고 별도 함수로 제공하고 있음을 볼 수 있다. 그래서 `Employee.full_time_transition_at != None`을 `Employee.full_time_transition_at.isnot(null())`으로 표현해야만 예상한대로 동작하는 것이다.

## 마무리

주로 JPA나 QueryDSL을 사용한 경험을 가지고 있다보니 처음에는 `Employee.full_time_transition_at.isnot(null())`이렇게 사용하면 되지 굳이 `__eq__`을 재정의하면서 `Employee.full_time_transition_at != None`을 사용할 필요가 있을까 라는 생각을 했었다. 하지만 파이썬에는 파이썬스러움(Pythonic)이라는 단어가 있을 정도로 파이썬만이 가진 일관성과 단순함, 우아함을 추구하는 철학이 있다. 모두다 이해했다고 말하긴 힘들지만 아마 이러한 관점에서 SQLAlchemy의 operators도 매직함수들을 재정의하면서 `==`, `!=`, `~`등을 표현할 수 있도록 하지 않았을까 라는 생각이 든다.

이번 이슈를 파헤치면서 책으로 보았던 Magic Method의 실제 사용 사례를 좀더 깊계 살펴볼 수 있게 되었고 파이썬이라는 언어와 SQLAlchemy에 대해 좀더 이해할 수 있는 계기가 되어 좋은 경험을 했다고 생각한다.

> [https://docs.sqlalchemy.org/en/13/index.html](https://docs.sqlalchemy.org/en/13/index.html){: target="\_blank" }
>
> [https://rszalski.github.io/magicmethods/](https://rszalski.github.io/magicmethods/){: target="\_blank" }
>
> [https://wikidocs.net/83755](https://wikidocs.net/83755){: target="\_blank" }
