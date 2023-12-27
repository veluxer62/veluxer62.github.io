---
layout: single
title: "2021년 5월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2021-may-dev-log/
---

어느덧 2021년의 2분기의 마지막 달이 다가왔다. 지금 회사에서도 벌써 3개월차에 접어들었다. 시간이 참 빠른것 같다.

파이썬을 사용하는 것은 아직 능숙하게 사용한다고는 하지 못하겠지만 어느정도 익숙해 진 것 같다. [Fluent Python](https://www.hanbit.co.kr/store/books/look.php?p_code=E9050803868){: target="\_blank" }를 읽어가면서 Python이 가진 언어적 특성도 어느정도 이해하게 되었고(물론 전적으로 동의하지 않는 부분도 많다.) 나름 데코레이터를 만들어서 적용해보면서 새로운 시도들도 해보고 있다.

섣부른 판단일 순 있으나 지금까지 내가 내린 Flask는 경량 API로 빠르게 프로토 타입의 API를 만들어 내어 검증하고 사용하거나 MSA에서 가벼운 서버로 사용해 봄직하다고 생각이 든다. 하지만 모놀리식에서 사용하기에는 성능적인 부분은 모르겠고 유지보수 측면에 아쉬움을 느끼고 있다. 이전 회사에서도 느꼈지만 경량 프레임워크는 자유도가 높은 대신 개발자들의 역량에 영향을 많이 미친다고 느껴진다. 물론 잘하는 개발자는 회사의 니즈에 맞는 좋은 설계를 바탕으로 효율적인 개발을 수행해 갈 수 있다. 하지만 모든 개발자가 동일하게 높은 수준을 가질 수 없고 오히려 높은 자유도로 인해 유지보수가 불가능할 정도로 지나치게 복잡한 코드를 양산할 수 있다. 그리고 스타트업의 특성상 새로운 개발인력의 유입이 빈번할 수 밖에 없는데, 새로 입사한 개발자는 도메인 파악 뿐만 아니라 회사가 가진 자신만의 프레임워크(?)를 익히기 위한 러닝커브를 감수해야한다. 차라리 [Django](https://www.djangoproject.com/){: target="\_blank" }를 사용해서 어느정도 잡혀진(?) 틀 내에서 비지니스에 집중하는게 더 낫지 않을까 라는 생각이 들기도 한다.

현재 회사에서는 코드리뷰를 도입하고 적극 활용하고 있다. 사실 이전 회사들도 코드리뷰제도를 도입하고 있었고 어느정도 활용하고 있었지만 현재 회사와 같이 활발히 리뷰활동을 하지는 않았었다. 하지면 지금은 코드 작성 시간만큼이나 코드리뷰에 시간을 투자하고 있으며 내가 작성한 코드에 대해서도 동료분들의 적극적인(?) 리뷰를 받으면서 기능을 만들어 가고 있다. 개인적으로는 참으로 좋은 경험을 하고 있다고 생각한다. 가끔 생산성에 대한 고민이 들기도 하지만 좋은 코드리뷰를 하기 위한 활동에 대해서도 고민해볼 수도 있고 내가 모르는 부분, 특히 파이썬을 처음 사용하면서 미쳐 알지 못했던 것들을 리뷰를 통해 많이 배우고 있어서 동료들에게 감사한 마음을 표하고 싶다. 가끔 설계 방법에 대한 의견차이로 설전(?)을 벌이기도 하지만 앞으로도 이런 적극적인 리뷰문화가 이어져 갔으면 좋겠다.

아래는 5월동안 겪었던 개발 내용들이다.

## python 테스팅 시 시간을 고정하고 싶다면 freezegun을 사용해보자

테스트를 하다보면 특정 시간을 고정하고 테스트를 진행해야 하는 상황이 생긴다. 이런 경우 [freezegun](https://github.com/spulec/freezegun)을 활용하면 손쉽게 테스트 할 수 있다.

## SQLAlchemy Hybrid

SQLAlchemy에서 Entity의 동일한 함수를 정적 함수와 인스턴스 함수로 활용하기 위해서 `@hybrid_property`를 지원하고 있다. 어떤때 활용할 수 있는가 하면 일단 정적함수는 SQLAlchemy의 Query문의 Filter에 expression으로 활용한다. 그리고 인스턴스 함수는 Entity의 property로 활용된다.

python은 method overloading을 언어레벨에서 지원하지 않는다. [stackoverflow 참고](https://stackoverflow.com/questions/6434482/python-function-overloading){: target="\_blank" } SQLAlchemy에서는 `@hybrid_property`를 이용하여 하나의 name으로 filter expression과 Entity의 propery로 활용하고 싶었을 것이다. 하지만 나는 다르게 동작하는 두기능을 런타임에서 동작할 것이라는 가정을 하고 사용하는 것이 좋지 못한 디자인이라 생각이 든다.

공식 문서에서 제공하는 아래 예제를 보자

```python
@hybrid_property
def fullname(self):
    return self.firstname + " " + self.lastname
```

Entity의 인스턴스 함수라면 전혀 논란의 여지가 없다. `self`는 Eneity 자신을 의미하고 자신의 속성값들을 조합해서 반환한다. 반환값은 생략되어 있지만 `str`일 것으로 추측된다. 하지만 filter expression이라면 조금 이상해진다. `self`는 오히려 `cls`라는 네이밍이 좀더 적합해 보인다. 그리고 반환값은 `str`이 아닌 `Expression`이 반환된다.

위 함수를 하나의 함수로만 정의한 이유는 filter expression으로 사용해도 예상한 대로 동작하기 때문이다. 만약 다르게 동작한다면 아래와같이 별도로 정의해 주어야 한다.

```python
@hybrid_property
def fullname(self):
    if self.firstname is not None:
        return self.firstname + " " + self.lastname
    else:
        return self.lastname

@fullname.expression
def fullname(cls):
    return case([
        (cls.firstname != None, cls.firstname + " " + cls.lastname),
    ], else_ = cls.lastname)
```

파이썬 개발에 익숙한 분들의 입장에서는 제대로 동작하면 뭐가 문제냐라고 할 수도 있겠다. 하지만 나는 유지보수 측면에서 바라보고 싶다. 아래와 같이 작성되었을 때 작성자 입장에서는 코드의 양이 많아져서 조금의 번거로움이 있겠지만 유지보수 하는 개발자 입장에서는 어느 코드에 집중하면 되는지 명확해지고 고쳐야할 범위를 산정하기가 아주 쉬울 수 있다.

```python
@hybrid_property
def fullname(self):
    return self.firstname + " " + self.lastname

@fullname.expression
def fullname(cls):
    return cls.firstname + " " + cls.lastname
```

그런 의미에서 개인적으로는 SQLAlchemy가 `@hybrid_property`를 제공하기 보다는 `expression`을 사용하고자 한다면 동일한 이름이라도 반드시 추가로 정의하도록 하였더라면 좋지 않았을까라는 생각을 가져본다.

## SQLAlchemy의 Expression 사용시 주의할 점

SQLAlchemy를 처음 사용할 때 겪었던 실수가 `null` 여부를 판단할 때 `is Non` 또는 `is not None`을 사용한 것이었다. 하지만 SQLAlchemy의 filter expression은 `is None`과 `is not None`을 제대로 표현해 주지 않는다. `== None` 또는 `!= None`을 사용해야 정상적으로 동작한다. 관련해서는 [SQLAlchemy의 filter에서 is None/is not None은 왜 동작하지 않을까?](https://veluxer62.github.io/explanation/sqlalchemy-filter-is-null/){: target="\_blank" }글에 자세히 정리해 두었다.

## system exit 관련 코드

system exit 관련 코드들을 정리해두면 유용하게 사용할 수 있을 것 같아서 정리해 둔다.

```
E2BIG = 7

EACCES = 13
EADDRINUSE = 48
EADDRNOTAVAIL = 49
EAFNOSUPPORT = 47
EAGAIN = 35
EALREADY = 37
EAUTH = 80

EBADARCH = 86
EBADEXEC = 85
EBADF = 9
EBADMACHO = 88
EBADMSG = 94
EBADRPC = 72
EBUSY = 16

ECANCELED = 89
ECHILD = 10
ECONNABORTED = 53
ECONNREFUSED = 61
ECONNRESET = 54

EDEADLK = 11
EDESTADDRREQ = 39
EDEVERR = 83
EDOM = 33
EDQUOT = 69

EEXIST = 17

EFAULT = 14
EFBIG = 27
EFTYPE = 79

EHOSTDOWN = 64
EHOSTUNREACH = 65

EIDRM = 90
EILSEQ = 92
EINPROGRESS = 36
EINTR = 4
EINVAL = 22
EIO = 5
EISCONN = 56
EISDIR = 21

ELOOP = 62

EMFILE = 24
EMLINK = 31
EMSGSIZE = 40
EMULTIHOP = 95

ENAMETOOLONG = 63
ENEEDAUTH = 81
ENETDOWN = 50
ENETRESET = 52
ENETUNREACH = 51
ENFILE = 23
ENOATTR = 93
ENOBUFS = 55
ENODATA = 96
ENODEV = 19
ENOENT = 2
ENOEXEC = 8
ENOLCK = 77
ENOLINK = 97
ENOMEM = 12
ENOMSG = 91
ENOPOLICY = 103
ENOPROTOOPT = 42
ENOSPC = 28
ENOSR = 98
ENOSTR = 99
ENOSYS = 78
ENOTBLK = 15
ENOTCONN = 57
ENOTDIR = 20
ENOTEMPTY = 66
ENOTRECOVERABLE = 104
ENOTSOCK = 38
ENOTSUP = 45
ENOTTY = 25
ENXIO = 6

EOPNOTSUPP = 102
EOVERFLOW = 84
EOWNERDEAD = 105

EPERM = 1
EPFNOSUPPORT = 46
EPIPE = 32
EPROCLIM = 67
EPROCUNAVAIL = 76
EPROGMISMATCH = 75
EPROGUNAVAIL = 74
EPROTO = 100
EPROTONOSUPPORT = 43
EPROTOTYPE = 41
EPWROFF = 82

ERANGE = 34
EREMOTE = 71
EROFS = 30
ERPCMISMATCH = 73

ESHLIBVERS = 87
ESHUTDOWN = 58
ESOCKTNOSUPPORT = 44
ESPIPE = 29
ESRCH = 3
ESTALE = 70

ETIME = 101
ETIMEDOUT = 60
ETOOMANYREFS = 59
ETXTBSY = 26

EUSERS = 68

EWOULDBLOCK = 35

EXDEV = 18
```

## AWS lambda function name limit

aws lambda function name 적을때 64글자 이내로 만들어야한다. 기억해두자. [AWS 문서 참고](https://docs.aws.amazon.com/lambda/latest/dg/API_CreateFunction.html){: target="\_blank" }

## 손상방지 레이어

만약 새로운 시스템이 추가되거나 기존의 API를 새롭게 변경하면서 하위호환을 유지하는 등의 기능을 만들어야할 때 [손상방지 레이어](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/anti-corruption-layer){: target="\_blank" }를 두면 새로운 시스템이나 API의 디자인을 해치치 않으면서 기존의 시스템이나 API를 변경하지 않을 수 있어 유용하게 사용할 수 있다.

## Null value index

nullable 컬럼에 null과 null이 아닌 값을 조회할때 인덱스를 따로 주지 않아도 조회속도가 나올까?? 찾아보니 Null 값은 인덱스를 타지 않는 것처럼 보인다. [Stackoverflow 참고](<https://stackoverflow.com/questions/9175591/index-for-nullable-column#:~:text=By%20default%2C%20relational%20databases%20ignore,is%20ignored%20(by%20default).>){: target="\_blank" }

## SSM port forwarding

SSM을 이용하여 prot forwarding을 하기 위해서 아래와 같은 방법들이 있다.

### 방법1

```
ssh -o ProxyCommand="aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'" <i-로 시작하는 인스턴스 이름>
```

다만 포터포워딩만 될뿐 SSH 키가 등록된 것이 아니기 때문에 `aws ssm start-session --target <인스턴스이름>`로 접속한 다음 `~/.ssh/authorized_keys`에 자신의 퍼블릭 키를 등록 해 준 후 위 커맨드로 다시 실행하면 된다.

[AWS 문서 참고](https://aws.amazon.com/ko/blogs/aws/new-port-forwarding-using-aws-system-manager-sessions-manager/){: target="\_blank" }

### 방법2

로컬환경의 `~/.ssh/config`에 아래 코드를 추가한다.

```
Host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
    User ec2-user
    PreferredAuthentications publickey
```

그러면 `ssh <i-로시작하는인스턴스이름>`명령어 만으로도 접근할 수 있다. 다만 포터포워딩만 될뿐 SSH 키가 등록된 것이 아니기 때문에 [1번 방법](#방법1) `aws ssm start-session --target <인스턴스이름>`로 접속한 다음 `~/.ssh/authorized_keys`에 자신의 퍼블릭 키를 등록 해 주어야 한다.

## python에서 가변 객체를 default로 사용할때 주의할 점

python에서 매개변수의 기본값을 사용되는 값이 가변 객체인 경우에 예기치 못한 유령객체를 생성할 수 있으므로 주의를 기울여야 한다. 되도록이면 가변객체 사용을 지양하고 불변객체를 사용하는 것이 좋다. 아래 예시를 보면 좀더 이해가 편할 것 같다. [코드 출처: Fluent Python](https://www.hanbit.co.kr/store/books/look.php?p_code=E9050803868){: target="\_blank" }

```python
class HauntedBus:
  """유령 승객이 출몰하는 버스 모델"""
  def __init__(self, passengers=[]):
    self.passengers = passengers

  def pick(self, name):
    self.passengers.append(name)

  def drop(self, name):
    self.passengers.remove(name)
```

```
>>> bus1 = HauntedBus(['Alice', 'Bill']) >>> bus1.passengers
['Alice', 'Bill']
>>> bus1.pick('Charlie')
>>> bus1.drop('Alice')
>>> bus1.passengers
['Bill', 'Charlie']
>>> bus2 = HauntedBus()
>>> bus2.pick('Carrie')
>>> bus2.passengers
['Carrie']
>>> bus3 = HauntedBus()
>>> bus3.passengers
['Carrie']
>>> bus3.pick('Dave')
>>> bus2.passengers
['Carrie', 'Dave']
>>> bus2.passengers is bus3.passengers
True
>>> bus1.passengers
['Bill', 'Charlie']
```

## 코딩 시 문법을 지켜야 하는가?

코드를 작성할 때 클래스나 변수, 함수명의 선택은 참으로 어렵다. 한국인이기 때문에 영어로된 적절한 단어를 찾는 것도 개발의 일부라 할 수 있을 만큼 많은 시간을 소요하고 또한 중요한 부분이라 생각한다. 가끔 영문법에 맞지 않는 단어를 사용하는 코드들을 볼 수 있는데 개인적으로는 코드를 읽는 다른 개발자들의 가독성에 영향을 미친다고 생각하기 때문에 문법을 지키는 것이 바람직하다고 생각한다. 아래는 stackoverflow에서 문법에 대한 질문과 그 답변에 대한 링크이다. [stackexchange 링크](https://softwareengineering.stackexchange.com/questions/30895/should-i-point-out-spelling-grammar-related-mistakes-in-someones-code/30900#30900){: target="\_blank" }

## Shell alias parameter 설정

shell 의 alias를 이용하여 커멘드를 실행할때 파라미터 변수를 사용할 수없다. 사용하고 싶다면 별도 함수를 정의하고 사용해야 한다.

```
fun_foo() {
  ./foo.sh $1 $2
}
alias foo=fun_foo
```

## Edit vs Update vs Change

중요하지 않을 수 있는데 회사내 정책으로 잡아두면 일관되게 쓰일 수 있을 거 같아서 고민해보면 좋을 것 같다.

[stackexchange 링크](https://ux.stackexchange.com/questions/43174/update-vs-modify-vs-change-create-vs-add-delete-vs-remove#:~:text=Also%2C%20in%20programming%2C%20change%20denotes,aka%2C%20'dirty%20records'){: target="\_blank" }

## Fig

[https://fig.io/?ref=hn](https://fig.io/?ref=hn){: target="\_blank" }는 terminal tool인데 홈페이지를 보면 알겠지만 직관적인 코드 어시스턴트를 보여준다. zsh랑 호환도 되어서 사용해봄직 하다.

## Transactional Decorator

함수에서 SQLAlchemy의 세션을 시작하고 커밋 후 종료하는 코드를 매번 사용하는게 좋지 않아보여서 데코레이터를 만들어 보았다. session 객체는 local proxy에 저장되어 request session 내에서는 동일한 객체를 사용하도록 개발되어 있어서 session 객체를 매개변수로 넘기도록 구현하진 않았다.

```python
def transactional(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        try:
            session.begin_nested()
            result = func(*args, **kwargs)
            session.commit()
            return result
        except Exception as e:
            session.rollback()
            raise e
    return wrapper
```

코드를 보면 `transactional`이 중첩될 수 있기에 `session.begin_nested()`를 사용했다. 하지만 실제 코드에서는 정상적으로 동작하지 않아 결국엔 롤백을 했다... 디버깅을 해보니 request session을 종료할 때 이유없이 `session.rollback()`이 실행되는 것을 볼 수 있었다. 좀 살펴보니 request session이 시작될때 SQLAlchemy의 session을 불러오면 자동으로 `session.begin()`이 실행되고 commit된 코드가 없으니 (정확히 말해서 netsted에서만 commit이 실행되었으니) `session.close()`시 `session.rollback()`을 실행하지 않을까라는 의심이 된다. 후... 재정비하고 다시 도전해봐야 겠다.

## component Decorator

서비스 레이어를 추가하면서 컴포턴들의 라이프사이클을 관리할 필요성이 생기게 되었다. 그냥 static으로 만들어도 무방하지만 되도록 빌드타임을 줄이고 싶었고 메모리에 상주시키는 타임을 최대한 미루고 싶어서 클로저의 캐싱기능을 이용해 보기로 했다.

```python
component 데코레이터 추가

def component(func: Callable) -> Callable:
    components = {}

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        name = func.__name__
        cached_component = components.get(name)

        if cached_component:
            return cached_component

        components[name] = func(*args, **kwargs)
        return func(*args, **kwargs)

    return wrapper
```

## test anti-pattern

현재 테스트를 작성하는 것은 유행처럼 당연시 되고 있다. 개인적으로 좋은 일이라 생각된다. 다만 어떻게 테스트를 잘 작성해야 하는 지에 대한 고민은 필요해 보인다. 테스트는 면죄부가 아니다. 면죄부를 부여하기 위한 테스트 코드는 깨지기 쉽다. 오류를 포함하기 쉽다. 테스트를 잘 작성하는 방법은 오랜 연습을 필요로 한다. 하지만 테스트 시 하지말아야 할 패턴은 익히기 쉽고 연습하기 쉽다. 아래는 안티 패턴에 대한 글들을 모아보았다.

- [https://en.wikipedia.org/wiki/Test-driven_development](https://en.wikipedia.org/wiki/Test-driven_development){: target="\_blank" }
- [http://blog.codepipes.com/testing/software-testing-antipatterns.html](http://blog.codepipes.com/testing/software-testing-antipatterns.html){: target="\_blank" }
- [https://exubero.com/junit/anti-patterns/](https://exubero.com/junit/anti-patterns/){: target="\_blank" }

## Code with me

인텔리제이도 이제 라이브 코드 쉐어링 기능을 제공한다. 재택하면서도 페어프로그래밍 하기 좋겠다.

[https://www.jetbrains.com/help/idea/code-with-me.html](https://www.jetbrains.com/help/idea/code-with-me.html){: target="\_blank" }

## Share enum

Alembic에서 기존에 추가된 enum을 그대로 사용하고 싶을 때 아래와 같이 `create_type=Fasle`를 사용하면 된다. 단 주의할점은 ENUM의 패키지는 SQLAlchemy의 것이 아니라 postgresql의 것이어야 한다. 출처: [https://stackoverflow.com/questions/39023877/sqlalchemy-creating-tables-that-share-enum](https://stackoverflow.com/questions/39023877/sqlalchemy-creating-tables-that-share-enum){: target="\_blank" }

```python
from sqlalchemy.dialects.postgresql import ENUM

receipt_image_group_type = ENUM(
    'a',
    'b',
    'c',
    name='enum_type',
    create_type=False,
)
```
