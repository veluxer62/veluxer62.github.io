---
layout: single
title: "JWT를 이용한 Spring Security 인증 구현"
categories:
  - TUTORIALS
tags:
  - JWT
  - Spring
  - Spring Security
  - Kotlin
  - Authentication

toc: true
toc_sticky: true
excerpt: "사내에서 Spring Security를 이용한 JWT인증을 구현하면서 `UsernamePasswordAuthenticationFilter`와 같이 이미 구현되어있는 인증 구현체가 없어서 JWT를 이용한 인증 필터를 새롭게 구현하게 되었다. Session 기반 인증만 구현하면서 기존 구현체를 이용해 단순하게 사용해오던 Spring Security를 JWT로 인증로직을 수행하도록 하기위해 Custom filter를 구현하면서 Spring Security의 Authentication 아키텍쳐에 대해 좀더 공부해 볼 수 있는 시간을 가질 수 있게 되었고 이를 구현하면서 배웠던 부분들을 적어보고자 한다. 먼저 앞으로 소개될 기술들에 대한 간단한 개념부터 정리하고 튜토리얼을 이어가보자."
---

사내에서 Spring Security를 이용한 JWT인증을 구현하면서 `UsernamePasswordAuthenticationFilter`와 같이 이미 구현되어있는 인증 구현체가 없어서 JWT를 이용한 인증 필터를 새롭게 구현하게 되었다.
Session 기반 인증만 구현하면서 기존 구현체를 이용해 단순하게 사용해오던 Spring Security를 JWT로 인증로직을 수행하도록 하기위해 Custom filter를 구현하면서 Spring Security의 Authentication 아키텍쳐에 대해 좀더 공부해 볼 수 있는 시간을 가질 수 있게 되었고 이를 구현하면서 배웠던 부분들을 적어보고자 한다.

먼저 앞으로 소개될 기술들에 대한 간단한 개념부터 정리하고 튜토리얼을 이어가보자.

## JWT

JWT란 Json Web Token의 약자로 상호간에 JSON 객체와 같은 콤펙트하고 독립적인 정보를 안전하게 전달하기 위한 개방형 표준(RFC 7519)이다. JWT는 시크릿(HMAC 알고리즘) 또는 공개/비공개 키(RSA/ECDSA)를 이용하여 서명됩니다.

JWT는 `Header`, `Payload`, `Signature`로 구성되어 있으며 각 부분은 `Base64`로 인코딩 되어있고 `.`으로 구분되어 있다.

![jwt-structure](/assets/images/posts/spring-security-with-jwt/jwt-structure.png)

## Session 기반 인증 vs Token 기반 인증

JWT는 Token 기반 인증 방식 중 하나이다. Token 기반 인증은 Session 기반 인증과 비교해 보면 그 특징을 쉽게 파악할 수 있다.

### Session 기반 인증

상태를 가지지 않는 HTTP에서 상태를 유지하기 위한 수단으로 Session을 사용하여 사용자의 인증을 수행하는 방법을 말한다. 서버에서 클라이언트의 인증 정보를 저장소에 저장하여 이를 통해 세션정보를 관리하며 보통 사용자의 요청을 받으면 저장소에 저장된 정보를 이용하여 인증로직을 수행하는 방식으로 동작한다.

![session-auth-diagram](/assets/images/posts/spring-security-with-jwt/session-auth-diagram.png)

#### 장점

- 서버에서 클라이언트의 인증 상태를 유지하므로 인증 상태를 관리할 수 있다. 즉, 로그인 여부 확인 또는 필요하면 강제 로그아웃도 수행할 수 있다.
- 클라이언트의 매 요청 마다 인증 정보를 검증하므로 Token 기반 인증에 비해 비교적 위변조에 안전하다.

#### 단점

- 서버에서 상태를 관리하므로 클라이언트의 증가에 따라 서버의 부하도 함께 증가한다.
- Scale out 시 세션관리, 멀티 디바이스의 인증 처리 등 인증 관리의 어려움이 있다.

### Token 기반 인증

인증된 사용자임을 증명하기 위해 클라이언트에서 매 요청 시 마다 인증된 token을 서버로 전송하는 방법을 말한다. 서버는 클라이언트가 전송한 인증 token이 유효한지 여부만 판단하여 인증로직을 수행하기 때문에 별도의 저장소를 요하지 않는다는 특징이 있다.

![jwt-auth-diagram](/assets/images/posts/spring-security-with-jwt/jwt-auth-diagram.png)

#### 장점

- 서버에서 상태를 관리하지 않으므로 클라이언트 증가에 따른 서버 부하에 대한 부담이 적다.
- Scale out, 멀티 디바이스 등 서버 확장에 대한 대응이 용이하다.

#### 단점

- token에 정보를 많이 담을 경우 Request가 커질 수 있다.
- 서버에서 상태를 가지지 않으므로 클라이언트의 인증 상태 관리를 할 수 없다.
- token이 도난당하는 서버에서 대응할 수 있는 수단이 없다. (그래서 access token의 만료시간을 짧게 설정하고 refresh token을 두어 사용성을 높이는 전략을 채택하기도 한다.)

## Spring Security

Spring Security는 엔터프라이즈 애플리케이션에 인증, 인증 및 기타 보안 기능을 제공하는 Java/Java EE 프레임워크이다. 쉽게말해서 Spring 어플리케이션에서 인증되지 않은 사용자 또는 권한을 가지지 않은 사용자가 리소스 사용을 제한해 주는 일련의 기능들을 제공해주는 프레임워크이다.

### Authentication Architecture

Spring Security는 Spring MVC와는 별개의 프레임워크이지만 이번에 Spring MVC에서 적용했으니 [Servlet Application](https://docs.spring.io/spring-security/reference/servlet/index.html){: target="\_blank" }에 대한 아키텍쳐에 대해서만, 그리고 Servlet Application의 [Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/index.html#servlet-authentication){: target="\_blank" }, [Authorization](https://docs.spring.io/spring-security/reference/servlet/authorization/index.html#servlet-authorization){: target="\_blank" }, [Protection Against Exploits](https://docs.spring.io/spring-security/reference/servlet/exploits/index.html#servlet-exploits){: target="\_blank" } 섹션 중 `Authentication`에 대해서만 다루어 보고자 한다.

`Servlet`은 클라이언트의 요청을 여과해주는 기능으로 [Filter](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters){: target="\_blank" }를 사용한다. Spring Security도 이 Filter를 이용하며 여러개의 Filter들을 묶어서 사용자 인증로직을 수행하도록 구현되어 있다.

![spring-security-sevlet-app-arch](/assets/images/posts/spring-security-with-jwt/spring-security-sevlet-app-arch.png)

## Implementation

이제 Spring Security를 이용해서 JWT 인증 필터를 구현한 코드를 보자. 각 인터페이스들의 역할을 소개하고 그 구현체들을 하나하나 소개하여 수행하는 역할들을 살펴보고자 한다.

### Authentication

Spring Security에서는 [Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication){: target="\_blank" }는 2가지 용도로 사용된다.

- [AuthenticationManager](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationmanager){: target="\_blank" }에서 인증을 위한 입력 객체로 사용된다. 이때 `isAuthenticated()`는 false를 반환한다.
- 현재 인증된 사용자를 나타낸다. 인증된 사용자는 [SecurityContext](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontext){: target="\_blank" }에 저장된다.

![security-context-holder-archi](/assets/images/posts/spring-security-with-jwt/security-context-holder-archi.png)

#### UserPrincipal

`Authentication`의 `Principal`로 사용될 객체로 사용자 정보를 담고 있다. ([Username/Password Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html)에서는 [UserDetails](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details.html#servlet-authentication-userdetails){: target="\_blank" }가 사용된다.)

```kotlin
class UserPrincipal(
    val id: UUID,
    val name: String,
    val authorities: Collection<GrantedAuthority>,
) {
    constructor(user: User) : this(
        id = user.id,
        name = user.name,
        authorities = setOf(SimpleGrantedAuthority(user.role.name))
    )

    val claims: Map<String, Any> get() = mapOf(
        "user_id" to id,
        "user_name" to name,
        "authorities" to authorities.map { it.authority }
    )
}
```

#### JwtPreAuthenticationToken

[Authentication](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/Authentication.html){: target="\_blank" }의 구현체이며 `AuthenticationManager`에서 인증하기 위한 입력값으로 사용된다.

```kotlin
class JwtPreAuthenticationToken(
    private val token: String,
) : AbstractAuthenticationToken(null) {
    override fun getPrincipal() = token
    override fun getCredentials() = null
}
```

#### JwtUserAuthenticationToken

[Authentication](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/Authentication.html){: target="\_blank" }의 구현체이며 `AuthenticationManager`에서 인증완료 후 `SecurityContext`에 저장되는 인증된 사용자를 나타낸다.

```kotlin
class JwtUserAuthenticationToken(
    private val principal: UserPrincipal,
) : AbstractAuthenticationToken(principal.authorities) {
    init {
        super.setAuthenticated(true)
    }

    override fun getPrincipal() = principal
    override fun getCredentials() = null
}
```

### AuthenticationProvider

[AuthenticationManager](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationmanager){: target="\_blank" }는 Spring Security의 Filter에서 인증로직을 수행하는 API를 제공해준다.

Spring Security에서는 일반적으로 `AuthenticationManager`의 구현체로 [ProviderManager](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-providermanager){: target="\_blank" }를 사용한다. `ProviderManager`는 인증로직을 수행하는 여러개의 [AuthenticationProvider](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationprovider){: target="\_blank" }를 가지고 있으며 각 `AuthenticationProvider`가 인증로직을 수행한다. ([Username/Password Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html){: target="\_blank" }에서는 [DaoAuthenticationProvider](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/dao-authentication-provider.html){: target="\_blank" }가 주로 사용된다.)

![provider-manager-archi](/assets/images/posts/spring-security-with-jwt/provider-manager-archi.png)

#### JwtAuthenticationProvider

[AuthenticationProvider](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/AuthenticationProvider.html){: target="\_blank" }의 구현체이며 JWT token이 유효한지 검사한 후 인증된 사용자 정보를 반환합니다.

### AuthenticationEntryPoint

[AuthenticationEntryPoint](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationentrypoint){: target="\_blank" }는 클라이언트의 자격증명 요청에 대한 HTTP 응답을 정의한다. ([Username/Password Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html){: target="\_blank" }에서는 [LoginUrlAuthenticationEntryPoint](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/LoginUrlAuthenticationEntryPoint.html){: target="\_blank" }, [BasicAuthenticationEntryPoint](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/www/BasicAuthenticationEntryPoint.html){: target="\_blank" } 등이 사용된다.)

#### UnauthorizedAuthenticationEntryPoint

[AuthenticationEntryPoint](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/AuthenticationEntryPoint.html){: target="\_blank" }의 구현체이며 로그인 실패에 대한 응답으로 단순히 [Unauthorized 응답](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401){: target="\_blank" }을 반환하도록 한다.

```kotlin
@Component
class UnauthorizedAuthenticationEntryPoint : AuthenticationEntryPoint {
    override fun commence(
        request: HttpServletRequest,
        response: HttpServletResponse,
        authException: AuthenticationException
    ) {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED)
    }
}
```

### AbstractAuthenticationProcessingFilter

Spring Security는 공통된 인증로직을 제공하기 위해서 [Filter](https://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html){: target="\_blank" }의 구현체로 [AbstractAuthenticationProcessingFilter](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/AbstractAuthenticationProcessingFilter.html){: target="\_blank" }를 제공해준다.

해당 클래스의 인증로직은 아래와 같다.

![abstract-authentication-processing-filter-archi](/assets/images/posts/spring-security-with-jwt/abstract-authentication-processing-filter-archi.png)

1. AbstractAuthenticationProcessingFilter는 인증로직을 수행하고 인증이 완료되면 Authentication을 생성한다.

2. 인증 정보를 담은 Authentication은 AuthenticationManager에 전달되어 인증된다.

3. 인증에 실패하면 아래 기능들이 수행된다.

   - SecurityContextHolder가 지워진다.
   - RememberMeServices가 구현되어 있으면 loginFail이 호출된다.
   - AuthenticationFailureHandler가 호출된다.

4. 인증에 성공하면 아래 기능들이 수행된다.

   - SessionAuthenticationStrategy에 새 로그인이 통보된다.
   - SecurityContextHolder에 인증이 설정된다.
   - RememberMeServices가 구현되어 있으면 loginSuccess이 호출된다.
   - ApplicationEventPublisher가 InteractiveAuthenticationSuccessEvent를 호출된다.
   - AuthenticationSuccessHandler가 호출된다.

#### JwtAuthenticationFilter

`AbstractAuthenticationProcessingFilter`의 구현체로 JWT token을 이용한 필터기능을 제공하기 위해 아래와 같이 기능을 제정의하였다.

```kotlin
class JwtAuthenticationFilter(
    private val http: HttpSecurity,
    processUrl: String,
) : AbstractAuthenticationProcessingFilter(processUrl) {
    init {
        // (1)
        super.setAuthenticationSuccessHandler { _, response, _ ->
            response.status = HttpServletResponse.SC_OK
        }
    }

    // (2)
    override fun getAuthenticationManager(): AuthenticationManager =
        http.getSharedObject(AuthenticationManager::class.java)

    // (3)
    override fun attemptAuthentication(request: HttpServletRequest, response: HttpServletResponse): Authentication {
        val token = extractToken(request)
        val authentication = JwtPreAuthenticationToken(token)
        return this.authenticationManager.authenticate(authentication)
    }

    // (4)
    override fun successfulAuthentication(
        request: HttpServletRequest,
        response: HttpServletResponse,
        chain: FilterChain,
        authResult: Authentication
    ) {
        super.successfulAuthentication(request, response, chain, authResult)
        chain.doFilter(request, response)
    }
}
```

1. 인증 성공시 [OK 응답](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200){: target="\_blank" }을 반환하도록 한다. 해당 설정을 추가하지 않으면 `SavedRequestAwareAuthenticationSuccessHandler`의 `onAuthenticationSuccess`가 실행되므로 [Redirect 응답](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302)을 반환한다.

2. `HttpSecurity`가 가진 공유된 `AuthenticationManager`를 반환하도록 한다. 커스텀 필터는 기본적으로 `AuthenticationManager`를 자동으로 주입되지 않는다. 그래서 해당 설정을 해주지 않으면 `NullPointerException`이 발생한다.

3. 추상함수인 `attemptAuthentication`를 재정의한다. `request`에서 token을 추출하고 인증되지 않은 `Authentication`을 생성하여 `AuthenticationManager`에게 인증 요청을 보낸다.

4. 인증 성공 후 다음 filter를 실행하도록 `successfulAuthentication`를 재정의해주었다. `setContinueChainBeforeSuccessfulAuthentication(true)`로 설정하면 다음 filter를 실행하도록 할 수 있지만 `SecurityContext`가 담기기 전에 실행되므로 인증 실패응답이 반환된다. 그래서 `AbstractAuthenticationProcessingFilter.successfulAuthentication`를 먼저 실행시키고 난 후 `FilterChain.doFilter`가 실행되도록 한다.

### JwtProcessor

JWT token을 생성해주거나 파싱하는 프로세서이다. 토큰의 유효성 검증 주요 로직도 이 클래스가 가지고 있다.

```kotlin
@Component
class JwtProcessor(private val jwtProperties: JwtProperties) {
    // (1)
    fun generateToken(principal: UserPrincipal): String {
        val nowMillis = Clock.systemDefaultZone().millis()
        val issuedAt = Date(nowMillis)
        val expiration = Date(nowMillis + jwtProperties.expiration)

        return Jwts.builder()
            .setClaims(principal.claims)
            .setIssuedAt(issuedAt)
            .setExpiration(expiration)
            .signWith(SignatureAlgorithm.HS256, jwtProperties.secret)
            .compact()
    }

    // (2)
    fun getPrincipal(token: String): UserPrincipal {
        assert(token)
        return generatePrincipal(token)
    }
}
```

1. 사용자 정보를 기반으로 token을 생성한다.
2. token을 파싱해서 사용자 정보를 반환한다. 이때 유효 토큰을 함께 검증한다.

### Architecture

구현된 클래스들을 적용한 아키텍쳐를 보면 아래와 같다.

![implement-archi](/assets/images/posts/spring-security-with-jwt/implement-archi.png)

## Wrap Up

지금까지 JWT를 이용한 Spring Security 인증 로직 구현 방법을 알아보았다. Custom Filter를 구현하기 위해서 자료를 찾아보던 중 많은 블로그 글과 Github 코드에서 UserDetailsService를 활용하여 저장소에서 사용자정보를 조회한 후 인증 로직을 수행하는 것을 발견할 수 있었다. (심지어 [Wikipedia에서 소개한 Spring Boot JWT Auth](https://www.devglan.com/spring-security/spring-boot-jwt-auth){: target="\_blank" }에서도 동일하게 구현되어 있다.)

위에서 말한 [Token 기반 인증](#token-기반-인증)에서도 말했다시피 token 기반 인증은 클라이언트가 보내준 token이 유효한지만 체크하지 서버 내 상태를 체크하지 않는 것이 특징이다. 그래서 인증 로직을 조금 다르게 구현해 보았다.

위에서 소개한 코드는 구현코드의 일부만 적은 것이다. 자세한 코드를 보고자 한다면 [Github 저장소](https://github.com/veluxer62/spring-security-jwt-tutorial){: target="\_blank" }에서 확인하길 바란다.

> [https://jwt.io/introduction](https://jwt.io/introduction){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/JSON_Web_Token](https://en.wikipedia.org/wiki/JSON_Web_Token){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Spring_Security](https://en.wikipedia.org/wiki/Spring_Security){: target="\_blank" }
>
> [https://mangkyu.tistory.com/56](https://mangkyu.tistory.com/56){: target="\_blank" }
>
> [https://millo-l.github.io/Session-%EA%B8%B0%EB%B0%98-%EC%9D%B8%EC%A6%9D%EB%B0%A9%EC%8B%9D/](https://millo-l.github.io/Session-%EA%B8%B0%EB%B0%98-%EC%9D%B8%EC%A6%9D%EB%B0%A9%EC%8B%9D/){: target="\_blank" }
>
> [https://millo-l.github.io/JWT-%EA%B8%B0%EB%B0%98-%EC%9D%B8%EC%A6%9D%EB%B0%A9%EC%8B%9D/](https://millo-l.github.io/JWT-%EA%B8%B0%EB%B0%98-%EC%9D%B8%EC%A6%9D%EB%B0%A9%EC%8B%9D/){: target="\_blank" }
