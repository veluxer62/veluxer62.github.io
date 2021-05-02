---
layout: single
title: "2021년 4월 개발일지"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

어느덧 현재 회사에 입사한지 1달을 넘어 2달을 향해 달려가고 있다. 그동안 4번의 스프린트를 마치고 다음주부터는 5번째 스프린트를 진행을 앞두고 있다. 아직 다 적응했다고 말하긴 힘들지만 회사도 개발내용도 점점 적응해 가고 있다.

Python을 사용하게 되면서 어느정도 예상은 했지만 Python이 추구하는 개발 방향과 내가 이전에 사용해오던 자바/코틀린의 방향이 좀 달라서 혼란스러운 부분이 있다. 개발하면서 한번씩 왜이렇게 되었을까 하는 의문과 거부감도 조금 있었다. 하지만 내가 아직 언어에 대한 이해도가 부족하다고 생각하고 좀더 Python을 공부해보면서 이해력을 넓혀 가기로 하였다. 정적언어만 사용해왔는데 동적언어가 가지는 장점을 느껴보는것도 나쁘지 않으리라 생각된다. (~~자바스크립트도 어드민을 개발하면서 사용해왔지만 제대로 사용했다고 할 순 없다.~~)

현재 회사에서는 내가 이전에 겪었던 회사들에 비해 코드리뷰에 좀더 신경을 많이 쓴다. PR을 올릴때마다 적게는 50개, 많게는 100개까지 리뷰 코멘트가 달리기도 한다. 가끔 답답함을 느낄때도 있지만 개발자들이 내가 작업한 코드에 시간을 투자해서 꼼꼼하게 리뷰를 봐주는 것은 참으로 고마운 일이고 Python이 익숙하지 않은 나에게 페어프로그래밍 없이 가장 손쉽게 코드를 적응할 수 있도록 도와주는 활동이기도 하다. 다만 더 좋은 코드리뷰 환경을 조성하기 위한 고민은 필요해 보여 고민할 필요는 있어보인다. PR이 클수록 코드리뷰에 시간을 많은 시간을 쓰고 제대로된 코드리뷰가 힘들어 질 수 있다.

현재 회사는 유연근무제를 시행하고 있고 재택도 자율이다. 그날그날 컨디션에 따라서 출근시간과 퇴근시간을 조절할 수 있고 재택도 원한다면 언제든 해도 된다. 유연근무제에서 내가 느낀 가장 큰 장점은 사무실에서 근무하다가 집중력이 떨어지거나 자택으로 돌아가서 일을 하고 싶으면 언제든 그럴 수 있기 때문에 근무시간 대비 생산성이 아주 좋다는 것이다. 이전에 어떻게 근무했나 싶다. 자율재택도 아주 만족하는 제도이다. 만약 집에서 근무를 하기 좋은 환경이었다면 재택근무를 자주 했을듯 하다. 다만 회사에서 사람들과 마주치며 유대감을 키우는게 쉽지 않아서 이부분은 조금 아쉽긴 하다.

아래는 4월동안 겪었던 개발 내용들이다.

## postgresql explain

서비스중인 서버의 데이터가 쌓이면서 점점 조회 성능이 떨어지고 DB의 부하가 증가하는 현상을 겪게 되었다. DB에서 사용되는 쿼리가 문제라고 판단되어 `explian`으로 조회쿼리의 실행계획을 분석하게 되었다. 쿼리 분석에 대한 설명은 [Postgresql 문서](https://postgresql.kr/docs/11/sql-explain.html){: target="\_blank" }를 참고하자.

## GIT ignore

GIT ignore는 IDEA마다 프로젝트마다 다르다. 깃헙에서 운영하는 레파지토리에서 이러한 gitignore 파일들을 모아놓고 사용할 수 있도록 제공하고 있다. [https://github.com/github/gitignore](https://github.com/github/gitignore){: target="\_blank" }를 참고하자.

## Jakarta bean validation

자바에서 사용하는 Validation Spec을 정의해놓은 사이트가 있어 적어둔다. [https://beanvalidation.org/](https://beanvalidation.org/){: target="\_blank" }를 참고하자.

## TDD의 이점

회사에서는 구현코드 작성 이후에 테스트코드를 작성하고 있다. 이는 TDD로 개발하는 것이 아닌데 테스트코드를 작성한다라는 부분에서는 꼭 좋지 않다고는 할 수 없다고 생각한다. 어찌되었든 우리가 만들 서버의 안정성을 어느정도 보장해주니 말이다. 하지만 나는 TDD가 가지는 장점이 분명이 존재한다고 생각하고 있고 동료들에게 뭔가 말로써 제대로 설명하기가 어려워 TDD를 잘하시는 이전 직장 동료분께 여쭤보고 TDD를 했을 때의 이점에 대해 내용을 정리해 본다.

- 스토리를 짜면서 테스트 케이스를 적기때문에 내가 무엇을 작업해야할지 생각해보고 구상해볼 수 있는 시간이 있다.
- 엣지 케이스가 보인다. 사후 테스트도 엣지 케이스를 적을 수 있지만 TDD로 하면 좀더 엣지케이스를 고민할 수 있는 확율이 올라간다.
- 쓸데없는 행동이 준다. 테스트를 통과하는 작업만 하면 되기 때문에 쓸데없는 코드가 줄어든다.
- 피드백을 빨리 받다보니까 더 좋은 설계를 할 수 있다. (테스트가 복잡하면 디자인이
- 잘못되었는지 볼 수 있다. - 테스트가 잘된다고 디자인이 좋은건 아니다)
- TDD를 할때 사후 테스트 보다 코드가 훨씬 깔끔해진다.

## ElasticSearch text vs keyword

ElasticSearch에서 문자열을 나타내는 대표적인 타입이 `text`와 `keyword`가 있다.
`text` 타입은 전체 일치 검색 뿐만아니라 분석기를 둠으로써 본문 안의 각 단어를 검색할 수 있는 이른바 `Full text search`가 가능하다. <br/>
`keyword` 타입은 정형화된 문자열 컨텐츠를 색인하는 경우 사용하는데 정렬이나 집계를 이용할때 많이 사용한다. (`text`타입도 `fielddata` 속성을 사용하여 집계용도로 사용할 수 있으나 메모리 사용량이 높아지므로 추천하지 않는다.)

참고 : [https://esbook.kimjmin.net/07-settings-and-mappings/7.2-mappings/7.2.1](https://esbook.kimjmin.net/07-settings-and-mappings/7.2-mappings/7.2.1){: target="\_blank" }

## ElasticSearch의 aggregation에서 전체 건수를 조회하는 쿼리

ElasticSearch에서 aggregation의 전체 건수를 조회하려면 `cardinality`를 사용하면 된다.

```json
{
  "aggs": {
    "byCategory": {
      "terms": {
        "field": "category_keyword.keyword"
      }
    },
    "total": {
      "cardinality": {
        "field": "category_keyword.keyword"
      }
    }
  }
}
```

## ElasticSearch의 aggregation에서 페이징 쿼리

ElasticSearch에서 페이징을 사용하려면 `bucket_sort`를 사용하면 된다. 이때 주의해야할 부분은 `terms`의 `size`를 예상되는 조회값 만큼 주어야 한다는 것이다. 만약 기본값인 10으로 주면 페이지 사이즈가 10인 첫번째 페이지 이후 두번째 페이지에서는 데이터가 조회되지 않을 것이다.

```json
{
  "aggs": {
    "data": {
      "terms": {
        "field": "category_keyword",
        "size": 999
      },
      "aggs": {
        "paging": {
          "bucket_sort": {
            "from": 0,
            "size": 10
          }
        }
      }
    }
  }
}
```

## Python `__init__`과 `__all__`

Python에서 해당 디렉토리가 패키지임을 알려주는 용도(?)로 디렉토리 내부에 `__init__.py` 파일을 생성한다. 디렉토리 내에 해당 파일이 존재하지 않으면 패키지로 인식되지 않을 수도 있다고 하니 추가해 두는 습관을 가지자. (~~파이썬을 주로 다루는 동료의 얘기를 들어보니 요즘에는 잘 안쓰는 추세라고 한다. 일단 보수적으로 판단하자.~~)

Python에서 특정 디렉터토리의 모듈을 와일드카드문자(`*`)를 이용하여 import하기 위해서 `__all__` 변수를 활용한다. 현재 사내에서는 와일드카드 문자를 잉요한 import를 지양하고 있으므로 해당 코드가 보이면 보이는 족족 제거하고 있다.

## Python List Comprehension

코드리뷰를 하면서 `map`과 `filter`를 이용하여 배열의 값을 다루는 코드를 작성했었는데 동료분께서 `List Comprehension`을 사용해 보는게 어떤지 제안해 주셔서 사용해 보게 되었다. 문법이 익숙하지 않지만 적제적소에 사용한다면 아주 깔끔한 코드를 작성할 수 있어 보여 자주 활용해보아야 겠다. (추가로 지인얘기로는 List Comprehension이 성능적으로도 이점이 있다고 들었다.)

## ElasticSearch term vs match

ElasticSearch에서 Query에 사용하는 filter중에 `term`과 `match`가 있다.

`term`은 검색어에 대한 분석을 수행하지 않고 온전히 그 검색어와 일치하는 문서를 검색할때 사용한다. <br/>
`match`는 분석기를 통해 토크나이징된 단어들을 검색하는 용도로 사용한다.

## Pycharm에서 pytest 설정 방법

1. `settings` -> `python integrated tools` -> `default test runner`를 `pytest`로 설정
2. `edit configuration` -> `edit: configuration templates` -> `python tests` -> `pytest` 템플릿 선택
3. `working directory`를 `Project root`로 설정

## ElasticSearch aggregation 유사도 정렬

query 사용 시 tokenizer의 유사도에 따라 index가 나오지만 aggs를 쓰는 순간 정렬이 중복개수순으로 정렬이 된다. 이런경우 유사도에 따라서 정렬을 할 수 있지 않을까 하는 생각이 들었는데 aggregation 시 유사도를 max값을 추출하고 해당 max값을 기준으로 정렬을 하면되도록 하니 잘되었다. sum이 아니라 max를 한 이유는 중복개수가 많은것이 상위로 노출이 되는것을 방지하기 위해서 이다.

```json
GET _search
{
  "query": { "match": { "name": "E2E-품목명" } },
  "aggs": {
    "data": {
      "terms": {
        "field": "name_keyword",
        "size": 2147483647,
        "order": {
          "maxScore": "desc"
        }
      },
      "aggs": {
        "paging": {
          "bucket_sort": {
            "from": 0, "size": 10
          }
        },
        "maxScore": {
          "max": {
            "script": {
              "source": "_score"
            }
          }
        }
      }
    },
    "total": { "cardinality": { "field": "name_keyword" } }
  }
}
```

## Postgresql의 FK는 인덱스가 자동적용되지 않는다.

MariaDB와 달리 Postgresql은 기본적으로 FK에 인덱스를 적용하지 않는다고 한다. 이유는 아직 명확히 모르겠다. 찾아봐야 하겠지만 아무튼 join 성능이나 현재 개발되어 있는 서비스에서는 fk를 검색조건 대상으로 하는 경우들이 많아 인덱스를 줘야 할 필요성이 있어 보였다. (인덱스가 실제로 적용되지 않는지에 대한 실험은 별도의 블로그로 적어보려고 한다.)

## 매개변수(parameter)와 인수(arguments)

매개변수와 인수를 혼용해서 사용하는 경우가 많다고 한다. (~~나를 포함해서~~) 그래서 용어를 다시한번 적어보면서 머릿속에 기억해두려고 한다.

- `매개변수(parameter)`는 함수에 입력으로 전달된 값을 받는 변수를 의미
- `인수(arguments)`는 함수를 호출할 때 전달하는 입력값을 의미

## ElasticSearch max_buckets

개발서버에서 잘되던 aggregation 쿼리가 QA서버에서는 아래와 같이 오류가 발생하는 현상이 있어 확인해보니 ElasticSearch의 버킷 수 제한 설정 때문인것으로 확인이 되었다.

```json
{
  "took": 20,
  "timed_out": false,
  "_shards": {
    "total": 10,
    "successful": 9,
    "skipped": 0,
    "failed": 1,
    "failures": [
      {
        "shard": 0,
        "index": "cart_product_2021.04.21",
        "node": "0f3zN3aKRaiAjis76RqNsg",
        "reason": {
          "type": "too_many_buckets_exception",
          "reason": "Trying to create too many buckets. Must be less than or equal to: [10000] but was [10001]. This limit can be set by changing the [search.max_buckets] cluster level setting.",
          "max_buckets": 10000
        }
      }
    ]
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "total": {
      "value": 0
    },
    "data": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": []
    }
  }
}
```

ElasticSearch에서는 기본으로 `100000`의 최대 버킷수로 제한하고 있다.

최대 버킷 수 설정 방법은 아래와 같다.

```json
PUT _cluster/settings
{
  "transient": {
    "search.max_buckets": 20000
  }
}
```

## CircleCI GIT Hook 이슈

CircleCI를 사용하다가 갑자기 Git Hook의 모든 요청이 400오류를 반환하며 실패하는 현상을 겪었다. 정확한 원인은 파악하지 못했는데 CircleCI에서 해당 repository를 `unfollow`했다가 다시 `follow`하니 해결되었다.

[참고했던 글](https://teratail.com/questions/319236){: target="\_blank" }

## 파이썬에서 서비스 객체를 사용할 때 어떻게 생성할지에 대한 고민

아직 제대로 찾아보진 않았는데 아래 후보군들이 있다. 각자 장단점이 있어서 어떻게 할지 고민이다. 스프링처럼 IoC 컨테이너가 있으면 좋겠는데 그런게 있는지도 찾아봐야겠다.

- [https://www.slideshare.net/subhashbhushan/clean-architecture-applications-in-python](https://www.slideshare.net/subhashbhushan/clean-architecture-applications-in-python){: target="\_blank" }
- [https://www.kenneth-truyers.net/2014/11/19/using-partial-application-for-dependency-injection/](https://www.kenneth-truyers.net/2014/11/19/using-partial-application-for-dependency-injection/){: target="\_blank" }
- [https://blog.ploeh.dk/2017/01/30/partial-application-is-dependency-injection/](https://blog.ploeh.dk/2017/01/30/partial-application-is-dependency-injection/){: target="\_blank" }

## Terraform RDS 사양 변경 이슈

사내에서는 Terraform을 이용하여 AWS의 자원을 관리한다. 최근 RDS의 사양을 변경해야할 필요가 있어 변경 적용을 하였는데 왠일인지 변경이 되지 않았다. 처음에는 수동적용한 적이 있어 그 이유로 되지 않는게 아닐까 라는 의심을 했었는데 알고보니 Terraform에 설정 중 즉시적용이 기본값으로 off로 설정되어 있음을 알게 되었다.

예약변경으로 하는게 좋긴하지만 아무래도 모니터링 등 이유로 사용자가 적은 시간에 즉시 적용하는게 좋아 보이니 팀원과 협의 후 값을 변경할지 검토해 봐야 겠다.

참고 링크 : [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance){: target="\_blank" }
