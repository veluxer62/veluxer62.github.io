---
layout: single
title: "2021년 6월 개발일지"\
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2021-june-dev-log/
---

벌써 2021년의 반이 지나가고 7월이다. 지난 반년을 돌아보면 짧다면 짧고 길다면 긴 시간동안 이직도 하고 코로나 핑계로 그동안 만나지 못했던 지인들을 만나기도 하고 새롭게 운동도 시작하기도 하는 등 많은 것들은 하였다. 자세한 내용은 2021년 회고 때 다루어보도록 해야겠다. 

아쉬운 점은 상반기 동안 책을 많이 읽지 않았다는 것이다. 꾸준히 읽어가곤 있지만 독서량이 아주 많이 적어졌다. 집에서 독서하기 좋은 구조로 되어있던 환경이 불편하게 바뀌면서 그나마 가지려고 노력했던 독서습관을 멈춘게 아닌가 라는 핑계를 대어본다. 하반기에는 좀더 많은 책을 읽기위해 노력해야겠다. (~~내 책상을 마련할 수 있는 집으로 이사 가고 싶다.~~)

6월부터 헬스장을 다니기 시작했다. 이전까지는 아침이나 저녁에 가끔 자전거를 타곤 했는데, 최근 비도 자주오고 날씨 영향을 많이 받는 야외 운동이라 꾸준히 하기 힘들어 보여서 그동안 코로나 핑계로 미뤄왔던 헬스를 다시 시작하게 되었다. 

처음 일주일은 오랜만의 근육운동이라 근육통과 함께 피곤함이 많이 느껴졌는데 현재는 많이 적응해서 그런지 운동을 하기 전보다 훨씬 활력이 넘치고 삶의 질도 높아져서 왜 진즉 하지 않았나 라는 생각이 든다. (~~사람의 실수는 끝없이 반복된다...쿨럭~~) 

목표는 다이어트이긴 하지만 식이를 하지 않으니 드라마틱 하게 살이 빠지진 않는다. (~~몸무게가 오히려 늘어난건 안비밀~~) 그래도 꾸준히 하다보면 건강한 몸을 유지할 수 있지 않을까?

아래는 6월동안 겪었던 개발 내용들이다.

## auto comment

Github에서 Pull Request나 Issue 이벤트 발생 시 Github Action이 발동하도록 하는 플러그인이 있어서 적어본다. [auto-comment](https://github.com/marketplace/actions/auto-comment){: target="\_blank" }이다. PR등록 시 특정 코멘트나 리엑션이 있다면 실행되도록도 할 수 있어서 여러방면으로 활용하기 좋을 듯 하다.

## Mapping Class Inheritance Hierarchies

다른 이유로 도입하지 않았지만 Entity모델의 삭제 시 백업용 Entity모델을 관리하기 위한 용도로 SQLAlchemy의 [Mapping Class Inheritance Hierarchies](https://docs.sqlalchemy.org/en/14/orm/inheritance.html){: target="\_blank" }를 검토해 보았었다.

Mapping Class Inheritance Hierarchies는 총 3가지의 기법을 제공해 주는데 [Joined Table Inheritance](https://docs.sqlalchemy.org/en/14/orm/inheritance.html#joined-table-inheritance){: target="\_blank" }와 [Single Table Inheritance](https://docs.sqlalchemy.org/en/14/orm/inheritance.html#single-table-inheritance){: target="\_blank" } 그리고, [Concrete Table Inheritance](https://docs.sqlalchemy.org/en/14/orm/inheritance.html#concrete-table-inheritance){: target="\_blank" } 이다.

### Joined Table Inheritance

부모 Entity를 두고 자식 Entity가 부모 Entity를 상속받아 부모가 가진 속성이나 행위를 공통으로 사용하는데, 실제로 DB에서는 부모 테이블과 자식들의 테이블이 각각 만들어지고 영속화된 데이터는 join을 통해서 가져오는 기법을 사용한다.

### Single Table Inheritance

하나의 DB테이블을 사용하면서 부모 Entity와 자식 Entity를 나누어 사용하는 방식이다. 주의할 점은 하나의 자식 Entity가 다른 자식 Entity와 동일한 속성명을 정의할 수 없다는 점인데 이는 `@declared_attr`을 통해 해결 할 수 있다.

### Concrete Table Inheritance

부모 Entity의 속성들을 자식 Entity가 상속하지만 자식 Entity에서 동일한 속성이라도 재정의 한다는 점에서 Joined Table Inheritance와 조금 다르다. 실제로 DB에서도 영속화된 데이터를 조회할 때 `UNION ALL`을 통해서 데이터를 조회한다. 다른 속성을 가진 자식 Entity들을 부모 Entity 타입으로 모두 조회할 때 유용할 수 있겠다.

## black

Python의 Code Formatter로 [black](https://github.com/psf/black){: target="\_blank" }이라는 게 있어서 적어둔다. 아직 활용은 안해봐서 어떤지는 모르겠지만 일반적인 formatter가 제공하는 기능들을 제공하지 않을까?

## 보안 관련 질문

사내에서 물리서버를 사용하는데 외부 접근 제어가 필요하여 아는 보안엔지니어분께 아래와 같은 질문을 하였었고 좋은 답변을 받아서 기억해 두고자 적어본다.

### SSM이 VPN에 비해 좋은점?

VPN이나 베스쳔은 결국 외부퍼블릭망에 공개되어 네트워크적인 공격/위협이 있는 방면 SSM은 이미 공개된 AWS API엔드포인트와 통신하며 리버스프록시 방식이라, 실제 서버가 노출될 가능성이 적음과 더불어 비용적 측면이 VPN보다 낫다. 다만, VPN을 운용해야되는 타 상황이 있다면 혼용해서 같이 운용함을 추천한다. (디비접근 등) 그와 더불어 SSM은 패치매니저를 통한 EC2서버를 업데이트하는 일련의 과정을 중앙에서 관리할 수 있는 기능을 제공해서 편리하다.

### SSM을 이용해서 사무실 내에 물리서버 접근 제어를 할 수 있나?

가능하다. AWS SSM agent만 설치하면 된다.

### SSH와 VPN의 접근 제어에서의 차이점?

SSH와 VPN의 차이점은 다른레이어에서 말하는건데, 굳이 꺼냈으니 답변하자면, SSH Brute Forcing에 대한 공격자 비용과 VPN Brute Forcing에 대한 비용은 확연히 차이가 난다.

### AWS 계정이 탈취당하는 것에 대한 위험성은?

인증일원화나 SSO 도입을 통해서 중앙계정관리를하고, 그 이후의 프로세스에 대한 risk assetment 수립이 중요하다. VPN역시도 계정관리란 측면을 벗어날 순 없다.

## gitmoji

git에서 commit 메시지에 사용할 수 있는 이모지 가이드 문서가 있어서 적어본다. [https://gitmoji.dev/](https://gitmoji.dev/){: target="\_blank" }

## 파이썬 내장 커멘드라인 스크립트

파이썬 사용시 활용해 봄직한(?) 쓸만한 내장 커멘드라인 스크립트를 소개한 글이 있어 적어본다. [https://ryanking13.github.io/2021/06/10/python-commandline-scripts.html](https://ryanking13.github.io/2021/06/10/python-commandline-scripts.html){: target="\_blank" }

json 포메팅이나 인코딩/디코딩 등은 익숙해지면 자주 사용할 것 같다.

## 단위테스트 모범사례

Microsoft 문서는 볼때마다 감탄스럽다. 시간날때마다 읽을만한거 선정해서 정독하면 좋을 만큼 좋은 글들이 많다. 특히 기술에 대한 단순한 정의 뿐만 아니라 모범사례 등을 소개하면서 좋은 인사이트를 줗은 문서들은 정말 고마운 문서들이다. 이번에 [단위테스트에 대한 모범사례글](https://docs.microsoft.com/ko-kr/dotnet/core/testing/unit-testing-best-practices){: target="\_blank" }을 보았는데 기존에 알고 있던 단위 테스트에 대한 지식을 정리할 수 있는 시간을 가질 수 있어서 좋았다. 특히 [Mock에 대한 잘못된 사용을 바로잡아 주는 부분](https://docs.microsoft.com/ko-kr/dotnet/core/testing/unit-testing-best-practices#lets-speak-the-same-language){: target="\_blank" }은 이때까지 모호하게 사용하던 Stub과 Mock, Spy에 대해 정리 할 수 있어서 많은 도움이 되었다.

## SQLAlchemy cascade 이슈

SQLAlchemy에서 Entity 삭제 시 cascade 동작에 대한 이슈를 겪었었다. 만약 아래와 같이 코드를 작성해서 Entity를 삭제하면 cascade가 동작하지 않는다.

```python
session.query(Foo).filter(Foo.id == id).delete()
```

아쉽게도 [공식문서](https://docs.sqlalchemy.org/en/14/orm/cascades.html#delete){: target="\_blank" }에는 이 구문이 왜 동작하지 않는지에 대한 명확한 설명글이 없다. (못찾은 것일 수도 있다. 하지만 눈에 띄는 경고문구가 없다 ㅠ) 경고문구라도 있었다면 이런실수를 하는 개발자가 적을텐데 조금 아쉬운 부분이다.

아래와 같이 작성하면 동작한다. 이때 조심해야 하는 부분은 `foo`가 `None`인 경우 오류를 반환하니 예외처리를 잘 해주어야 한다.

```python
foo = session.query(Foo).get(id)
session.delete(foo)
```

관련 이슈는 [stackoverflow](https://stackoverflow.com/questions/19243964/sqlalchemy-delete-doesnt-cascade){: target="\_blank" }에도 이슈로 등록되어 있다.

## kite

[kite](https://www.kite.com/){: target="\_blank" }는 AI code completion을 제공해주는 에디터이다. 현재는 pycharm을 사용하고 있어서 필요성이 없는데 괜찮은지 한번 사용해봐야 겠다.

## Dont Catch Exceptions

최근 예외처리와 관련한 이런저런 글을 보면서 관련 자료를 찾다가 괜찮은 글을 발견해서 기록해 둔다. [https://wiki.c2.com/?DontCatchExceptions](https://wiki.c2.com/?DontCatchExceptions){: target="\_blank" } 필요한 상황이 아니라면 예외를 잡아채지 마라.

## Lazy 로딩 실수

SQLAlchemy를 사용할때 특정 Entity를 조회할때 연관된 데이터는 Lazy로딩으로 조회하는 것이 기본값이다. 이번에 이 메커니즘을 이해하고 있음에도 불구하고 실수한 부분이 있어서 적어본다.

먼저 Entity를 적어보면 아래와 같다.

```python
class Foo(Base):
    # 생략
    bars = relationship('Bar')

class Bar(Base):
    # 생략
    foo = relationship('Foo', back_populates='bars')
```

구현사항은 `Bar` Entity들 중 하나를 지운다고 하였을 때 만약 `Foo` Entity가 삭제한 `Bar` Entity만 가지고 있다면 `Foo` Entity도 함께 삭제하는 것이었다. 처음 구현은 아래와 같았다.

```python
bar = session.query(Bar).get(id)
foo = bar.foo
session.delete(bar)

if len(foo.bars) == 1:
    session.delete(foo)
```

`len(foo.bars) == 1`에서 예상한 부분과 다르게 동작하면서 오류가 발생하였는데 그이유는 만약 `Foo` Entity가 삭제한 `Bar` Entity만 가지고 있다면 `len(foo.bars) == 0`이어야 하기 때문이다. 처음 작성할땐 아무생각없이 `foo`가 `bar`를 메모리상에 가지고 있을 거라 생각했다. lazy 로딩을 간과한 것이었다. 실제로 `foo`는 `foo.bars`가 호출되기 전까지 `bar`들을 메모리상에 가지고 있지 않고 호출되는 시점에 조회해서 가지고 온다. `session.delete(bar)`가 `foo.bars`를 호출하기 이전에 실행이 되었기 때문에 `bar`는 이미 삭제되어 버려서 존재하지 않을 것이고 `foo.bars`의 조회 결과는 0일 것이다. 그러므로 아래와 같이 작성되어야 정상적으로 동작한다.

```python
bar = session.query(Bar).get(id)
foo = bar.foo
session.delete(bar)

if not foo.bars:
    session.delete(foo)
```

또 다른 해결책으로는 [Eager 로딩](https://docs.sqlalchemy.org/en/14/orm/loading_relationships.html#joined-eager-loading){: target="\_blank" }을 활용하거나 아래 코드와 같이 메모리에 먼저 불러온 후 삭제하는 방법을 활용할 수 있는데, Eager 로딩은 Lazy 로딩의 장점을 포기하거나 별도로 Eager 로딩을 하도록 코드를 추가해야 한다는 점에서 위 해결방법보다 좋지 못하다는 생각이 들었고 아래 코드도 억지스러운 면이 있어서 좋지 못하다 생각이 든다.

```python
bar = session.query(Bar).get(id)
foo = bar.foo
bars = foo.bars
session.delete(bar)

if bars == 1:
    session.delete(foo)
```

## Video to GIF convertor

비디오를 GIF로 변환해주는 온라인 컨버터 사이트를 발견했는데 앞으로 활용할 일이 많을 것 같아서 적어본다. [http://videotogif.thetimetube.com/](http://videotogif.thetimetube.com/){: target="\_blank" }

## 예외의 목적

다음과 같은 목적으로 예외를 사용한다.

- 적합한 수준에서 문제를 처리한다(어떻게 할지 모르는 경우에는 예외를 잡지 않도록 한다).
- 문제를 수정하고 예외를 발생시켰던 메소드를 다시 호출한다.
- 수정 부분을 교정하고 메소드를 재실행하지 않고 계속 진행한다.
- 메소드에서 산출하기로 되어 있었던 것을 대신할 수 있는 대안 결과를 추정한다.
- 현재의 컨텍스트에서 할 수 있는 것을 하고, 상위 컨텍스트에 대해 동일한 예외를 다시 던진다.
- 현재의 컨텍스트에서 할 수 있는 것을 하고, 상위 컨텍스트에 대해 다른 예외를 던진다.
- 프로그램을 종료한다.
- 단순화한다(예외 설계가 좀더 복잡해지면, 사용하기가 힘들고 성가시게 된다).
- 라이브러리와 프로그램을 안전하게 한다(이는 디버깅을 위한 단기적인 투자이며, 견고한 애플리케이션을 위한 장기적 투자이다).

## SQLAlchemy 연관관계 매핑 실수

위에서 다루었던 Entity를 다시 가져와보자. cascade를 추가했다.

```python
class Foo(Base):
    # 생략
    bars = relationship('Bar', cascade='all')

class Bar(Base):
    # 생략
    foo = relationship('Foo', back_populates='bars')
```

Entity 저장 시 아래와 같이 할 수 있다.

```python
bars = [
    Bar(),
    Bar(),
    Bar(),
    Bar(),
]

foos = [
    Foo(bars=bars),
    Foo(bars=bars),
]

session.add_all(foos)
session.commit()
```

위와 같이 실행하면 어떻게 저장될까? 바로 2개의 `foo`중 `bars`는 한곳에만 저장이 된다. 참조값을 넘겨준 것이기 때문에 어찌보면 당연하다. (~~이런 기초적인 실수를 하다니 반성하자~~) 문제는 이걸 테스트 시 발견을 빠르게 하지 못했는데 이유는 2개의 `foo`중 한곳에 랜덤하게 저장이 되면서 테스트가 성공과 실패를 반복했기 때문이다. 심지어 패턴도 없어서 왜 테스트가 성공하는지 원인을 찾는데 한참 헤맸다.

## zsh auto suggestion

zsh를 사용할때 이전에 사용한 이력을 바탕으로 자동완성과 비슷한 커멘드 제안 기능을 제공하는 gist가 있어서 적어본다. [https://gist.github.com/Xednicoder/b6522a1e70e75c0118b966cd02fbce14](https://gist.github.com/Xednicoder/b6522a1e70e75c0118b966cd02fbce14){: target="\_blank" }

## cupcake anti pattern

[cupcake anti pattern](https://www.thoughtworks.com/insights/blog/introducing-software-testing-cupcake-anti-pattern){: target="\_blank" }이라는 글인데 좋은 글이라 적어둔다. 여기서 말하는 GUI는 백엔드 시선에서는 API Endpoint로 보아도 무방할 것 같다.

## jacoco

[kotest](https://kotest.io/){: target="\_blank" }에서 jacoco도 내장으로 지원해준다. 이번에 프로젝트 하면서 사용해 봐야 겠다.

[https://kotest.io/docs/framework/integrations/jacoco.html](https://kotest.io/docs/framework/integrations/jacoco.html){: target="\_blank" }
