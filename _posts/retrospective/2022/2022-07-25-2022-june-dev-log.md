---
layout: single
title: "2022년 6월 개발일지"
categories:
  - RETROSPECTIVE
tags:
  - 개발일지
toc: true
---

말도 많고 탈도 많았던 `주문/채팅 MVP`프로젝트가 마무리 되었다. 4~5개월간 가진 긴 정비기간 이후에 처음으로 하는 프로젝트이다보니 회사에서 기대하는 바가 컸고 그 기대가 큰만큼 요구사항은 많았고 원하는 배포일자는 정해져 있어서 개발기간이 여유로운 상황도 아니었다. 다행히(?) 백엔드는 인원도 가장 많았고 구현해야하는 기능이 생각보다 많지는 않고 복잡하지 않았기 때문에 괜찮았지만 프론트엔드와 앱 개발자 분들은 프로젝트를 진행하는 동안 야근도 많이하시고 샌드버드 연동을 하면서 이슈도 많이 겪으면서 스트레스를 많이 받으셨던 것 같다. 다들 예민해져서 그런지 평소라면 대수롭지 않게 여기며 넘겼을 사소한 이슈들도 큰 이슈로 대두되기도 하였고 조그마한 팀에서 니편 내편과 같은 정치가 펼쳐지는 상황도 생긴 것 같다. 어쩌면 프로젝트를 진행하면서 개발적인 스트레스보다 그 외의 것들로 인한 스트레스가 더 컷던 시기였다고 생각된다. 그래도 프로젝트가 잘 마무리되어서 이전에 있었던 이슈들은 하나의 해프닝(?)으로 끝났지만 지금도 여전히 그 불씨는 남아 있는 것 같다.

프로젝트가 마무리가 되고 지난 프로젝트에 대한 회고를 하였다. 사실 앞에서 프로젝트가 성공적으로 마무리 되었다고 이야기를 하였지만 출혈이 많았던 프로젝트였다고 생각한다. 앱과 프론트엔드 개발자들은 정해진 기간내에 개발을 마무리하기 위해 주말근무와 새벽근무까지 해야했고 QA분들도 QA일정을 맞추기 위해 외부 인력까지 잠시 투입하기도 하였다. 그래서 이번 회고 때 많은 사람들이 지난 프로젝트 과정에 대해 이야기를 할 것이라 예상했었다. 프로젝트를 진행하면서도 문제가 되는 부분들은 회고에서 얘기하자고 종종 말하기도 하였으니 말이다. 하지만 회고에서는 많은 이야기가 나오지 않았다. 어쩌면 문제였다고 생각한게 나만의 생각일 수도 있다는 생각도 들기도 했고 다른 분들이 잘 마무리된 프로젝트에서 굳이 싫은 소리를 꺼낼 필요가 없다고 생각했을 수도, 성향상 다들 모여있는 자리에서 굳이 목소리를 낼 필요성이 없다고 생각할 수도 있을 수도 있다는 생각이 들었다. 어찌되었든 나는 이번 회고에서 많은 이야기가 오고가지 않았다는 점은 위험한 신호라고 생각이 든다. 조직이 적극적인 의사소통이 되지 않고 있다는 뜻일 수도 있으니까.

아래는 6월동안 정리한 이슈 내용들이다.

## uuidgen

[uuidgen](https://man7.org/linux/man-pages/man1/uuidgen.1.html){: target="\_blank" } 은 터미널에서 랜덤한 UUID를 자동으로 생성해주는 도구이다. 매번 [Online UUID Generator](https://www.uuidgenerator.net/version4){: target="\_blank" } 에서 생성했었는데 터미널에서 하는것이 훨씬 편하고 빠르게 UUID를 얻을 수 있으니 앞으로 자주 활용해야겠다.

## 보안 공격

최근 서버의 에러 로그에서 아래와 같은 로그가 자주 보이기 시작했다.

```
UT005023: Exception handling request to /images/..%2finfo.html
org.springframework.security.web.firewall.RequestRejectedException: The request was rejected because the URL contained a potentially malicious String "%2f"
```

이런류의 에러는 Spring Security의 [FirewalledRequest](https://docs.spring.io/spring-security/site/docs/3.2.8.RELEASE/apidocs/org/springframework/security/web/firewall/FirewalledRequest.html){: target="\_blank" }에서 보안적으로 문제가 되는 요청을 오류를 던지도록 처리되어있어서 발생하는 것이었다.

보안엔지니어에게 이런 공격이 들어오는 이유를 물어보았다. 그분의 답변은 아래와 같았는데 결론은 무시하거나 오류에 대한 예외처리를 하는 방법밖에 없는듯 하다.

- 스프링시큐리티 crx 취약점쪽으로 보임
- 1-day attack 시도
- 악의적공격이고..보통은 mass scanning

문제는 Spring Security에서 `RequestRejectedException`가 발생하면 예외를 던져버려서 서버 응답이 실제로 500으로 반환하고 있다는 것이고 500에러가 발생하니 서버에 오류 알림이 계속해서 발생하고 있다는 것이었다. 그래서 예외가 발생해도 알림이 발생하지 않도로고 조치를 해야했다. `RequestRejectedException`예외의 경우 500에러가 아니라 400번대 에러로 반환되도록 해서 처리를 할 수 있지만 왠지 이미 만들어져있는 핸들러가 있을것 같아서 찾아보았다.

[FilterChainProxy](https://docs.spring.io/spring-security/site/docs/3.2.8.RELEASE/apidocs/org/springframework/security/web/FilterChainProxy.html){: target="\_blank" }에서 `RequestRejectedException`이 발생하면 핸들링하는 코드가 있다.

```kotlin
@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
		if (!clearContext) {
			doFilterInternal(request, response, chain);
			return;
		}
		try {
			request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
			doFilterInternal(request, response, chain);
		}
		catch (RequestRejectedException ex) {
			this.requestRejectedHandler.handle((HttpServletRequest) request, (HttpServletResponse) response, ex);
		}
		finally {
			SecurityContextHolder.clearContext();
			request.removeAttribute(FILTER_APPLIED);
		}
	}
```

그래서 `requestRejectedHandler`를 빈으로 등록해주어서 예외를 처리할 수 있을 것 같았다.

다행히(?) [HttpStatusRequestRejectedHandler](https://docs.spring.io/spring-security/site/docs/current-SNAPSHOT/api/org/springframework/security/web/firewall/HttpStatusRequestRejectedHandler.html){: target="\_blank" }라는 구현체가 있어서 이걸 사용하면 되었다. 그래서 아래와 같이 빈을 등록해주었다.

```kotlin
@Bean
fun requestRejectedHandler() = HttpStatusRequestRejectedHandler()
```

## 코드너리

[코드너리](https://www.codenary.co.kr/techblog/list){: target="\_blank" }는 스타트업의 기술 및 블로그들을 보여주는 사이트이다. 블로그들 모아서 보여주는 사이트가 있으면 좋겠다 생각했는데 앞으로 자주 들어가서 봐야겠다.

## hashcode

[java hashcode](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html){: target="\_blank" }는 응용프로그램마다 유일함을 보장해준다. 즉 분산환경에서 hashCode를 쓰면 서버마다 다른 해시값을 제공할 수 있다. 이와 관련된 내용은 [우당탕탕 주문서 개발기](https://veluxer62.github.io/reference/order-sheet-development-story/){: target="\_blank" }에 자세히 적어두었다.

## 배포 방식의 변경

이전에는 개발서버, QA서버, 운영서버에 작업내용을 배포할 때 브랜치에 코드를 병합하면서 자동으로 배포되도록 배포 파이프라인을 설정해 두었다. 우리팀은 QA와 운영서버에 배포할때마다 버전정보를 기입하는데 브랜치에 병합하면 자동으로 배포되도록 해두다 보니 매번 버전정보를 TAG에 남기는것을 잊어버리는 경우가 발생하였다. 그리고 main 브랜치에 병합되면 배포 파이프라인이 실행되다보니 코드는 병합하였지만 굳이 배포를 하지 않고 싶은 경우에 실행되는 파이프라인을 취소하거나 배포 승인을 하지 않는 방법을 활용해야 하는 불편함이 있었다.

이러한 문제점을 해결하기 위해서 우리는 TAG를 생성할 때 배포가 되도록 배포 방법을 변경하였다. QA서버는 운영에 배포할 버전에 `rc`라는 접미사를 추가하는 패턴을 가지므로 `rc` 문자열이 포함된 태그명이 추가되면 QA 서버로 배포되도록 하였다. 예를 들면 `1.0.0-rc1`와 같이 적을 수 있다. QA의 라운드가 새롭게 시작될 때마다 `rc`의 버전은 올라가게 된다. Circle CI의 filter를 보면 아래와 같다.

```
filter_tag_qa: &filter_tag_qa
  tags:
    only:
      - /^\d+\.\d+\.\d+\-rc\d+$/
  branches:
    ignore: /.*/
```

운영서버는 QA서버 보다는 단순하다 그냥 배포할 버전을 그대로 적어주면 된다. 예를 들면 `1.0.0`과 같이 말이다.

```
filter_tag_prod: &filter_tag_prod
  tags:
    only:
      - /^\d+\.\d+\.\d+$/
  branches:
    ignore: /.*/
```

테스트 서버에 배포는 조금 고민거리가 있었다. 처음에는 `test`이름을 가진 브랜치에 코드가 병합되면 테스트 서버에 배포되는 방식이였다가 `test`태그를 추가하면 테스트 서버에 배포되도록 하는 방식으로 바꾸었다. 그렇다보니 테스트 서버에 배포하려면 매번 `test`태그를 force push를 해야하였는데 그러다보니 어떤 코드가 테스트 서버에 배포가 되었는지 확인하기 어렵다는 문제가 있었다. 그래서 다음으로 `test-{배포일시}`를 적어주는 형태로 태그를 생성하도록 정해보았다. 이 방법은 `test`태그를 매번 force push하지 않아도 된다는 장점이 있지만 이것도 역시 배포된 서버의 코드가 어떤 것들이 있는지 직접 태그의 커밋을 하나하나 보아야 한다는 단점을 여전히 가지고 있었다. 그래서 결국 우리는 테스트 서버의 배포는 이전과 동일하게 `test` 브랜치에 코드를 병합하면 배포가 되도록 설정을 롤백하였다.

```
filter_tag_test: &filter_tag_test
  branches:
    only: test
```

## 재택

우연히 [Die, office, die](https://otterletter.com/die-office-die/){: target="\_blank" }라는 글을 보게되었다. 우리회사도 자율재택근무제도에서 3일은 무조건 출근하도록 제도를 바꾸게 되었다. 이유를 물었을 때 돌아오는 답변은 원활한 의사소통을 통한 생산성 향상이었다. 하지만 최근 출근을 하고 프로젝트를 진행하면서 과연 바뀐 근무제도가 우리의 생산성을 끌어올렸을까? 나의 생각은 '아니다'이다. 아래 차트는 최근 6개월간 백엔드 개발자들의 이슈 처리 사이클 타임을 보여주는 그래프이다.

![cycle time](/assets/images/retrospective/backend-cycle-time.png)

붉게 표시한 부분이 재택이 끝나고 주문/채팅 MVP 프로젝트를 진행한 기간인데 작업의 효율성을 나타내는 파란색 선이 상승하는 것을 볼 수 있다. 이는 즉 생산성이 높아진것이 아니라 오히려 떨어졌다고 볼 수 있는 것이다.

회의나 팝업 미팅등과 같은 활동은 재택보다 대면이 훨씬 나아졌으니 작업의 효율성이 떨어졌다고 해도 전체적으로 프로젝트 진행에 대한 효율성은 높아진 것이지 않는가? 라는 이야기를 할 수 있을 것이다. 하지만 나는 이 또한 좋아졌다고 말하기 힘들다 생각한다. 앞서 소개한 글에서 보면 발언권을 가진 사람이 대면회의를 선호한다는 것을 말해주고 있다. 왜냐하면 내가 말하는 것을 듣고 있다는 걸 볼 수 있으니 말이다. 하지만 회의는 한사람만 이야기 하는 자리가 아니다. 모두가 의견을 제시하고 논의하는 자리이다. 특정 내용을 공유하는 회의라면 그냥 문서를 써서 공유하면되고 논의를 하는 자리라면 모두가 그 발언권을 평등하게 가져갈 수 있는 환경이 더 효율적일 것이다. 그런관점에서 온라인 회의보다 오프라인 회의가 더 낫다라는 평가는 나는 동의하기가 힘들다. 물론 화이트보드에 무얼 적어가며 긴밀하게 의사소통을 해야하는 회의는 온라인보다는 오프라인이 효율적일 수 있다. 그렇다면 이런 회의가 필요할 때 출근하는 것을 협의해도 되지 않을까?

## mjml

[mjml](https://mjml.io/){: target="\_blank" }은 반응형 이메일 코딩을 손쉽게 해주는 프레임워크이다. 이메일 전송 시 HTML 코드를 심어야 하는 상황에서 사용하면 유용할 듯 하다.

## github vscode

웹 환경의 github에서 코드를 보는 페이지에서 `.`을 입력하면 온라인 vscode가 열린다. 그럼 손쉽게 코드를 탐색할 수도 심지어는 고칠 수도 있다.

## dynamicUpdate

`DynamicUpdate`를 사용하면 원치않는 컬럼의 업데이트가 발생하지 않도록 할 수 있다. 하지만 업데이트 쿼리를 생성하는 비용이 발생하므로 적절하게 사용하는것이 좋다.

## 클린코드에서 말하는 공백의 중요성

클린코드에서는 수평적 개방성과 밀도(Horizontal Openness and Density)를 이야기하면서 수평 공백을 사용하여 강하게 관련된 것들을 연관시키고 더 약하게 연관된 것들을 연관을 해제하는 것을 말해주고 있다. 나는 이 수평 공백이 가지는 힘이 아주 크다고 생각한다. 왜냐하면 가독성을 아주 손쉽게 끌어올릴 수 있기 때문이다. 그리고 이렇게 연관된 것들을 모아주다보면 리펙터링하면 좋을만한 코드들이 보이기 시작한다. 선순환이 되는 것이다. 코드리뷰에도 이야기 하고 싶은데 너무 주관적인 리뷰로 보일 수 있기 때문에 좀 고민스러운 부분이긴 하다.

## 세너티 테스팅(Sanity testing)

[세너티 테스팅](https://en.wikipedia.org/wiki/Sanity_check){: target="\_blank" }이란 새로운 SW version이 주요 테스팅 업무를 수행하기에 충분히 적합한가를 판단하기 위해 수행하는 테스트를 말한다. 그래서 개발팀 또는 개발자가 테스트 주체가 되어서 주요한 기능들을 테스트 하는 것인데 QA에 입고하기 전에 테스트 하는 테스팅의 이름이 무엇인지 찾다가 발견하게 된 테스팅 기법이다.
