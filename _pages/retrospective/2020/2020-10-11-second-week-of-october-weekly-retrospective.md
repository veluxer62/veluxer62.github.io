---
layout: single
title: "2020년 10월 2주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-second-week-of-october-weekly-retrospective/
---

10월초는 추석과 명절이 있어 2주가 훌쩍 지나가 버렸다. 1주차에는 도저히 정리할 만한 내용이 없어(~~일안하고 놀았다는 거다~~) 2주차 회고에 한꺼번에 정리하려고 한다. 지난 2주동안 SKU 서비스 개발은 잠시 멈추고 네이버페이 대사 기능 개발을 진행하였다. kotlin을 사용하다가 오랜만에 다시 java를 사용하려고 하니 한동안 버벅되었던것 같다.(~~누가 세미콜론좀 없애줘~~)

이번 네이버 대사 작업에서의 나의 목표는 리펙토링이었다. 솔직히 네이버 대사 기능을 추가하는 작업은 크게 어려운 작업은 아니었다. API만 연동하고 현재 개발되어있는 대사 서비스 로직을 복사해서 사용하면 되었으니까. 하지만 분명 네이버 페이 뿐만 아니라 다른 PG사와의 연동이 추가될 것이고 그때마다 새로운 서비스를 추가한다는 건 불필요한 코드 중복과 리소스가 낭비될 것이 뻔해 보였다. 그래서 빠르게 API 연동을 마무리 짓고 리펙토링 작업을 진행하였다. 약 2주정도 리펙토링을 진행하였는데, 쉽게 생각했던 리펙토링이 테스트 코드와 함께 있는 서비스를 고치려고 하니 상당히 번거로운 작업이 되었다. 이번 작업을 진행하면서 테스트 코드도 꾸준히 리펙토링을 해야 함을 뼈저리게 느끼게 되었고, 무엇보다 유닛테스트도 중요하지만 테스트의 scope를 잘 잡는것이 얼마나 중요한지 알게되었다. 이 내용은 한번 정리해 두면 좋을 것 같으니 꼭 후기를 적어봐야겠다.

아래는 이번 회고때까지 내가 고민했거나 겪었던 이슈들을 적은 것이다.

## RestTemplate 공통 설정

서버를 개발하다보면 특히 MSA를 하거나 외부 API와 연동이 필수인 도메인을 개발하다 보면 RestTemplate에 항상 동일한 코드를 적어줘야 하는 보일러 플레이트 코드를 적어줘야 한다. 예를 들어 HOST 명이라든지, 인증 헤더 선언이라던지 등등.

현재 대사 작업을 하면서 네이버 페이와 연동하는 기능을 개발 중인데 위와 같은 코드중복을 줄이기 위해서 네이버페이만을 위한 RestTemplate Bean을 등록하여 사용하였다. 네이버 페이는 API 전송 시 항상 헤더에 인증값들을 넣어주어야 하는데 Bean을 등록해서 사용하면 인증값을 매번 넣어줘야 하는 번거로움을 없앨 수 있다.

코드 예제

```java
@Component
@RequiredArgsConstructor
public class NaverPayRestTemplateFactory {
    private final NaverPayConfig naverPayConfig;

    @Bean(name = "naverPayRestTemplate")
    public RestTemplate naverPayRestTemplate() {
        HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
        httpRequestFactory.setConnectTimeout(naverPayConfig.getConnectionTimeout());
        httpRequestFactory.setReadTimeout(naverPayConfig.getReadTimeout());

        RestTemplate restTemplate = new RestTemplate(httpRequestFactory);
        restTemplate.setInterceptors(List.of(new NaverPayDefaultHeaderInterceptor()));
        restTemplate.setUriTemplateHandler(new DefaultUriBuilderFactory(naverPayConfig.getHost()));

        return restTemplate;
    }

    @SuppressWarnings("NullableProblems")
    private class NaverPayDefaultHeaderInterceptor implements ClientHttpRequestInterceptor {
        @Override
        public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                                            ClientHttpRequestExecution execution) throws IOException {
            HttpHeaders headers = request.getHeaders();
            headers.add("X-Naver-Client-Id", naverPayConfig.getClientId());
            headers.add("X-Naver-Client-Secret", naverPayConfig.getClientSecret());
            headers.add(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
            return execution.execute(request, body);
        }
    }
}
```

```java
@Getter
@Configuration
@ConfigurationProperties(prefix = "naverpay")
public class NaverPayConfig {
    private final String host;
    private final String clientId;
    private final String clientSecret;
    private final long connectionTimeout;
    private final long readTimeout;
}
```

properties를 테스트할때에는 `EnableConfigurationProperties`를 사용하면 된다.

```java
@ActiveProfiles("test")
@SpringBootTest(classes = {NaverPayWithHeaderRestTemplateFactory.class})
@EnableConfigurationProperties(value = {NaverPayConfig.class})
class NaverPayRestTemplateFactoryTest {

    @Autowired
    private RestTemplate naverPayRestTemplate;

    @Test
    public void test_api() {
        // Given
        final var request = Map.of("paymentId", "12345");
        final var path = "/payments/v2.2/list/history";

        // When
        final var response = naverPayWithHeaderRestTemplate
                .postForEntity(path, request, Map.class);

        // Then
        final var body = Objects.requireNonNull(response.getBody());
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(body.get("code")).isNotNull();
        assertThat(body.get("message")).isNotNull();
        assertThat(body.get("body")).isNotNull();
    }

}
```

## AWS Route53 Traffic Flow

AWS의 Route53에서는 트패픽 흐름관리를 지원해준다. 트래픽 흐름관리는 쉽게 말해서 사용자의 요청을 미리 정해둔 정책에 맞춰 목적지에 도달하도록 조정하는 것이라 할 수 있다. 자세한 내용은 [링크](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/traffic-flow.html){: target="\_blank" }를 참고하자.

현재 회사에서는 백엔드 서버를 AWS Beanstalk에서 EKS로 모두 이전하였다. 안정적인 이전을 위해서 택한 방법이 바로 Traffic Flow인데 처음에 10%의 모든 요청을 EKS로 보내도록 하면서 오류 모니터링을 진행하고 점진적으로 퍼센트를 늘려가는 방식으로 하였다.

한번에 서버 DNS를 변경하여 이슈에 대응하는 방법보다 훨씬 안정적이고 이슈에 대한 대응이 용이하다는 점에서 아주 좋은 방법이라 생각이 든다.

## "진상보존의 법칙, 룰블레이커는 어디에나 있다." 블로그를 읽고

[진상보존의 법칙, 룰블레이커는 어디에나 있다.](https://greypencil.tistory.com/78){: target="\_blank" }라는 글을 회사 동료가 공유해 줘서 읽게 되었다.

이전 회사에서 느끼고 겪었던 문제들을 비슷하게 겪으신 것 같아서 많은 부분이 공감되고 또한 아쉽기도 하였다. 겸손한 시스템, 사용자가 편리한 시스템을 만드는 것이 좋다는 것에 100% 동의한다. 사용자가 없는 시스템은 더이상 시스템이라 할 수 없다. 기술만능주의에 빠져 예쁜 쓰레기를 만드는 것은 팀에게나 회사에게나 모두 독이다.

하지만 너무 서비스에만 치중하는 것 또한 독이라 생각한다. 바쁜 일정에 치여 고민없이 당장 필요한 기능만 개발하기에 급급하다면 결국엔 그 시스템은 위에서 말한 사용자가 편리한 시스템이 되지 못한다. 타협점이 필요하겠지만 좋은 설계와 현재 보이지 않는 문제점을 예측하려는 노력은 아끼지 않아야 한다. 개발자는 코더가 아니다. 생각하고 고민없는 코딩은 모래위에 성을 쌓는 것과 다를 바 없다.

뻔한 얘기일 수도 있겠지만 결국엔 서비스와 기술 사이에 적절한 타협점을 찾는게 중요하고 그것을 찾아서 좋은 서비스를 개발하는 개발자가 좋은 개발자라고 생각한다. 뭐든 다 만족할 순 없다. 하지만 더 좋은 서비스를 위해, 더 좋은 설계를 위해 항상 최선을 다하고 실패를 통해 배워나간다면 점점더 발전 할 수 있지 않을까 생각 된다.

좋은 설계를 위한 노력을 기술자랑으로 치부해 버리지 말았으면 좋겠고, 좀더 나은 서비스를 위한 노력을 기술력의 부족으로 치부하지 말았으면 좋겠다.

## "'개발자 채용시 기술검증 어떻게 할 것인가' 워크샵 참석 후기" 세미나 블로그를 읽고

['개발자 채용시 기술검증 어떻게 할 것인가' 워크샵 참석 후기](https://jojoldu.tistory.com/285#ref=github){: target="\_blank" }라는 글을 면접 가이드 문서에서 보게 되었다.

채용 시 어떤것들을 고민해야 하고 어떻게 면접을 보면 좋은지에 대한 자료를 볼 수 있어서 많은 도움이 되었다. 회사 내 개발자가 부족하다 보니 전사적으로 개발자 채용에 열을 올리고 있다. 단순히 좋은 개발자를 뽑아야 한다라고 생각하기 보다 우리가 지금 어떤 개발자를 원하고 있는지, 어떤 부분을 중점적으로 봐야 하는지를 정하면 좋겠다 라는 생각을 해본다.

## 의존주입

동일한 Bean에게 다른 의존성을 부여하고 싶어서 Configuration을 통해 Bean을 주입하는 방법을 적용하였고 현재 적용해본 결과 아주 만족스럽게 동작함을 볼 수 있었다. 하지만 Annotation을 이용한 의존주입에 익숙해져있다보니 Configuration에서 Bean을 생성하고 주입하는 방식을 도입해 봤는데 조금 어색한 감이 없지않아 있다. 어떤 서비스는 Annotation 주입방식을 사용하고 어떤 서비스는 Configuration 주입방식을 사용하니 일관성이 없어 조금 헷갈리는 것도 문제다.

## Java Interface Default method

자바 8부터 interface에 default method와 static method를 제공한다. (~~최근에야 알게되었다 ㅠㅠ~~)
그러면서 인터페이스를 Abstract class처럼 사용할 수 있게 되었는데 해당 인터페이스를 구현하는 클래스들에게 공통적인 기능을 제공하기 위해서 활용해 보았다. 이때 접근제어자를 사용하면 좋았을 것 같은데 아쉽게도 public으로 밖에 제공되지 않는 것 같다.

생각해보니 내가 의도한 구현은 default method를 사용하는 것이 아닐 수도 있겠다 라는 생각이 들기도 하다.

## Implementation VS Parameters

대사 기능을 리펙토링을 하면서 인터페스를 구현하는 클래스를 여러개를 두는 방법과 매개변수를 추가해서 조건을 추가하는 방법중에 고민을 한 적이 있다. 처음에는 인터페이스에 매개변수를 제한하고 구현체를 늘리는 것이 마냥 좋은 것이라고 생각했었는데, 코드의 중복이 많아지고 클래스가 너무 많아지는 단점이 발견 되어서 적절하게 매개변수를 섞어 줌으로써 해당 문제를 해결하였다. 인터페이스를 잘 둠으로써 확장성을 가져가는 것도 좋지만 적절한 매개변수를 주어 코드의 중복을 줄이는 방법도 중요하다는 것을 배울 수 있는 경험이었다.

## kubectx

[kubectx](https://github.com/ahmetb/kubectx){: target="\_blank" }는 kubectl과 같이 쿠버네티스 명령어 도구이다.
쿠버네티스 공부하면서 알게 되었는데 유용한 명령어가 많으니 사용해보자.

## kubectl completion

shell에서 kubectl 자동완성 기능 적용 방법

- Z shell

  ```
  echo 'source <(kubectl completion zsh)' >>~/.zshrc
  ```

- Bash shell

  ```
  echo 'source <(kubectl completion bash)' >>~/.bashrc
  ```
