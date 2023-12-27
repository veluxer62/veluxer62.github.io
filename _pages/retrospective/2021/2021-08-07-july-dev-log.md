---
layout: single
title: "2021년 7월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2021-july-dev-log/
---

올해는 작년에 비해 초여름부터 엄청 뜨겁다. 출퇴근할때 잠깐 밖에 걸을 때 조차도 힘든데 밖에서 근무하시는 분들은 얼마나 힘드실까. 학생때 경마공원에서 주차관리 아르바이트를 했던 기억이 있는데, 그때를 기억하면 어떻게 일했나 싶다.

7월부터 코로나 신규감염자수가 갑자기 급증하면서 회사에서 다시 전사 재택을 권장하고 있다. 나에게 있어 풀재택은 마냥 좋게만 느껴지진 않는다. 우선 좋은 점은 출퇴근 시간을 아낄 수 있으니 피로도가 많이 줄어든다는 점과 그 시간 만큼 여유가 있어 다른 것들을 할 수 있다는 것이다. 그리고 감염에 대한 위험성을 덜 걱정해도 되니 마스크를 쓰지 않는다는 등의 편안함도 있다. 하지만 출퇴근시간의 경계가 사라지면서 생활패턴이 깨지는 점은 아쉽다. 내가 의지가 강한 사람이라면 나름 생활계획표를 잘 짜서 지킬테지만... 그러지 못하는 성격이라 잘 지켜지지 않아 이부분은 나에게 단점으로 다가왔다. 패턴이 깨지다보니 출퇴근의 경계가 모호해지고 업무의 집중도가 많이 떨어지는 부분도 부차적으로 생기는 문제점이다. 최근엔 그래서 다시 사무실로 출근하고 있다.

예방접종률이 오르면서 어쩌면 이 긴 대유행이 종식될까 했던 기대감이 어쩌면 좀더 오래 지속되지 않을까 하는 우려가 생긴다. 어쩌면 코로나라는 이 바이러스는 감기처럼 우리 생활에 계속해서 함께할지도 모르겠다. 하루빨리 이 유행이 종식되어서 다시 일상으로 돌아갔으면 좋겠다. 문득 드는 생각인데 코로나가 종식되면 어떤 새로운 서비스들이 유행하게 될까? 그동안 사람들이 억눌러왔던 것들을 표출하게 될텐데 어떤식으로 표출이 될지 지켜보는 것도 재미있을 것 같다.

- [코로나19 팬데믹으로 시작된 대한민국의 변화와 미래](https://zdnet.co.kr/view/?no=20210515200929){: target="\_blank" }
- [코로나 이후 네 가지 주요변화에 주목해야](https://www.korea.kr/news/contributePolicyView.do?newsId=148883026){: target="\_blank" }
- [포스트 코로나19 시대를 위한 변화와 적응](https://www.ibric.org/myboard/read.php?Board=report&id=3718){: target="\_blank" }

이런저런 핑계로 미루어두었던 [빈자리 예약봇 V2](https://github.com/veluxer62/occupying-v2){: target="\_blank" }를 작업하고 있다. 기존에 재미삼아 만들어봤던 걸 구조를 새로 바꾸면서 작업하고 있는데 오랜만에 익숙한 Kotlin과 Spring을 사용해서 개발을 하니 재미가 있다. 평일이나 주말에 틈나는대로 작업을 하고 있다보니 작업 속도가 빠르진 않지만 어서 마무리 지어서 유종의 미를 거두었으면 좋겠다.

최근 콤펙트한 키보드에 계속 눈길이 간다. 최근 봤던 제품중에 마음에 들었던게 [듀가드 퓨전 블루투스 기계식키보드 스팀 저소음 적축](https://www.coupang.com/vp/products/4945355563?itemId=6522590198&vendorItemId=73817074279&src=1191000&spec=10999999&addtag=400&ctag=4945355563&lptag=CFM91783024&itime=20210702140500&pageType=PRODUCT&pageValue=4945355563&wPcid=16225971019382016078617&wRef=&wTime=20210702140500&redirect=landing&isAddedCart=){: target="\_blank" } 인데, 디자인도 레트로하고 유무선을 모두 지원하면서 가격도 나쁘지 않아서 지켜보고 있다. [레오폴드 FC660C](http://www.leaderskey.com/shop/item.php?it_id=1524732401){: target="\_blank" }도 좋은데 무접점도 타건감이 좋아서 매력적이다. 현재는 [바밀로 VA87M 저소음 흑축](https://shopping.interpark.com/product/productInfo.do?prdNo=7257100836&dispNo=016001&bizCd=P14374&utm_medium=affiliate&utm_source=yellomobile&utm_campaign=shop_p14300_p14374&utm_content=price_comparison_coocha_mobile){: target="\_blank" }을 사용중인데 첫 기계식 키보드가 적축이다보니 흑축을 구매하고 싶어서 사보았는데 나름 만족하고 쓰고 있다. 다만 윤활제를 바르지 않으면 다소 키압이 무거울 수 있으니 윤활제를 바르는걸 추천한다.

아래는 7월동안 겪었던 개발 내용들이다.

## graphene-sqlalchemy

현재 회사에서는 Flask 기반 GraphQL을 사용하고 있다. ORM으로는 SQLAlchemy를 사용하고 있고 [graphene-sqlalchemy](https://github.com/graphql-python/graphene-sqlalchemy){: target="\_blank" } 라이브러리는 Graphene과 SQLAlchemy를 아주 손쉽게 연동해서 사용할 수 있도록 기능을 제공해주는 라이브러리이다.

개인적으로는 개발 초기 MVP용으로는 나쁘지 않은 라이브러리라 생각한다. 하지만 제품을 본격적으로 개발하면서 사용하기에는 많은 문제점을 가지고 있는데 그 문제점들에 대해서 적어보고자 한다.

### 레이어가 분리되어 있지 않다.

우리가 API서버를 사용하는 가장 큰 이유 중 하나가 바로 캡슐화라고 생각한다. 내부 데이터(DB, 로직, 상태 등등)를 숨기고 필요한 인터페이스만 제공함으로써 클라이언트의 잘못된 사용으로부터 데이터를 보호하고 숨겨할 데이터는 숨기고 필요한 정보만 제공하는 역할을 하는 것이다. 하지만 graphene-sqlalchemy라이브러리는 sqlalchemy의 Model 정보를 그대로 query field를 통해 노출한다. 물론 신경을 쓴다면 어느정도 노출은 막을 수 있겠으나 `SQLAlchemyObjectType`를 통해 너무도 쉽게 query field를 만들 수 있기 때문에 개발자가 자신도 모르게 Model을 Field로 사용하는 우를 범한다.

레이어 분리가 되어있지 않는 또다른 사례가 있는데, 바로 Presentation Layer에서 SQLAlchemy의 Query를 직접 사용한다는 것이다.
`graphene-sqlalchemy`의 `UnsortedSQLAlchemyConnectionField.resolve_connection`를 보면 아래와 같이 코드가 작성되어 있다.

```python
@classmethod
def resolve_connection(cls, connection_type, model, info, args, resolved):
    if resolved is None:
        resolved = cls.get_query(model, info, **args)
    if isinstance(resolved, Query):
        _len = resolved.count()
    else:
        _len = len(resolved)
    connection = connection_from_list_slice(
        resolved,
        args,
        slice_start=0,
        list_length=_len,
        list_slice_length=_len,
        connection_type=connection_type,
        pageinfo_type=PageInfo,
        edge_type=connection_type.Edge,
    )
    connection.iterable = resolved
    connection.length = _len
    return connection
```

코드를 보면 알겠지만 connectionField를 생성할 때 SQLAlchmey의 Query 결과를 기반으로 한다는 것을 볼 수 있다. 이는 Resovler가 Repository의 책임도 함께 하도록 유도하고 있으며 (심지어 [가이드 문서](https://github.com/graphql-python/graphene-sqlalchemy#examples){: target="\_blank" }에서 조차 예제로 resolver에서 query를 작성하고 있다.) 이는 코드의 중복을 증가시키고 재사용성을 현저하게 떨어뜨리는 문제를 야기한다. 아주작은 어플리케이션에서는 이 방법이 매우 빠르고 실용적일 순 있겠지만, 도메인이 조금만 커지면 resolver 여기저기 산재되어 있는 비슷하거나 동일한 쿼리들을 변경하고자 할 때와 공통 로직을 변경해야할 때 그 고통은 말로 표현할 수 없을 것이다.

### 확장성이 떨어진다.

최근 특정 도메인에서 제공하던 Query를 하나의 테이블에서 두개의 테이블정보를 합쳐서 보여주도록 변경하는 작업을 하고 있다. 기존의 인터페이스를 변경하지 않고 즉, Query Field는 변경하지 않고 내부 로직만 바꾸면 클라이언트 입장에서는 전혀 손을 대지 않고 기능의 변경을 꾀할 수 있는 것이다.

하지만 지금 서버의 코드에서는 위와 같은 방법을 바로 적용하기가 쉽지 않았다. `SQLAlchemyObjectType`은 Model에 종속적이기 때문에 다른 테이블의 정보를 합쳐서 보여주는 것이 어렵다. 왜냐하면 Model은 대부분 하나의 (RDB를 기준으로)테이블 정보를 가지고 있기 때문이다.

또한 위에서 언급한 것 처럼 Query를 기반으로 Connection Field를 만드는 부분으로 인해 두개의 조회 결과를 넘길 수 있는 좋은 방법이 없었다. (`union all`을 사용하여 할 수 있다고 생각하지말자... DB 쿼리 중심적인 설계는 나중에 큰 발목을 잡을 것이다.)

그래서 우리는 기존의 `SQLAlchemyObjectType`을 Graphene의 `ObjectType`으로 변경하는 작업을 선행하고 Connection Field를 새롭게 만들어 SQLAlchemy의 Query에 종속적이지 않도록 변경하는 작업을 하고 있다. 이렇게 변경하고나면 외부 인터페이스를 변경하지 않고도 내부코드(DB 스키마를 포함해서)를 자유롭게 변경할 수 있다.

## Relay Pagination

Connection Field를 새로 만들면서 Graphene의 Connection Field를 뜯어보았는데 조금 의아한 부분들이 있었다. `connection_from_list_slice` 함수를 보면 아래와 같은데,

```python
def connection_from_list_slice(list_slice, args=None, connection_type=None,
                               edge_type=None, pageinfo_type=None,
                               slice_start=0, list_length=0, list_slice_length=None):
    """
    Given a slice (subset) of an array, returns a connection object for use in
    GraphQL.
    This function is similar to `connectionFromArray`, but is intended for use
    cases where you know the cardinality of the connection, consider it too large
    to materialize the entire array, and instead wish pass in a slice of the
    total result large enough to cover the range specified in `args`.
    """
    connection_type = connection_type or Connection
    edge_type = edge_type or Edge
    pageinfo_type = pageinfo_type or PageInfo

    args = args or {}

    before = args.get('before')
    after = args.get('after')
    first = args.get('first')
    last = args.get('last')
    if list_slice_length is None:
        list_slice_length = len(list_slice)
    slice_end = slice_start + list_slice_length
    before_offset = get_offset_with_default(before, list_length)
    after_offset = get_offset_with_default(after, -1)

    start_offset = max(
        slice_start - 1,
        after_offset,
        -1
    ) + 1
    end_offset = min(
        slice_end,
        before_offset,
        list_length
    )
    if isinstance(first, int):
        end_offset = min(
            end_offset,
            start_offset + first
        )
    if isinstance(last, int):
        start_offset = max(
            start_offset,
            end_offset - last
        )

    # If supplied slice is too large, trim it down before mapping over it.
    _slice = list_slice[
        max(start_offset - slice_start, 0):
        list_slice_length - (slice_end - end_offset)
    ]
    edges = [
        edge_type(
            node=node,
            cursor=offset_to_cursor(start_offset + i)
        )
        for i, node in enumerate(_slice)
    ]

    first_edge_cursor = edges[0].cursor if edges else None
    last_edge_cursor = edges[-1].cursor if edges else None
    lower_bound = after_offset + 1 if after else 0
    upper_bound = before_offset if before else list_length

    return connection_type(
        edges=edges,
        page_info=pageinfo_type(
            start_cursor=first_edge_cursor,
            end_cursor=last_edge_cursor,
            has_previous_page=isinstance(last, int) and start_offset > lower_bound,
            has_next_page=isinstance(first, int) and end_offset < upper_bound
        )
    )
```

Pagination arguments로 `before`와 `after`, `first`, `last`를 사용한다는 것이다. [GraphQL Pagination 공식문서](https://graphql.org/learn/pagination/){: target="\_blank" }에서 보면 `offset`과 `first`를 사용하는 것처럼 보이는데 인자가 다르게 정의되어 있는 것이다. 조금 더 찾아보니 문서의 말미에 [GraphQL Cursor Connections Specification](https://relay.dev/graphql/connections.htm){: target="\_blank" }를 따른 것이라고 적혀있는 부분을 발견할 수 있었고 Relay문서에서 보면 `before`, `after`, `first`, `last` 인자들을 사용하는 것을 볼 수 있었다. `offset`은 페이지 기반 페이지네이션을 위해 새롭게 추가된 인자로 보이고 `before`와 `last`는 `Backward pagination`이 불필요하다고 판단해서 제거한게 아닌가 생각된다.

## OCR

사내에서 OCR 도입을 검토하면서 비교해본 라이브러리들을 적어본다.

### Amazon Textract

[Amazon Textract](https://aws.amazon.com/ko/textract/){: target="\_blank" } AWS에서 제공하는 OCR로 데이터 추출 뿐만아니라 기계 학습 기능도 있고 무엇보다 현재 회사에서 AWS를 사용하고 있기에 손쉽게 도입 할 수 있을 것이라는 기대감에 검토를 해보았다. 하지만 한글 지원이 약해서 검토 대상에서 최종적으로 제외되었다.

### OCRmyPDF

[OCRmyPDF](https://github.com/jbarlow83/OCRmyPDF){: target="\_blank" }은 지인의 추천으로 알게된 OCR이다. 무엇보다 오픈소스이기에 무료로 사용할 수 있고 아직 테스트 해보지는 못했지만 지인이 이전에 사용했을 때 나름 만족스럽게 사용했다는 얘기를 해주어서 검토대상에 포함시켰다. 아무래도 오픈소스다 보니 기술지원이 되지 않는 부분이나 유료 서비스에 비해 퀄리티가 좋지 않을 수 있다는 점은 우려스럽다.

### Naver Cloud OCR

한글 지원이 되는 OCR중에 가장 손꼽히는 OCR이 바로 [Naver Cloud OCR](https://www.ncloud.com/product/aiService/ocr){: target="\_blank" }이 아닐까 생각된다. 유로라는 점과 템플릿 시스템이 불편하다는 것 이 단점이다.

## 형태소 분석기

사내 ElasticSearch에서 형태소 분석기로 [Nori](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori.html){: target="\_blank" }를 사용하고 있다. 사실 형태소 분석기에 대한 지식이 거의 없어서 종류별로 어떤 차이점이 있는지 정확히 알지는 못하는데 새로운 형태소 분석기들에 대한 정보를 얻게되어 적어본다.

- [https://bitbucket.org/eunjeon/mecab-ko-dic/src/master/](https://bitbucket.org/eunjeon/mecab-ko-dic/src/master/){: target="\_blank" }
- [https://konlpy.org/en/latest/](https://konlpy.org/en/latest/){: target="\_blank" }

## WebClient timeout 설정

최근 Webflux로 토이프로젝트를 하면서 WebClient를 사용했었는데, timout 설정을 하면서 적용했던 코드를 적어본다. RestTemplate의 타임아웃 설정에 비해 조금 코드가 많은 편이긴 하다.

```kotlin
WebClient.builder()
    .baseUrl("http://localhost")
    .clientConnector(
        ReactorClientHttpConnector(
            HttpClient.create().responseTimeout(Duration.ofMillis(korailProperties.timeout))
        )
    )
    .build()
```

## MockServer

[Kotest](https://kotest.io/){: target="\_blank" }로 테스트 코드를 작성하면서 Mock을 자주 사용한다. Kotest는 Mockk를 이용한 API Mockking을 하는 방법 외에도 [MockServer extension](https://kotest.io/docs/extensions/mockserver.html){: target="\_blank" }을 제공해주는데 MockServer를 이용하면 테스트 코드를 아주 심플하게 짤 수 있는 장점이 있다.

```kotlin
class MyMockServerTest : FunSpec({
    listeners(MockServerListener(1080), MyMockServerListener())

    test("login test") {
        val sut = Client()
        val actual = sut.login("id", "pw")
        actual.status shouldBe OK
    }
})

class MyMockServerListener : TestListener {
    override suspend fun beforeTest(testCase: TestCase) {
        MockServerClient("http://localhost", 1080).`when`(
            HttpRequest.request()
               .withMethod("POST")
               .withPath("/login")
               .withHeader("Content-Type", "application/json")
               .withBody("""{"username": "id", "password": "pw"}""")
         ).respond(
            HttpResponse.response()
               .withStatusCode(200)
         )
    }
}
```

## Custom Codecs

개인 프로젝트를 하면서 연동하는 API의 응답값이 json 문자열을 반환함에도 불구하고 미디어타입이 `application/json`이 아니라 `text/plain`인 API가 있었다.(~~그럴수 있지.~~) 그래서 WebClient의 응답값을 DTO로 반환하도록 설정하면 오류가 발생하였다. 그냥 String 타입으로 반환하도록 해서 jackson의 ObjectMapper를 이용해서 파싱을 해줄 수도 있지만 해당 API를 사용하는 곳마다 ObjectMapper를 사용하는 방법은 아주 좋지 못한 방법이라 생각된다.

[Spring Webflux 문서](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/web-reactive.html#webflux-codecs-custom){: target="\_blank" }를 보면 기본 Codec이 아닌 추가적인 미디어타입을 지원할 수 있도록 Custom Codecs를 제공해준다. `Jackson2Decoder`가 `text/plain` 미디어 타입도 DTO로 변환할 수 있도록 하는 Codec을 추가한다면 WebClient를 이용하여 외부 API를 연동할 때 따로 변환코드를 넣어주는 불필요한 중복을 방지할 수 있을 것이다. 변환하는 예제 코드는 아래와 같다.

```kotlin
WebClient.builder()
    .baseUrl(korailProperties.host)
    .codecs {
        it.defaultCodecs()
            .jackson2JsonDecoder(
                Jackson2JsonDecoder(jacksonObjectMapper(), MimeTypeUtils.TEXT_PLAIN)
            )
    }
    .build()
```

## jacoco-badge-generator

[jacoco-badge-generator](https://github.com/marketplace/actions/jacoco-badge-generator){: target="\_blank" }는 Git Action을 이용하여 Jacoco 결과를 Repository 내 뱃지를 달아주는 기능이다. 현재 배포버전의 커버리지를 쉽게 공유할 수 있는 기능이라 좋아 보인다. 개인프로젝트에 적용해서 써봐야 겠다.

## Jira 사용 시 겪었던 이슈

### sub task

sub task는 말 그대로 하위 작업이다. story의 task 용도로 사용하면 나중에 번다운 차트나 번업 차트 등 리포트를 생성할때 estimate값을 불러오지 못한다.

### component

component는 서브 프로젝트 용도로 사용할 수 있는데 epic을 그루핑 해주는 용도로 사용하기에 나쁘지 않다. 하지만 상위 티켓으로 사용되는 것이 아니기에 불편함이 있다.

### 계층

결국 티켓의 계층은 epic과 story(task, backlog 등) 두 계층으로만 나눌 수 있다. 이슈링크를 통해 계층을 표현할 순 있지만 결국 링크일 뿐이라 유연함은 있지만 가시적이지 않다.

## cucumber

Kotest 블로그를 쓰다가 BDD 테스트 도구로 [cucumber](https://cucumber.io/){: target="\_blank" }를 발견하게 되었다. 개발자 친화적이지 않은 도구이길 바랬는데 아니어서 조금 아쉽긴 하지만 BDD에 최적화 되어 있다보니 테스트 실행결과를 표시하는 방식이 상당히 매력적이다.

## jacoco gradle plugin document

이전에 jacoco를 설정하면서 [우아한 형제들 기술블로그: Gradle 프로젝트에 JaCoCo 설정하기](https://techblog.woowahan.com/2661/){: target="\_blank" }를 참고해서 설정을 했었다. 당시 설정할때 다소 코드가 많았던 것 같은데 얼마전에 [gradle plugin document](https://docs.gradle.org/current/userguide/jacoco_plugin.html){: target="\_blank" }에서 jacoco 설정과 관련한 문서를 보았는데 단순한 설정으로 jacoco를 설정할 수 있는 것을 볼 수 잇었다. gradle + kotlin 설정도 가이드 하고 있으니 앞으로 jacoco 설정은 이 문서를 참고하자.

## git case sensitive issue

git 사용시 파일의 대소문자를 구분하지 못해서 커밋 시 이슈가 생긴 경우가 있었다. 이런 경우 `git mv`명령어를 사용하면 되니 기억해 두자. 참고: [https://stackoverflow.com/questions/17683458/how-do-i-commit-case-sensitive-only-filename-changes-in-git](https://stackoverflow.com/questions/17683458/how-do-i-commit-case-sensitive-only-filename-changes-in-git){: target="\_blank" }

## 아르고 CD

최근 [아르고 CD](https://crossplane.io/){: target="\_blank" }에 대한 글들을 많이 볼 수 있다. Git을 이용한 배포버전관리가 유행하면서 각광받는 것 같은데 CI도구를 이용하여 CD를 함께 사용하는 경우들이 많은데 아직 사용해보진 않았지만 아르고 CD를 함께 이용해보면 좋을 것 같은 생각이 든다.
