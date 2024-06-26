---
layout: single
title: "2023년 5월 개발일지"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2023-may-dev-log/
---

한해 중 행사가 많은 달을 꼽자면 5월이라 생각한다. 가정의 달이기도 하지만 특히 우리가족은 5월 생일자가 많기 때문에 더욱 행사도 많았던 것 같다. 거기에 일본 여행도 다녀오기도 하였고 코로나가 끝나고 다들 미뤄두었던 결혼식을 하려고 청첩장을 나눠주는 약속도 많이 생기기도 하였다.

![2023-may-calendar](/assets/images/retrospective/2023-may-calendar.jpg)

5월 달력을 보면 약속이 없는날보다 약속 있는날이 더 많은걸 볼 수 있다.

<br/>

일본 여행을 다녀왔다. 4년전 아내 친구 결혼식에 참가하려고 도쿄에 간게 처음이자 마지막 일본 방문이었는데 코로나가 잠잠해지면서 아내의 친구의 초대로 다시 도쿄를 방문하게 되었다. 4년 사이에 아내의 친구는 애기를 낳았고 그 애기가 벌써 3살이나 되어서 의사소통이 가능할 정도로 컸다. 나는 언제 애기를 낳아 기를까 걱정부터 되는데 다른 사람의 애기는 어떻게 그렇게 무럭무럭 자라는지 모르겠다.

이전에 일본 여행을 갔을 때 일본인들이 영어를 잘하지 못해서 의사소통에 어려움이 있었던 기억이 있었다. 그래서 아내는 30일 단기 일본어 학습 책을 구매해서 열심히 공부하였다. 여전히 파파고 앱을 사용해서 의사소통을 해야했지만 그래도 공부한 덕(?)을 조금씩 보긴 하였다. 하지만 관광지가 아닌 로컬 음식점만 찾아가도 메뉴 주문부터 엄청난 어려움을 겪었기에 다음번에는 좀 더 일본어 공부를 열심히 해서 방문해야겠다는 생각이 들었다. (애니메이션이라도 봐야하나....)

4년만에 찾은 일본은 많은 부분이 바뀌었다. 우선 현금 위주로 결제를 하던게 애플페이를 이용하여 좀 더 손쉽게 거래를 할 수 있었다. 여전히 로컬음식점은 현금을 달라고 하였지만 예전에 비하면 엄청나게 나아진 것 같았다. 그리고 4년전엔 처음 방문이라 이것저것 많은 음식들을 시도해보았는데 이번은 꼭 먹고싶은 음식들과 광광지가 아닌 로컬 음식점 위주로 찾아가려고 노력했다. 여행 중 가장 맛있었던 음식을 꼽아보자면 야키토리라는 꼬치구이와 레몬 사와라는 일본식 칵테일이었다. 여전히 음식이 전반적으로 짜다는 느낌을 받았지만 편의점 간식들은 지금의 한국 편의점 음식들이 아무리 상향 평준화 되어 있다고 해도 그 맛을 따라가지 못한다는 느낌을 받았다.

4년전엔 [도쿄 디즈니 씨](https://www.tokyodisneyresort.jp/kr/tds/){: target="\_blank" }를 방문했었는데 이번엔 [도쿄 디즈니랜드](https://www.tokyodisneyresort.jp/kr/tdl/){: target="\_blank" }를 방문했다. 디즈니 씨는 놀이기구 위주의 놀이동산이라면 디즈니랜드는 컨텐츠 위주의 놀이동산이라고 말할 수 있을 것 같다. 어떻게 이렇게 디테일을 잘 살렸는지 보는 재미가 쏠쏠했다.

비가와서 아쉬운점도 있었지만 이번 도쿄여행도 알차고 재미있게 다녀온 것 같다. 다음 일본 여행은 언제가 될지 모르겠지만 오사카를 한번 가서 유니버셜 스튜디오를 한번 가보고 싶다.

<br/>

아래는 5월동안 정리한 이슈 내용들이다.

## sudo로 Spring Boot 어플리케이션 실행 시 이슈

sudo 로 `./gradlew bootRun`을 실행한 이후 변경사항이 있고 IDEA에서 컴파일을 수행하면 컴파일 에러가 발생한다. 

![build-error](/assets/images/retrospective/build-error.png)

이유를 보니 sudo로 build한 파일을 지울 수 없어서인듯하다. 그래서 sudo로 `./gradlew clean`을 수행해주면 에러를 해결할 수 있다.

## Spring Boot Built-In Request Logging Filter

제품을 만들다보면 로깅을 위한 Filter를 구현해야하는 경우가 종종있다. 이때까지는 대부분 직접 구현해서 사용했었는데 최근에 Built-In Filter가 있다는 것을 알게되었다.

바로 [CommonsRequestLoggingFilter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/CommonsRequestLoggingFilter.html){: target="\_blank" }와 [ServletContextRequestLoggingFilter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/ServletContextRequestLoggingFilter.html){: target="\_blank" } 이다.

해당 필터를 사용하면 SevletRequest를 따로 래핑하는 번거로운 작업 없이 손쉽게 로깅을 구현할 수 있다.

참고: [https://www.baeldung.com/spring-http-logging](https://www.baeldung.com/spring-http-logging){: target="\_blank" }

## ContentCachingResponseWrapper response 이슈

로깅을 위해 [ContentCachingResponseWrapper](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/ContentCachingResponseWrapper.html){: target="\_blank" }의 `contentSize`를 호출하는 코드를 작성하게 되었다. 해당 함수를 호출하면 실제 응답에서 response가 null이 되는 이슈가 있었는데, 이를 해결하기 위해서는 contentSize를 호출 후 반드시 copyBodyToResponse를 사용해주어야 한다. 다만 이러한 코드는 코드 작성자가 놓치기 좋기 때문에 아래와 같이 Wrapper 클래스를 만들어두면 좋다.

```kotlin 
class AutoCopyContentCachingResponseWrapper(response: HttpServletResponse) : ContentCachingResponseWrapper(response) {
    override fun getContentSize(): Int {
        return super.getContentSize().also {
            super.copyBodyToResponse()
        }
    }
}
```

## Collections.synchronizedList

Thread Safe하게 List를 다루려면 [Collections.synchronizedList](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedList-java.util.List-){: target="\_blank" }를 사용하면 좋다.

## 민감정보 직렬화 마스킹 방법

로깅을 할때 비밀번호와 같은 민감한 정보는 마스킹 처리해야할 경우가 있다. 이럴 경우 매 로깅 코드에다가 replace하는 방법보다는 직렬화 할 때 마스킹하도록 처리하는 방법을 사용하면 좋다. 이렇게 하면 실제 JSON 문자열로 반환할 때에도 마스킹처리할 수도 있다.

```kotlin
@Target(AnnotationTarget.FIELD)
annotation class Mask

object MaskStringValueSerializer : StdSerializer<String>(String::class.java), ContextualSerializer {
    override fun serialize(value: String, gen: JsonGenerator, provider: SerializerProvider) {
        gen.writeString("**********")
    }

    override fun createContextual(prov: SerializerProvider, property: BeanProperty): JsonSerializer<*> {
        val annotation = property.getAnnotation(Mask::class.java)
        if (annotation != null) {
            return MaskStringValueSerializer
        }

        return prov.findKeySerializer(property.type, property)
    }
}

object MaskJacksonAnnotationIntrospector : JacksonAnnotationIntrospector() {
    override fun findSerializer(a: Annotated): Any? {
        val annotation = a.getAnnotation(Mask::class.java)
        if (annotation != null) {
            return MaskStringValueSerializer::class.java
        }

        return super.findSerializer(a)
    }
}

data class LoginWithIdPasswordRequest(
    val username: String,
    @Mask
    val password: String,
)

class LoginWithIdPasswordRequestTest : FunSpec() {
    private val objectMapper = jacksonObjectMapper()
        .apply { registerModule(JavaTimeModule()) }
        .apply { setAnnotationIntrospector(MaskJacksonAnnotationIntrospector) }

    init {
        context("serialize") {
            test("데이터가 주어지면 올바르게 직렬화 한다.") {
                // Given
                val request = LoginWithIdPasswordRequest(
                    username = "test",
                    password = "password",
                )

                // When
                val actual = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(request)

                // Then
                actual shouldBe """
                    {
                      "username" : "test",
                      "password" : "**********"
                    }
                """.trimIndent()
            }
        }
    }
}
```

## ControllerAdvice 이슈

`@ControllerAdvice`를 이용해서 WEB Request에 대한 에러 핸들링을 수행하곤 한다. 이 때 Exception Handelr를 정의해놓았음에도 불구하고 Spring 내부에서 발생하는 예외(예를 들면, 주어진 Request Body의 값이 정의한 DTO와 일치하지 않는 경우)는 무조건 500 error가 발생하는 상황을 겪에되었다. 

해결 방법을 찾아보니 [ResponseEntityExceptionHandler](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/ResponseEntityExceptionHandler.html){: target="\_blank" }를 상속해줌으로써 문제를 해결할 수 있다고 한다.

```kotlin
@ControllerAdvice
class ExceptionHandler : ResponseEntityExceptionHandler() {
    @ExceptionHandler(IllegalArgumentException::class, IllegalStateException::class)
    fun handleBadRequest(exception: RuntimeException): ResponseEntity<CommonResponse<Unit>> {
        return ResponseEntity.badRequest()
            .body(CommonResponse.badRequest(exception.message))
    }
}
```

## 기능 테스트 시 RestTemplate 이슈

RestTemplate으로 기능 테스트를 수행하다가 로그인 테스트에서 예상치 못하게 실패하는 경우를 발견하게 되었다. 특이한점은 단일 클래스를 테스트하면 성공하는데 전체 테스트를 수행하면 테스트를 실패하는 것이었다. Debugging 모드로 테스트용 RestTemplate을 보니 `cookie store`에 `sessionid`가 의도치 않게 저장되어있는것을 볼 수 있었다.

![test-reste-template-debugging](/assets/images/retrospective/test-reste-template-debugging.png)

그래서 위 문제를 해결하기 위해서 테스트용 RestTemplate에서 `cookie store`를 사용하지 않도록 아래와 같이 설정하여 해결하였다.

```kotlin
fun testClient(@Value("\${local.server.port}") port: Int): RestTemplate {
    val client = HttpClients.custom().disableCookieManagement().useSystemProperties().build()
    return RestTemplateBuilder()
        .rootUri("http://localhost:$port")
        .errorHandler(NoThrowErrorHandler)
        .requestFactory { HttpComponentsClientHttpRequestFactory(client) }
        .build()
}
```

사실, 테스트할 때 이런류의 이슈를 해결하기 위해서 Spring에서 [TestRestTemplate](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/web/client/TestRestTemplate.html){: target="\_blank" }이라는 구현체를 만들어 두었다.

설정을 살짝 엿보면 위에서 언급한것들 뿐만 아니라 에러응답에 대한 핸들링까지도 구현되어있는것을 볼 수 있다. 그러니 큰 이슈가 없다면 `TestRestTemplate` 구현체를 사용하자.

```java
protected static class CustomHttpComponentsClientHttpRequestFactory extends HttpComponentsClientHttpRequestFactory {

    private final String cookieSpec;

    private final boolean enableRedirects;

    public CustomHttpComponentsClientHttpRequestFactory(HttpClientOption[] httpClientOptions) {
        Set<HttpClientOption> options = new HashSet<>(Arrays.asList(httpClientOptions));
        this.cookieSpec = (options.contains(HttpClientOption.ENABLE_COOKIES) ? CookieSpecs.STANDARD
                : CookieSpecs.IGNORE_COOKIES);
        this.enableRedirects = options.contains(HttpClientOption.ENABLE_REDIRECTS);
        if (options.contains(HttpClientOption.SSL)) {
            setHttpClient(createSslHttpClient());
        }
    }

    private HttpClient createSslHttpClient() {
        try {
            SSLConnectionSocketFactory socketFactory = new SSLConnectionSocketFactory(
                    new SSLContextBuilder().loadTrustMaterial(null, new TrustSelfSignedStrategy()).build());
            return HttpClients.custom().setSSLSocketFactory(socketFactory).build();
        }
        catch (Exception ex) {
            throw new IllegalStateException("Unable to create SSL HttpClient", ex);
        }
    }

    @Override
    protected HttpContext createHttpContext(HttpMethod httpMethod, URI uri) {
        HttpClientContext context = HttpClientContext.create();
        context.setRequestConfig(getRequestConfig());
        return context;
    }

    protected RequestConfig getRequestConfig() {
        Builder builder = RequestConfig.custom()
            .setCookieSpec(this.cookieSpec)
            .setAuthenticationEnabled(false)
            .setRedirectsEnabled(this.enableRedirects);
        return builder.build();
    }

}

private static class NoOpResponseErrorHandler extends DefaultResponseErrorHandler {

    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
    }

}
```

## `AuthenticationProvider` 중복 호출 이슈

비밀번호 실패시 실패 횟수 증가 로직을 `AuthenticationProvider`를 통해 구현하다가 중복으로 횟수가 증가하는 이슈를 발견하게 되었다.

첫번째 호출 시
![duplicate-auth-provider-call1](/assets/images/retrospective/duplicate-auth-provider-call1.png)

두번째 호출 시
![duplicate-auth-provider-call2](/assets/images/retrospective/duplicate-auth-provider-call2.png)

두번째 호출하는 이유를 살펴보니 provider의 parent로 호출되는 것을 볼 수 있었다.

![duplicate-auth-provider-call-cause](/assets/images/retrospective/duplicate-auth-provider-call-cause.png)

스프링 시큐리티의 필터 메커니즘 때문인데, 이를 방지하기 위해서 [OncePerRequestFilter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/OncePerRequestFilter.html){: target="\_blank" }라는 Filter 구현체가 있기는 하다. 하지만 `AuthenticationProvider`를 이용한 인증을 구현하기 위해서는 [AbstractAuthenticationProcessingFilter](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/AbstractAuthenticationProcessingFilter.html){: target="\_blank" }를 상속받아서 사용해야 했기 때문에 어쩔 수 없이 다른 방법을 찾아야 했다.

그래서 비밀번호 실패 횟수 증가를 `AuthenticationFailureHandler`에서 구현하기로 하였다.

```kotlin
@Component
class UsernamePasswordAuthenticationFailureHandler(
    private val objectMapper: ObjectMapper,
    private val userService: UserService,
) : AuthenticationFailureHandler {
    override fun onAuthenticationFailure(
        request: HttpServletRequest,
        response: HttpServletResponse,
        exception: AuthenticationException,
    ) {
        val message = if (exception is InvalidPasswordException) {
            val result = userService.addAuthenticationFailure(exception.username)
            "${exception.message} (${result.attempt}/${result.limit})"
        } else {
            exception.message
        }

        response.status = HttpStatus.UNAUTHORIZED.value()
        response.addHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        response.characterEncoding = Charsets.UTF_8.name()
        response.writer.write(createBody(message))
    }
}
```

## Spring Security로 로그인 API 구현 시 Swagger에 표시되지 않는 이슈

Spring Security를 이용해서 로그인 API를 구현했는데 Swagger에 표시되지 않는 이슈가 있다. 여기저기 찾아봐도 설정 사례는 보이지 않는다. 직접 문서에 강제로 추가하는 방법도 있을 것 같은데 이부분은 좀 더 찾아봐야겠다. 일각에서는 보안적인 이유로 Swagger에 보여주지 않는 것을 추천하기도 하더라.

## 중첩 AOP 테스트

아래와 같이 AOP를 중첩해서 적용하면 어떻게 동작할까?

```kotlin
@Aspect
@Component
class TestAop {
    @Around("execution(* com.tatum.backenduser.FooService.*(..))")
    fun handle(pjp: ProceedingJoinPoint) {
        println("aop 시작")
        pjp.proceed()
        println("aop 끝")
    }
}

@Configuration
@EnableAspectJAutoProxy
class Config

@Component
class FooService {
    fun test1() {
        println("test1 실행")
    }

    fun test2() {
        test1()
        println("test2 실행")
    }

    fun test3() {
        test2()
        println("test3 실행")
    }

    fun test4() {
        println("test4 실행")
    }
}

@RestController
class FooController(private val fooService: FooService) {
    @GetMapping("/test/1")
    fun test1() {
        fooService.test1()
    }

    @GetMapping("/test/2")
    fun test2() {
        fooService.test2()
    }

    @GetMapping("/test/3")
    fun test3() {
        fooService.test3()
    }

    @GetMapping("/test/4")
    fun test4() {
        fooService.test4()
    }
}
```

아래와 같은 결과를 볼 수 있다. 결국 중첩해서 적용하는 경우 가장 바깥에만 AOP가 적용되는 것을 볼 수 있다.

```
aop 시작
test1 실행
aop 끝


aop 시작
test1 실행
test2 실행
aop 끝


aop 시작
test1 실행
test2 실행
test3 실행
aop 끝


aop 시작
test4 실행
aop 끝
```

## RFC 관련 문서 찾아볼때 좋은 사이트

그렇게 자주 찾아볼일은 없지만 RFC 관련 문서를 찾아봐야할 일이 생겼을 때 사용하면 좋을만한 사이트를 적어둔다.

[https://datatracker.ietf.org/](https://datatracker.ietf.org/){: target="\_blank" }

## Swagger UI Cookie 이슈

Swagger UI에서 Custom 로그인을 구현한 경우 Cookie 로그인은 제대로 수행되지 않는 이슈가 있다. 우회해서 풀어볼 수 있는 방법들이 있어보이나 보안적인 이유로 막아둔 듯 하니 Curl을 이용해서 호출하는 형태로 풀어야할 듯 하다.

> Note for Swagger UI and Swagger Editor users: Cookie authentication is currently not supported for "try it out" requests due to browser security restrictions. See [this issue](https://github.com/swagger-api/swagger-js/issues/1163) for more information. [SwaggerHub](https://swagger.io/tools/swaggerhub/) does not have this limitation.

[https://swagger.io/docs/specification/authentication/cookie-authentication/](https://swagger.io/docs/specification/authentication/cookie-authentication/){: target="\_blank" }

## Output Captuer

테스트할 때 로그를 제대로 출력하는지를 테스트 하려고하면 주로 테스트용 LogAppender를 아래와 같이 추가해서 테스트하곤 하였었다.

```kotlin
class TestLogAppender : AppenderBase<ILoggingEvent>() {
    init {
        LoggerFactory.getLogger(ROOT_LOGGER_NAME)
            .let { (it as Logger).addAppender(this) }
        start()
    }

    private val events: MutableList<ILoggingEvent> = mutableListOf()

    override fun append(e: ILoggingEvent) {
        events.add(e)
    }

    val lastMessage: String get() = events.filterNot { it.level == Level.DEBUG }.last().message
}

class LoggingTest(
    private val testLogger: TestLogAppender,
) : FunctionalTestBase() {
    init {
        context("WebLogFilter") {
            test("GET API를 호출하면 올바른 로그를 출력한다.") {
                // Given
                val url = "/v1/login/user?username=foo@foo.com"

                testClient.get<CommonResponse<GetLoginUserResponse>>(url)

                // Expect
                eventually(5.seconds) {
                    testLogger.lastMessage shouldContain Regex(
                        """
                        Completed \[GET /v1/login/user] API \(username=foo@foo.com\) \[Response 200 Bytes: \d+]
                        """.trimIndent(),
                    )
                }
            }
        }
    }
}
```

이런방법 말고도 Spring 에서 제공하는 [OutputCapture](https://docs.spring.io/spring-boot/docs/2.0.0.M5/reference/html/boot-features-testing.html#boot-features-output-capture-test-utility){: target="\_blank" }라는 구현체도 있으니 이것을 이용해 보는것도 좋을것 같다.

```java
public class MyTest {

	@Rule
	public OutputCapture capture = new OutputCapture();

	@Test
	public void testName() throws Exception {
		System.out.println("Hello World!");
		assertThat(capture.toString(), containsString("World"));
	}

}
```

## Kotest에서 SpringExtention 사용

Kotest에서는 스프링을 사용하려면 `SpringExtention`을 설정하라고 가이드하고 있다.

```kotlin
override fun extensions() = listOf(SpringExtension)
```

반드시 필요한 설정이 아니라면 굳이 설정하는 것을 좋아하지 않아서 해당 설정을 넣어두지 않았는데 이 설정을 넣지 않으니 테스트 클래스에서 `@Autowired`를 사용하지 못하는 현상을 발견하게 되었다.

테스트할때 공통 설정을 위해 베이스 클래스를 만들어두고 각 테스트 클래스들이 해당 베이스 클래스를 상속받아서 사용하도록 하는 경우에 생성자 매개변수를 사용하면 불편함이 있어 `@Autowired`를 사용하는 경우가 종종 있는데 이를 위해서는 `SpringExtension`을 설정해주어야 한다.

```kotlin
@Import(FunctionalTestConfig::class)
@SpringBootTest(
    webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
)
abstract class FunctionalTestBase : FunSpec() {
    override fun extensions() = listOf(SpringExtension)

    @Autowired
    protected lateinit var testHelper: TestHelper

    @Autowired
    protected lateinit var testClient: RestTemplate
}
```
