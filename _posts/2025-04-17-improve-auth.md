---
layout: single
title: 단지 권한 기능을 추가해달라고 했을 뿐인데(feat. 인증 기능 개선)
categories:
  - EXPLANATION
toc: true
toc_sticky: true
toc_h_max: 2
---

이 글은 사내 블로그에 작성한 [단지 권한 기능을 추가해달라고 했을 뿐인데(feat. 인증 기능 개선)](https://spoqa.github.io/2025/04/18/improve-auth.html){: target="\_blank" } 내용을 그대로 가져오면서 나의 블로그의 언어톤에 맞게 변경한 글이다.

개발자라면 누구나 오랫동안 미뤄두었던 과제가 하나쯤 있을 것이다. 업무의 우선순위가 낮거나 긴급한 과제들에 밀려 지속적으로 백로그에 쌓여 있던 작업 말이다. 최근 우리팀에서 왜 오랜 시간 미뤄두었던 인증 방식 개선 작업을 진행하게 되었는지, 그 과정에서 얻은 여러 경험을 여러분께 공유하고자 한다.

# 배경

나의 블로그를 꾸준히 보신 분이라면, [서버 언어 전환 이야기](https://veluxer62.github.io/reference/all-new-server/){: target="\_blank" } 글에서 [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token){: target="\_blank" } 관련 문제를 언급하며 향후 개선할 예정이라고 소개했던 내용을 기억할 것이다. 

![jwt](/assets/images/posts/improve-auth/jwt.png)

약 3년이 흐른 지금, 드디어 우리가 인증 방식 개선을 진행하게 된 가장 큰 이유는 바로 키친보드 매장 앱에 권한관리 기능이 추가되었기 때문이다.

![request-work](/assets/images/posts/improve-auth/request-work.png)

키친보드 매장 앱은 식자재 주문부터 거래대금 결제까지 다양한 기능을 제공한다. 이 과정에서 사장님은 직원이 매장의 월 거래 내역 등 민감한 정보를 조회하지 못하도록 권한을 제어할 필요가 생겼는데, 기존 JWT 인증은 무 상태(stateless) 특성상 권한 변경 시 즉각적으로 클라이언트의 인증 상태를 관리할 수 없다는 한계가 있었다. 

그래서 우리는 권한 기능을 추가하기에 앞서 인증 방식을 먼저 개선하기로 하였다.

# 인증 방식 개선 방법

## Refresh Token 도입

앞서 이야기했듯이, JWT 기반의 인증 방식은 서버가 사용자의 상태를 저장하지 않는다. 덕분에 서버의 확장성이 높고 서버 부하를 줄일 수 있다는 장점이 있지만, 한번 발급된 토큰을 서버에서 직접 제어할 수 없다는 단점이 있다.

이러한 특성은 보안 문제로 연결될 수 있다. 만약 인증을 통해 발급받은 토큰이 탈취된다면, 서버가 이 토큰을 제어할 수 없으므로 악의적인 사용자는 손쉽게 탈취된 토큰을 이용하여 정상 사용자처럼 서비스를 이용할 수 있게 된다. 보통 이러한 보안 위험을 방지하기 위해 Access Token의 만료 시간을 짧게 설정하지만, 이 경우 사용자가 자주 로그인해야 하는 번거로움이 발생하게 된다.

이와 같은 문제를 해결할 수 있는 대표적인 방법의 하나가 바로 [Refresh Token](https://auth0.com/docs/secure/tokens/refresh-tokens){: target="\_blank" } 의 도입이다. Refresh Token은 Access Token과 달리 서버가 상태를 관리하는 토큰으로, Access Token을 갱신하는 데 사용된다. 앞서 말한 대로, Access Token의 탈취 위험을 낮추기 위해 Access Token의 만료 시간을 짧게 설정하는 것이 좋다. 이때 Refresh Token을 활용하면 사용자가 Access Token의 만료 시점마다 다시 로그인하지 않아도 편리하게 새로운 Access Token을 발급받을 수 있다.

다음 그림에서 Access Token과 Refresh Token의 인증 과정을 자세히 확인할 수 있다.

![auth-flow-with-refresh-token](/assets/images/posts/improve-auth/auth-flow-with-refresh-token.png)

1. 사용자가 로그인을 요청하면 서버는 Access Token과 Refresh Token을 발급한다.
2. 사용자는 발급받은 유효한 Access Token을 이용해 API를 호출하고, 서버는 요청된 데이터를 정상적으로 응답한다.
3. 사용자가 만료된 Access Token을 가지고 API 요청을 하면 서버는 401 인증 에러를 반환한다. 이때 클라이언트는 Refresh Token을 사용하여 새로운 Access Token을 발급받고, 갱신된 Access Token으로 API를 재요청하여 정상적으로 데이터를 받을 수 있다.
4. 하지만 만약 사용자의 Refresh Token까지 만료된 상태라면, 서버는 최종적으로 401 인증 에러를 반환하여 사용자의 다시 로그인을 요구한다.

위 과정을 통해 일반적으로 Access Token의 만료 시간을 짧게 설정하여 Access Token의 탈취 위험을 최소화하고, Refresh Token을 통해 사용자 편의성 또한 유지할 수 있다.

아래 그림을 통해 Access Token의 탈취로 인한 공격 시나리오로 Access Token의 만료시간이 짧으면 짧을수록 보안 위험도가 감소하게 된다.

![malicous-case1](/assets/images/posts/improve-auth/malicous-case1.png)

이처럼 Refresh Token을 적절히 도입하고 관리하면 토큰 탈취로 인한 보안 위험을 효과적으로 감소시킬 뿐만 아니라 사용자가 매번 로그인해야 하는 문제도 해결할 수 있으므로 사용성도 함께 챙길 수 있게 된다.

## Refresh Token Rotation

한편, Access Token에 대한 탈취 위험은 Refresh Token도 동일한 것 아닌가? 라는 질문을 할 수 있을 것 같다. 맞다. Refresh Token이 탈취당하면 Access Token을 갱신할 수 있고 갱신된 Access Token을 통해 악의적 사용자는 손쉽게 탈취한 사용자인 척 서비스를 이용할 수 있게 된다.

이러한 문제를 해결하기 위해 우리는 [Refresh Token Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation){: target="\_blank" } 을 도입하기로 한다. Refresh Token Rotation은 아래와 같이 Refresh Token을 이용해 Access Token을 갱신할 때 Refresh Token도 함께 갱신하여 Refresh Token 탈취 시 발생할 수 있는 위험을 회피한다.

![malicous-case2](/assets/images/posts/improve-auth/malicous-case2.png)

# 이슈

## 클라이언트의 네트워크 이슈

앞서 우리는 Refresh Token Rotation을 이용하여 Refresh Token 탈취에 대한 위험성을 회피하고자 하였다. 이렇게 하면 Refresh Token을 이용하여 Access Token을 갱신 요청할 때 요청한 Refresh Token도 새롭게 발급되어 더 이상 Refresh Token을 사용할 수 없게 된다. 보안 수준은 강화되었지만, 클라이언트 개발자분들이 한가지 우려 점을 제기해 주셨다.

모바일 기기 특성상 지하실이나 엘리베이터안과같이 네트워크가 원활하지 않은 곳에서 사용할 가능성이 존재한다. 이때 아래 그림과 같이 첫 번째 요청한 Refresh Token을 재요청하는 경우가 발생할 수 있다. 하지만 Refresh Token을 매번 갱신하기 때문에 동일한 Refresh Token으로 여러번 Access Token을 갱신요청하게 된다면 두번째 요청부터는  인증 에러가 발생하게 된다.

![network-issue-case](/assets/images/posts/improve-auth/network-issue-case.png)

그래서 우리는 [Token Family 방식](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation#automatic-reuse-detection){: target="\_blank" } 을 사용하여 요청 시마다 기존 Refresh Token을 지우지 않고 과거 토큰을 저장해 두었다가 첫 번째 요청으로 새롭게 발급된 토큰 또는 모종의 이유로 인해 갱신하지 못한 기존 토큰으로 토큰 갱신 요청을 할 수 있도록 구현하여 Refresh Token을 재사용할 수 있도록 하였다.

![token-family-flow](/assets/images/posts/improve-auth/token-family-flow.png)

이로써 클라이언트는 네트워크 이슈가 발생해도 Refresh Token을 갱신할 수 있게 되었다.

![resolved-network-issue-flow](/assets/images/posts/improve-auth/resolved-network-issue-flow.png)

## 인증 토큰의 하위 호환

한편, 우리는 JWT를 다루는 라이브러리로 [JJWT](https://github.com/spoqa/jjwt){: target="\_blank" } 를 사용하고 있다. 앞서 JWT는 상태를 가지지 않기 때문에 사용자가 사용하는 Access Token을 서버에서 제어할 수 없다고 말했었다. 그래서 Access Token의 만료 시간을 두어 새롭게 Access Token을 발급받도록 하여 우회적으로 제어할 수 있다. Access Token을 만료시키는 또 다른 방법은, 해당 토큰을 생성할 때 사용된 암호키를 변경하는 것이다. 우리는 그래서 클라이언트에서 사용하는 Access Token을 만료시키고 새롭게 변경된 권한을 사용하는 Access Token으로 사용하도록 하기 위해 암호키를 바꾸기로 하였다. 다만 여기서 발생하는 문제가 바로 앱의 업데이트 타이밍이었다.

개발자라면 다들 잘 알겠지만, 서버와 앱은 동일한 시점에 개발이 완료되더라도 배포되는 시점이 다를 수 있다. 서버는 배포하는 즉시 배포가 되지만 앱은 심사 과정이 필요하고 배포가 되더라도 앱스토어에 배포된 버전이 전파되기까지 1일 이상 소요될 수 있다. 그러다 보니 Access Token을 변경하기 위해 키를 변경하게 되면 서버가 배포된 이후부터 앱이 업데이트되기 전까지 사용자가 서비스를 이용할 수 없다는 문제가 생길 수 있다. 그래서 우리는 과거 버전의 앱에서도 새롭게 배포된 서버의 인증을 문제없이 사용할 수 있도록 방법을 모색해야 했다.

### JJWT 버전 변경

한편, 우리는 비밀키를 바꾸는 김에, 과거에 사용하던 서명 알고리즘(HS256)에 비해 보안성이 강화된 서명 알고리즘(PS256)을 변경하기로 하였다. 그러다 보니 JJWT라이브러리 버전을 업그레이드해야 했다. Gradle에 아래처럼 동일한 라이브러리를 서로 다른 버전으로 사용하는 경우 패키지 충돌이 발생하여 신규 버전에서 제공하는 함수를 사용할 수 없게 된다.

```kotlin
// 구버전
implementation("io.jsonwebtoken:jjwt:0.9.1")

// 신규버전
implementation("io.jsonwebtoken:jjwt-api:0.12.6")
runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.6")
runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.6")
```

![jjwt-version-conflict](/assets/images/posts/improve-auth/jjwt-version-conflict.png)

이러한 문제를 해결하기 위해 우리는 [Jitpack](https://jitpack.io/){: target="\_blank" } 을 사용하기로 하였다. JitPack은 GitHub에 호스팅된 라이브러리를 쉽게 빌드하고 배포할 수 있게 해주는 Maven/Gradle 용 리포지터리 서비스이다. GitHub 저장소를 바탕으로 라이브러리를 빌드하므로, 별도의 중앙 저장소(예: Maven Central) 등록 과정을 거치지 않아도 된다는 장점이 있다. 그리고 오픈소스 저장소에, 한에 무료로 사용할 수 있다는 점도 장점이다.

우리는 JJWT 라이브러리를 fork하여 [Spoqa용 JJWT Github 저장소](https://github.com/spoqa/jjwt){: target="\_blank" } 를 생성하였다. 그런 다음 충돌 패키지 충돌이 발생하지 않도록 패키지명을 변경해 주었다.

![changed-package-commit](/assets/images/posts/improve-auth/changed-package-commit.png)

그런 다음 Release를 생성해 주면, 아래와 같이 Jitpack에서 조회할 수 있게 된다.

![jitpack](/assets/images/posts/improve-auth/jitpack.png)

마지막으로 아래와 같이 Gradle에 의존성을 추가해주면, 패키지명이 변경된 JJWT라이브러리를 사용할 수 있게 된다.

```kotlin
implementation("com.github.spoqa:jjwt:1.0.2")
implementation("javax.xml.bind:jaxb-api:2.3.1")

implementation("io.jsonwebtoken:jjwt-api:0.12.6")
runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.6")
runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.6")
```

### Composite 패턴 vs TokenManager

우리는 Jitpack으로 생성한 과거버전의 JJWT를 의존하는 구현체를 아래와 같이 `LegacyJwtProcessor`로 변경하고 신규 버전을 사용하는 `JwtProcessor`를 새롭게 생성하였다. 그런 다음 아래와 같이 인증 로직에 과거 버전의 Access Token과 신규 버전의 Access Token을 모두 수용할 수 있도록 구현하였다.

```kotlin
@Component
class JwtAuthenticationProvider(
    private val legacyJwtProcessor: LegacyJwtProcessor,
    private val jwtProcessor: JwtProcessor,
) : AuthenticationProvider {
    override fun authenticate(authentication: Authentication): Authentication? {
        if (!supports(authentication::class.java)) return null

        val principal =
            try {
                jwtProcessor.getPrincipal(authentication.principal.toString())
            } catch (e: AuthenticationException) {
                legacyJwtProcessor.getPrincipal(authentication.principal.toString())
            }

        return JwtUserAuthenticationToken(principal)
    }

    override fun supports(authentication: Class<*>): Boolean {
        return authentication == JwtPreAuthenticationToken::class.java
    }
}
```

이렇게 구현하면 앱이 배포되기 전에 서버가 먼저 배포되어도 기존 버전을 사용하는 사용자가 정상적으로 로그인을 유지할 수 있게 된다.

한편, 인증 로직을 구현하는 곳 말고도 LegacyJwtProcessor를 사용하는 곳이 다수 존재하였다. 그러다 보니 새롭게 만들어진 JwtProcessor로 전환하는 것을 누락할 가능성이 존재하였다. 다행히 기능 테스트가 있어 놓친 구현을 바로잡을 순 있었지만, 코드의 응집성 측면에서는 좋은 코드는 아니라 생각하였다.

그래서 Composite 패턴을 사용해서 아래와 같이 구현해 볼지 생각을 하였다.

```kotlin
interface JwtProcessor {
    fun generateToken(principal: UserPrincipal): String
    fun getPrincipal(token: String): UserPrincipal
}

class CompositeJwtProcessor(
    private val newJwtProcessor: JwtProcessor,
    private val legacyJwtProcessor: JwtProcessor,
): JwtProcessor {
    fun generateToken(principal: UserPrincipal): String {
        // 생략...
    }
    fun getPrincipal(token: String): UserPrincipal {
        return try {
            jwtProcessor.getPrincipal(token)
        } catch (e: AuthenticationException) {
            legacyJwtProcessor.getPrincipal(token)
        }
    }
}

class NewJwtProcessor: JwtProcessor {
    fun generateToken(principal: UserPrincipal): String {
        // 생략...
    }
    fun getPrincipal(token: String): UserPrincipal {
        // 생략...
    }
}

class legacyJwtProcessor: JwtProcessor {
    fun generateToken(principal: UserPrincipal): String {
        // 생략...
    }
    fun getPrincipal(token: String): UserPrincipal {
        // 생략...
    }
}
```

`CompositeJwtProcessor`를 이용하면 아래와 같이 `JwtAuthenticationProvider`는 더 이상 `legacyJwtProcessor`를 알지 않아도 되고 추후 `legacyJwtProcessor`가 제거되어도 영향범위는 `CompositeJwtProcessor`로 한정되기 때문에 응집도 높은 코드를 유지할 수 있게 된다.

```kotlin
@Component
class JwtAuthenticationProvider(
    private val compositeJwtProcessor: JwtProcessor,
) : AuthenticationProvider {
    override fun authenticate(authentication: Authentication): Authentication? {
        if (!supports(authentication::class.java)) return null

        val principal = compositeJwtProcessor.getPrincipal(authentication.principal.toString())

        return JwtUserAuthenticationToken(principal)
    }
    
    // 생략...
}
```

다른 방법으로는 TokenManager라는 상위 수준의 클래스를 만들어 응집도를 높이는 방법도 생각해 보았다.

```kotlin
@Service
class TokenManager(
    private val jwtProcessor: JwtProcessor,
    private val legacyJwtProcessor: LegacyJwtProcessor,
    private val refreshTokenService: RefreshTokenService,
) {
    fun getPrincipal(token: String): UserPrincipal {
        return try {
            jwtProcessor.getPrincipal(token)
        } catch (e: AuthenticationException) {
            legacyJwtProcessor.getPrincipal(token)
        }
    }

    fun generateAccessToken(userPrincipal: UserPrincipal): String {
        // 생략...
    }

    fun generateRefreshToken(entity: RefreshToken): String {
        // 생략...
    }

    fun replaceRefreshToken(principal: RefreshTokenUserPrincipal): String {
        // 생략...
    }
}
```

이렇게 하면 Token을 Composite 패턴을 사용한 것과 같이 하위호환을 지키는 코드와 함께 토큰과 관련된 다른 기능들도 해당 클래스로 모을 수 있어 응집도를 상당히 높일 수 있게 된다.

어떤 방식이 더 나은 방식이라고 말하긴 어려울 것 같다. 다만, 우리는 LegacyJwtProcessor는 앱 배포 이후에 제거될 클래스이므로 불필요하게 Composite 패턴을 사용하기보다 TokenManager를 생성하여 코드 응집도를 높이는 방법으로 결정하게 되었다.

### Spring Security - PreAuthorize

우리는 인증과 인가를 위해 Spring Security를 사용하고 있다. JWT를 통해 인증된 사용자는 UserPrincipal이라는 인증된 사용자로 변환되고 UserPrinciapl이 가진 authorities를 통해 권한 처리를 하고 있다.

```kotlin
@Component
class AccessTokenAuthenticationProvider(
    private val tokenManager: TokenManager,
) : AuthenticationProvider {
    override fun authenticate(authentication: Authentication): Authentication? {
        if (!supports(authentication::class.java)) return null

        val principal = tokenManager.getPrincipal(authentication.principal.toString())
        return AccessTokenAuthenticationToken(principal)
    }

    override fun supports(authentication: Class<*>): Boolean {
        return authentication == AccessTokenPreAuthenticationToken::class.java
    }
}

class AccessTokenAuthenticationToken(
    private val principal: UserPrincipal,
) : AbstractAuthenticationToken(principal.authorities) {
    init {
        super.setAuthenticated(true)
    }

    override fun getPrincipal() = principal

    override fun getCredentials() = null
}
```

사용자는 아래와 같이 `@Secured`를 통해 권한을 검증받고 API를 호출할 수 있다.

```kotlin
@DgsMutation
@Secured(STORE_ADMIN, STORE_MANAGER, VENDOR)
fun createOrderSheet(
    @InputArgument input: CreateOrderSheetInput,
): CreateOrderSheet {
    // 생략...
}
```

`@Secured`는 단순한 권한 Role 기반 접근을 제어하기에 적절하다. 이전까지 키친보드는 관리자, 매장 사용자, 유통사 사용자로 명확하게 Role이 나뉘어져 있었기 때문에 `@Secured`는 요구사항을 충분히 충족하면서 단순하게 구현할 방법이었다.

하지만 새로운 요구사항이 추가되면서 매장 사용자는 매장 관리자, 매장 직원으로 권한이 분리되게 되었는데, 이에 따라 매장 사용자 모두 접근을 할 수 있는 API에는 아래와 같이 `@Secured(STORE_ADMIN, STORE_MANAGER)` 표현해야 하는 불편함이 있게 되었다. 거기다 만약 `STORE_INTERN`이 추가된다면 매장 사용자 권한을 가져야 하는 API를 모두 찾아서 바꿔줘야 하니 상당히 번거로운 작업이 될 것이고 자칫 권한 변경을 누락할 수 있는 위험성 또한 내포하고 있었다.

이와 같은 문제를 해소하기 위해 Spring Security에서는 `@PreAuthorize`를 이용하여 유연하게 권한을 체크하는 기능을 제공한다. 그래서 우리는 아래와 같이 매장 사용자 여부를 확인하는 서비스 함수를 만들어 [SpEL](https://docs.spring.io/spring-framework/reference/core/expressions.html){: target="\_blank" } 을 이용해 권한을 체크하도록 함으로써 권한을 일일이 나열하지 않고 새로운 권한이 생기더라도 유연하게 대처할 수 있도록 하였다.

```kotlin
@PostMapping("/replace-store")
@PreAuthorize("@authorizationExpressionHelper.isManager()")
fun replaceStore(@RequestBody request: ReplaceStoreRequest): ReplaceStoreResponse {
    // 생략...
}

@Service
class AuthorizationExpressionHelper {
    fun isManager(): Boolean {
        return PrincipalProvider.userPrincipal.isManager
    }
}
```

# 마무리

지금까지 우리가 권한 기능을 추가하기 위해 인증 로직을 어떻게 개선하였고 개선하면서 겪었던 이슈들을 공유해 보았다. 단순히 권한을 추가해 달라는 요구사항에서 시작되었지만, 그동안 우리가 가지고 있던 기술 부채도 해결함과 동시에 기술적인 여러 고민을 할 수 있어서 개인적으로 배운 게 많은 프로젝트였다.

모쪼록 인증 기능 구현에 관심이 있으시거나 예정인 분들께 도움이 되었으면 한다.
