---
layout: single
title: "Active MQ Tutorial By Scenario"
categories:
  - TUTORIALS
tags:
  - active mq
  - queue model
  - topic model
  - virtual topic
  - dead letter queue
  - delay
  - scheduled
  - spring boot
  - jms

toc: true
toc_sticky: true
excerpt: 이번 글에서는 Active MQ를 시나리별로 사용하는 방법에 대해서 작성해 보고자 한다.
---

## Intro

이번 글에서는 Active MQ를 시나리별로 사용하는 방법에 대해서 작성해 보고자 한다.

Message Queue를 소개하고 사용하는 방법에 대해서 소개할 때 어떻게 알려주면 좋을 지 고민을 많이 해봤다. 단순히 Spring Boot에서 어떻게 사용하면 되는지 튜토리얼을 적는 것은 Active MQ를 왜 사용해야 하는 지에 대한 인사이트를 주지 못할 것 같다는 생각이 들었다. 아마 이 글을 보는 사람들이라면 아마 Message Queue에 대해서 익숙하지 않은 사람이거나 Active MQ에 대해서 익숙하지 않은 사람일 가능성이 높다고 생각된다. 단순히 사용하는 방법에 대한 내용을 적는 것 보다 특정 시나리오를 주어주고 그 시나리오에 맞는 사용법을 설명하면 좋을 것 같다고 생각하였고, 그래서 시나리오 별로 어떤 설정을 통해 Active MQ를 활용 할 수 있을 지 설명하면서 Spring Boot 코드를 가이드하면서 튜토리얼을 진행해 보자 한다.

Active MQ가 어떤 소프트웨어 인지 궁금하다면 이전에 [작성한 글](https://veluxer62.github.io/reference/message-queue/#amazon-mq){: target="\_blank" }을 통해서 확인해보자.

## How to use Active MQ

### Installation
Active MQ는 Home Brew를 통해 설치할 수 있다. (Windows의 경우 공식 홈페이지의 [다운로드링크](http://activemq.apache.org/components/classic/download/){: target="\_blank" }를 통해 설치 가능하다.)

```bash
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null
```

```bash
$ brew install activemq
```

### Run

설치가 완료되면 `/usr/local/Cellar/apache-activemq/x.x.x/` 경로에 설치가 완료된다.

실행방법은 아래와 같다.

```bash
$ activemq start
```

만약 현재 실행한 후에 컴퓨터를 재부팅해도 계속해서 사용하고 싶다면 아래 명령어를 입력하자.

```bash
$ brew services start activemq
```

정상적으로 실행되었다면 기폰 포트인 `61616`으로 실행 되었으니 확인하면 된다.

```bash
lsof -i -n -P | grep 61616
```

### Stop

Active MQ를 중지하는 방법은 아래 명령어를 입력하면 된다.

```bash
$ activemq stop
```

### Monitoring

Active MQ는 기본적으로 모니터링을 위한 Web Concole을 제공해준다. 기본 경로인 [`http://localhost:8161/admin`](http://localhost:8161/admin){: target="\_blank" }로 접속해보자.

접속하면 아래와 같이 로그인 화면이 표시되는데 Username을 `admin`, Password를 `admin`으로 입력하면된다.

[![admin login](/assets/images/posts/active-mq-tutorial-by-scenario/admin-login.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-login.png)

로그인 후 아래 화면으로 접속이 된다면 정상적으로 설치가 완료된 것이다.

[![admin home](/assets/images/posts/active-mq-tutorial-by-scenario/admin-home.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-home.png)

## Generate Spring Boot Project

이 글의 튜토리얼을 진행하려면 서버를 여러대 실행해야 한다. (서버 한대로 Producer와 Consumer를 구현할 순 있지만 좀더 현실적인 사례를 보기 위해서 서버를 여러대 실행해서 메시지가 전달되는 것을 보면 좋을 것 같아서 이다.) 그래서 공통적으로 프로젝트를 생성하는 방법에 대해서 가이드를 먼저 진행하려고 한다. 아래에서 시나리오에서는 여러 프로젝트를 새롭게 생성할 것이고 프로젝트 생성 내용에 대해서는 이 챕터를 참고하도록 가이드 하려고 한다.

### 프로젝트생성

[https://start.spring.io/](https://start.spring.io/){: target="\_blank" }를 접속한 후 `Spring for Apache ActiveMQ 5` 의존성을 추가해 준다.
이 글에서는 `Gradle Project` + `Kotlin`로 프로젝트를 설정하였지만 설정은 개인의 취향에 맞게 설정하면 된다.
설정이 완료 되면 `GENERATE` 버튼을 눌러 파일을 다운로드하자.

[![spring boot project](/assets/images/posts/active-mq-tutorial-by-scenario/init-spring-boot-project.png)](/assets/images/posts/active-mq-tutorial-by-scenario/init-spring-boot-project.png)

다운로드된 파일을 압축 파일이므로 원하는 디렉토리에 압축을 풀어 사용하자.

로컬에 설치한 Active MQ를 사용하기 위한 프로퍼티 설정을 해주자.

```yml
# application.yml
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

객체 타입 메시지를 문자형태로 전송하기 위한 컨버터를 추가하자. 

```kotlin
// ActiveMQConfiguration.kt

@Configuration
class ActiveMQConfiguration {

    @Bean
    fun messageConverter(): MessageConverter {
        val converter = MappingJackson2MessageConverter()
        converter.setTargetType(MessageType.TEXT)
        converter.setTypeIdPropertyName("_type")
        return converter
    }

}
```

메시지 객체도 생성해주자. 메시지 형식은 중요하지 않으니 단순하게 생성했다. (Kotlin Data Class를 사용하는 경우 Default 값을 넣어주지 않으면 메시지 컨버터에서 오류가 발생하니 사용시 유의하자.)

```kotlin
// OrderMessage.kt

data class OrderMessage(
        val payload: String = ""
) : Serializable
```

## Scenario

### 단일 사용자에게 순서를 보장하는 메시지를 전달해야 하는 경우

우리는 하나의 이벤트가 발생하면 발생한 이벤트 메시지를 한명의 수신자에게만 전달하여 처리하도록 하는 기능 개발을 많이 하고 있다. 

간단한 주문관리 어플리케이션을 예를 들어 보겠다. 어플리케이션 서버는 단일 서버이다. 이때 사용자가 어떤 물건에 대한 주문 요청을 보내고 바로 주문 취소요청을 보냈다고 가정을 해보자. 서버는 한대 뿐이므로 우리는 메시지를 한명의 수신자에게만 전송하면 된다. 그리고 주문 요청과 주문 취소 요청은 순서를 가지는 이벤트 이므로 순서를 꼭 지켜서 전송되어야 한다.

이렇듯 단일 사용자에게 순서를 보장하는 메시지를 전달해야 하는 경우에는 Queue Model을 사용하면 된다. Queue Model의 구조는 아래와 같다.

[![queue model](/assets/images/posts/active-mq-tutorial-by-scenario/queue-model.png)](/assets/images/posts/active-mq-tutorial-by-scenario/queue-model.png)

**Code Example**

해당 시나리오에 대한 튜토리얼을 진행하기 위해서는 Producer 서버와 Consumer 서버 각각 한개씩 실행하고자 한다.

**Producer**

메시지 전송을 위한 Producer 서버를 추가해보자.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

Spring Boot에서는 기본적으로 Queue Model로 동작하게 설정되어 있기 때문에 application.yml에서 변경사항은 없다. (포트 정도만 설정하자.)

```yml
# application.yml

server:
  port: 8080
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 전송을 위한 빈을 추가한다. 해당 빈은 `JmsTemplate`을 생성자로 주입 받아서 사용한다.

```kotlin
// MessageSender.kt

@Component
class MessageSender(private val jmsTemplate: JmsTemplate) {
    private val logger = LoggerFactory.getLogger(this::class.java)

    fun send(message: OrderMessage) {
        logger.info("Producer Message -> [$message]")
        jmsTemplate.convertAndSend("ORDER", message)
    }
}
```

어플리케이션이 실행될 때 메시지를 발송하도록 어플리케이션 실행 코드에 추가 코드를 작성하자.

```kotlin
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
    val app = runApplication<DemoApplication>(*args)
    val sender = app.getBean(MessageSender::class.java)
    
    (1..10).forEach { 
        sender.send(OrderMessage(payload = "test-message-${it}"))
    }
}
```

**Consumer**

메시지 수신을 위한 Consumer 서버를 추가해 보자.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

Consumer도 Queue Model로 동작하게 설정하기 위해 별도 설정 없이 포트만 Producer 서버와 다르게 설정하자.

```yml
server:
  port: 8081
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 수신을 위한 빈을 추가한다. Spring Boot에서 Active MQ의 메시지를 수신하기 위한 여러가지 방법이 있는데 이 글에서는 어노테이션을 이용한 방법으로 진행하였다. 다른 방법에 대해 궁금하다면 [공식 문서 링크](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#jms-receiving){: target="\_blank" }를 참고하자.

```kotlin
@Component
class MessageListener {
    private val logger = LoggerFactory.getLogger(this::class.java)

    @JmsListener(destination = "ORDER")
    fun listen(message: OrderMessage) {
        logger.info("Consumer Listen - > $message")
    }
}
```

**Run**

먼저 Consumer 서버 부터 실행 시키자. 사실 Producer에서 메시지를 먼저 전송한 후에 Consumer 서버를 실행해도 메시지를 수신할 수 있다. Active MQ에서 기본적으로 Consumer가 메시지를 수신할 수 있는 상태가 되면 메시지를 전송해 준다. 이 부분은 우리가 MQ를 사용하면 누릴 수 있는 여러 장점 중에 하나 이다.

다음으로 Producer 서버를 실행 시키자. Application 코드에 메시지를 전송하는 코드를 넣어 두었기 때문에 서버가 기동되면 자동으로 10개의 메시지를 발송한다.

[![run queue-model](/assets/images/posts/active-mq-tutorial-by-scenario/run-queue-model.gif)](/assets/images/posts/active-mq-tutorial-by-scenario/run-queue-model.gif)

위 영상에서 볼 수 있듯이 Producer가 메시지를 발송하면 Consumer가 순차적으로 메시지를 수신함을 확인할 수 있다.

그럼 만약에 하나가 아닌 여러 Consumer를 두게 되면 어떻게 동작하게 될까?

[![run queue-model-multi](/assets/images/posts/active-mq-tutorial-by-scenario/run-queue-model-multi.gif)](/assets/images/posts/active-mq-tutorial-by-scenario/run-queue-model-multi.gif)

영상에서 보는것 처럼 라운드 로빈 방식으로 메시지는 분산되어 전송된다. 여기에서 우리는 Topic 모델을 사용하지 않을 것이라는 가정하에 스케일 아웃을 통한 Consumer 서버의 가용성을 확보할 수 있을 것이라는 결론을 내릴 수 있다.

**Monitoring**

전송된 Queue Message들은 Active MQ의 Web Console의 [Queues](http://localhost:8161/admin/queues.jsp){: target="\_blank" } 메뉴에서 확인이 가능하다.

[![admin queue history](/assets/images/posts/active-mq-tutorial-by-scenario/admin-queue-history.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-queue-history.png)

**GIT**

Queue Model에 대한 코드는 아래 Github 주소를 참고하자.

- Producer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/queue-model/activemq-sender](https://github.com/veluxer62/activemq-tutorial/tree/queue-model/activemq-sender){: target="\_blank" }

- Consumer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/queue-model/activemq-receiver](https://github.com/veluxer62/activemq-tutorial/tree/queue-model/activemq-receiver){: target="\_blank" }

<br/>

### 하나의 메시지를 다수의 사용자에게 전달해야 하는 경우

이전에 보았던 시나리오를 다시 얘기하면 하나의 주문이 발생하였을 때 하나의 서버로 메시지를 전달하는 시나리오에 대해서 알아보았었다. 

만약 사용자가 요청한 주문을 또 다른 서버가 수신하여야 하는 상황이 되는 경우 어떻게 하면 좋을까? 바로 발행/구독 모델을 사용하면 좋다. 서비스를 개발하는 경우 이렇게 하나의 메시지에 대해 여러 수신자가 메시지를 수신해야 하는 경우가 많이 발생한다. 예를 들어 주문이 발생하면 서버1은 주문을 처리하고 사용자에게 처리한 주문 정보를 제공하는 역할을 수행하고 서버2는 주문에 대한 이력을 수집하여 정산을 위한 데이터를 생성하는 역할을 수행하고, 서버3은 제3의 사용자에게 주문정보를 전달해야 하는 경우 등을 들 수 있을 것 같다.

이렇듯 단일 메시지를 여러 수신자들이 메시지를 수신받도록 해야 하는 경우에는 Topic Model을 사용하면 된다. Topic Model의 구조는 아래와 같다.

[![publisher subscriber model](/assets/images/posts/active-mq-tutorial-by-scenario/pub-sub-model.png)](/assets/images/posts/active-mq-tutorial-by-scenario/pub-sub-model.png)

**Code Example**

해당 시나리오에 대한 튜토리얼을 진행하기 위해서는 Producer 서버 하나와 Consumer 서버 두개를 실행하고자 한다. (Consumer를 추가로 해도 무방하다.)

**Producer**

메시지 전송을 위한 Producer 서버를 추가해보자.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

Topic Model을 사용하기 위해서는 `pub-sub-domain`속성을 `true`로 변경해 주어야 한다.

```yml
# application.yml

server:
  port: 8080
spring:
  jms:
      pub-sub-domain: true
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 전송을 위한 빈을 추가한다. 해당 빈은 `JmsTemplate`을 생성자로 주입 받아서 사용한다.

```kotlin
// MessageSender.kt

@Component
class MessageSender(private val jmsTemplate: JmsTemplate) {
    private val logger = LoggerFactory.getLogger(this::class.java)

    fun send(message: OrderMessage) {
        logger.info("Producer Message -> [$message]")
        jmsTemplate.convertAndSend("TOPIC.ORDER", message)
    }
}
```

어플리케이션이 실행될 때 메시지를 발송하도록 어플리케이션 실행 코드에 추가 코드를 작성하자.

```kotlin
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
    val app = runApplication<DemoApplication>(*args)
    val sender = app.getBean(MessageSender::class.java)
    
    (1..10).forEach { 
        sender.send(OrderMessage(payload = "test-message-${it}"))
    }
}
```

**Consumer**

메시지 수신을 위한 Consumer 서버를 추가해 보자. 메시지가 정상적으로 Consumer에게 분산되어 전송되는지 확이하기 위해 동일한 서버를 하나 더 추가하자.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

Consumer도 Topic Model로 동작하게 하기 위해서 `pub-sub-domain`속성을 `true`로 변경해 주어야 한다. (추가된 서버의 포트는 이전에 설정한 포트 외에 다른 포트로 설정하자.)

```yml
server:
  port: 8081
spring:
  jms:
      pub-sub-domain: true
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 수신을 위한 빈을 추가한다. Queue Model에서 언급한 바와 같이 어노테이션 기반으로 작성하였다.

```kotlin
@Component
class MessageListener {
    private val logger = LoggerFactory.getLogger(this::class.java)

    @JmsListener(destination = "TOPIC.ORDER")
    fun listen(message: OrderMessage) {
        logger.info("Consumer Listen - > $message")
    }
}
```

**Run**

먼저 Consumer 서버 부터 실행 시키자. Topic Model은 Queue Model과 달리 메시지를 전송한 후 Consumer 서버를 실행하면 이전에 전달된 메시지를 수신할 수 없다. 만약 이전에 전달된 메시지를 수신하고 싶다면 [Duration 설정](http://activemq.apache.org/manage-durable-subscribers){: target="\_blank" }을 통해 수신이 가능하다.

다음으로 Producer 서버를 실행 시키자. Application 코드에 메시지를 전송하는 코드를 넣어 두었기 때문에 서버가 기동되면 자동으로 10개의 메시지를 발송한다.

[![run pub-sub-model](/assets/images/posts/active-mq-tutorial-by-scenario/run-pub-sub-model.gif)](/assets/images/posts/active-mq-tutorial-by-scenario/run-pub-sub-model.gif)

위 영상에서 볼 수 있듯이 Producer가 전달한 메시지를 여러 Consumer가 수신함을 볼 수 있다.

**Monitoring**

전송된 Topic Message들은 Active MQ의 Web Console의 [Topics](http://localhost:8161/admin/topics.jsp){: target="\_blank" } 메뉴에서 확인이 가능하다.

[![admin queue history](/assets/images/posts/active-mq-tutorial-by-scenario/admin-topic-history.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-topic-history.png)

**GIT**

Topic Model에 대한 코드는 아래 Github 주소를 참고하자.

- Producer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/topic-model/activemq-sender](https://github.com/veluxer62/activemq-tutorial/tree/topic-model/activemq-sender){: target="\_blank" }

- Consumer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/topic-model/activemq-receiver](https://github.com/veluxer62/activemq-tutorial/tree/topic-model/activemq-receiver){: target="\_blank" }

- Consumer Server2

  [https://github.com/veluxer62/activemq-tutorial/tree/topic-model/activemq-receiver2](https://github.com/veluxer62/activemq-tutorial/tree/topic-model/activemq-receiver2){: target="\_blank" }

<br/>

### 하나의 메시지를 다수의 사용자에게 전달해야 하는 경우에서 특정 서버가 스케일 아웃이 필요한 경우

만약 Topic Model을 사용하여 특정 메시지를 구독하고 있는 Consumer 서버가 수신된 메시지를 처리하는 부하를 분산하기 위해 스케일 아웃을 해야 하는 상황이라고 가정해 보자. Consumer 서버를 스케일 아웃 한다면 Topic Model에서는 아래 그림과 같이 구독자가 추가되는 것 과 동일하게 된다.

[![publisher subscriber model scale out](/assets/images/posts/active-mq-tutorial-by-scenario/pub-sub-model-scale-out.png)](/assets/images/posts/active-mq-tutorial-by-scenario/pub-sub-model-scale-out.png)

스케일 아웃된 서버가 주문을 처리하는 서버라면 위 그림과 같은 구조라면 사용자가 요청한 동일한 주문이 2개가 저장되는 상황이 발생하게 된다. 우리가 원하는 것은 주문을 처리하는 서버는 사용자의 주문 요청을 한번만 처리하면서 나머지 구독 서버들은 동일하게 주문요청을 수신할 수 있는 구조를 원하는 것이다.

이렇듯 하나의 메시지를 다수의 사용자에게 전달해야 하는 경우에서 특정 Consumer 서버가 스케일 아웃이 필요한 경우에 Virtual Topic을 사용하면 된다. Virtual Topic은 Topic Model에서 Broker와 Consumer 사이에 Queue가 존재하는 구조를 가진다. 그래서 Virtual Topic을 적용한 구독자는 Queue Model 처럼 메시지를 라운드 로빈 방식으로 수신할 수 있게 된다.

[![virtual topic model](/assets/images/posts/active-mq-tutorial-by-scenario/virtual-topic-model.png)](virtual-topic-model.png)

**Code Example**

해당 시나리오에 대한 튜토리얼을 진행하기 위해서는 Producer 서버 하나와 Consumer 서버 세개를 실행하고자 한다. Consumer 서버 두개는 Virtual Topic을 적용하고 나머지 한대는 Topic을 사용할 예정이다.

**ActiveMQ Configuration**

Virtual Topic을 사용하기 위해서는 ActiveMQ의 설정 변경이 필요하다. ([공식 홈페이지 참고](https://activemq.apache.org/virtual-destinations.html){: target="\_blank" })
먼저 `/usr/local/Cellar/activemq/x.x.x/libexec/conf` 디렉토리에 있는 `activemq.xml`파일을 열자.

`broker` 태그에 `useVirtualTopics`속성을 `true`값으로 추가하자.

```xml
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" useVirtualTopics="true">
```

그런다음 `broker` 태그 하위에 `destinationInterceptors` 태그를 아래와 같이 추가하자.

`virtualTopic` 태그의 `prefix`값은 규칙이 있으니 규칙을 꼭 참고해서 사용하자.

예를 들어 `name`속성의 값이 아래와 같이 `>`이고 `prefix`속성 값이 `VirtualTopicConsumers.*.`인 경우 모든 Topic의 Destination Name에 대해 Virtual Topic을 사용할 수 있고, Virtual Topic을 사용하는 Consumer들의 Destination Name은 `VirtualTopicConsumers.{clientId}.{TopicName}`패턴을 사용해야 한다.

또 다른 예를 들자면 `name`속성이 값이 `VirtualTopic.>`이고 `prefix`속성 값이 `Consumer.*.`인 경우 Virtual Topic을 사용하는 Consumer들의 Destination Name은 `Consumer.{clientId}.VirtualTopic.{TopicName}`패턴을 사용해야 한다.

```xml
<destinationInterceptors> 
  <virtualDestinationInterceptor> 
    <virtualDestinations> 
      <virtualTopic name=">" prefix="VirtualTopicConsumers.*." selectorAware="false"/>   
    </virtualDestinations>
  </virtualDestinationInterceptor> 
</destinationInterceptors>
```

설정이 완료되면 MQ를 재시작 해주자.

**Producer**

메시지 전송을 위한 Producer 서버를 추가해보자.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

Topic Model을 사용하기 위해서는 `pub-sub-domain`속성을 `true`로 변경해 주어야 한다.

```yml
# application.yml

server:
  port: 8080
spring:
  jms:
      pub-sub-domain: true
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 전송을 위한 빈을 추가한다. 해당 빈은 `JmsTemplate`을 생성자로 주입 받아서 사용한다.

```kotlin
// MessageSender.kt

@Component
class MessageSender(private val jmsTemplate: JmsTemplate) {
    private val logger = LoggerFactory.getLogger(this::class.java)

    fun send(message: OrderMessage) {
        logger.info("Producer Message -> [$message]")
        jmsTemplate.convertAndSend("TOPIC.ORDER", message)
    }
}
```

어플리케이션이 실행될 때 메시지를 발송하도록 어플리케이션 실행 코드에 추가 코드를 작성하자.

```kotlin
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
    val app = runApplication<DemoApplication>(*args)
    val sender = app.getBean(MessageSender::class.java)
    
    (1..10).forEach { 
        sender.send(OrderMessage(payload = "test-message-${it}"))
    }
}
```

**Topic Consumer**

메시지 수신을 위한 Consumer 서버를 추가해 보자. Topic Consumer는 Topic 모델과 동일하게 설정하면 된다. (여러개를 추가해도 무방하다.)

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

Consumer도 Topic Model로 동작하게 하기 위해서 `pub-sub-domain`속성을 `true`로 변경해 주어야 한다. (추가된 서버의 포트는 이전에 설정한 포트 외에 다른 포트로 설정하자.)

```yml
server:
  port: 8081
spring:
  jms:
      pub-sub-domain: true
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 수신을 위한 빈을 추가한다. Queue Model에서 언급한 바와 같이 어노테이션 기반으로 작성하였다.

```kotlin
@Component
class MessageListener {
    private val logger = LoggerFactory.getLogger(this::class.java)

    @JmsListener(destination = "TOPIC.ORDER")
    fun listen(message: OrderMessage) {
        logger.info("Consumer Listen - > $message")
    }
}
```

**Virtual Topic Consumer**

Virtual Topic을 수신하기 위한 Consumer 서버를 추가해 보자. 메시지가 라운드 로빈 방식으로 수신되는 지 확인하기 위해 최소 2대 서버를 설정해주자.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

Virtual Topic은 Queue Model 처럼 동작하기 때문에 `pub-sub-domain`속성을 `false`로 변경해 주어야 한다. (추가된 서버의 포트는 이전에 설정한 포트 외에 다른 포트로 설정하자.)

```yml
server:
  port: 8082
spring:
  jms:
      pub-sub-domain: false
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 수신을 위한 빈을 추가한다. Queue Model에서 언급한 바와 같이 어노테이션 기반으로 작성하였다.

Topic Model의 `destination`속성 값과 달리 Virtual Topic을 수신하기 위한 명명규칙을 사용한 것에 주의하자.

```kotlin
@Component
class MessageListener {
    private val logger = LoggerFactory.getLogger(this::class.java)

    @JmsListener(destination = "VirtualTopicConsumers.server1.TOPIC.ORDER")
    fun listen(message: OrderMessage) {
        logger.info("Consumer Listen - > $message")
    }
}
```

**Run**

Topic Model에서 실행했던 방식 처럼 먼저 Consumer 서버 부터 실행 시키자. 

다음으로 Producer 서버를 실행 시키자. Application 코드에 메시지를 전송하는 코드를 넣어 두었기 때문에 서버가 기동되면 자동으로 10개의 메시지를 발송한다.

[![run pub-sub-model](/assets/images/posts/active-mq-tutorial-by-scenario/run-virtual-topic-model.gif)](/assets/images/posts/active-mq-tutorial-by-scenario/run-virtual-topic-model.gif)

위 영상에서 볼 수 있듯이 Producer가 전달한 메시지를 3개의 Consumer가 수신함을 볼 수 있다. Topic Model을 사용하는 Consumer의 경우 10개의 메시지를 수신하고, Virtual Topic Model을 사용하는 Consumer들의 경우 각각 5개의 메시지를 나누어 수신한 것을 볼 수 있다.

**Monitoring**

전송된 Topic Message들은 Active MQ의 Web Console에서 Topic Model의 경우 [Topics](http://localhost:8161/admin/topics.jsp){: target="\_blank" } 메뉴에서 Virtual Topic Model의 경우 [Queues](http://localhost:8161/admin/queues.jsp){: target="\_blank" } 메뉴에서 확인이 가능하다.

[![admin queue history](/assets/images/posts/active-mq-tutorial-by-scenario/admin-virtual-queue-history.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-virtual-queue-history.png)

[![admin queue history](/assets/images/posts/active-mq-tutorial-by-scenario/admin-virtual-topic-history.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-virtual-topic-history.png)

**GIT**

Virtual Topic Model에 대한 코드는 아래 Github 주소를 참고하자.

- Producer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-sender](https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-sender){: target="\_blank" }

- Virtual Topic Consumer Server 

  [https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-receiver](https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-receiver){: target="\_blank" }

- Virtual Topic Consumer Server2

  [https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-receiver2](https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-receiver2){: target="\_blank" }

- Topic Consumer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-receiver3](https://github.com/veluxer62/activemq-tutorial/tree/virtual-topic-model/activemq-receiver3){: target="\_blank" }

<br/>

### 전송된 메시지를 특정 기간 이후에 수신해야 하는 경우

만약 사용자가 주문을 한 후에 일주일 뒤에 주문한 상품이 잘 도착했는지 알림을 발송해야 하는 기능을 구현해야 한다고 생각해보자. 

ActiveMQ를 사용하지 않는 경우라면 배치 프로세스를 두어서 주문한 상품이 주문한 시간을 체크해서 주문 후 7일이 경과했다면 메시지를 발송하는 방식을 생각할 수 있겠다. 배치프로세스를 구현할 때 얼마나 자주 Polling을 할 건지 고민을 해보아야 할것이다. 시간의 정확도가 높아지려면 Polling 주기를 더 자주 해야 할 것이니 시스템의 부하가 증가할 것이고 Polling주기를 줄인다면 시스템 부하는 줄일 수 있지만 메시지 전달 시간의 정확도가 줄어들 수 밖에 없을 것이다.

ActiveMQ의 Delay Message 기능을 사용한다면 위에서 고민했던 문제를 해결할 수 있다. Producer는 사용자의 주문 처리 후 7일 뒤에 알림을 발송하도록 하는 Delay Message를 전송한다. 알림 발송 Consumer는 Delay Message를 수신하면 단순히 알림을 발송하기만 하면된다. 이 방식은 위에서 고민하였던 시간 정확도를 충족시켜주면서 Polling에 대한 서버 부하도 걱정하지 않아도 되는 장점을 누릴 수 있다.

Active MQ의 `Delay and Schedule Message Delivery`기능에는 지연 전송 뿐만 아니라 Schedule 설정을 통해 특정 시간에 전송하게도 할 수 있으며, 특정 시간 마다 반복 전송하도록 설정할 수도 있다. Active MQ의 `Delay and Schedule Message Delivery` 기능과 관련된 내용은 [공식 홈페이지](http://activemq.apache.org/delay-and-schedule-message-delivery){: target="\_blank" }에서 참고하자.

**Code Example**

튜토리얼을 진행하기 위해 Producer와 Consuemr 서버를 각 1개씩 생성하자. 

**ActiveMQ Configuration**

Active MQ의 `Delay and Schedule Message Delivery` 기능을 사용하려면 먼저 Active MQ의 설정값을 변경해 주어야 한다.

`broker` 태그에 `schedulerSupport`속성 값을 `true`로 해서 추가 하자.

```xml
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" useVirtualTopics="true" schedulerSupport="true">
```

설정을 완료했다면 MQ를 재시작 해주자.

**Producer**

메시지 전송을 위한 Producer 서버를 추가해보자.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

전송 모델은 어떤걸 사용해도 무방하다. 이 글에서는 단순히 Queue Model을 사용할 것이므로 Queue Model의 설정 값을 그대로 사용하였다.

```yml
# application.yml

server:
  port: 8080
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 전송을 위한 빈을 추가한다. 해당 빈은 `JmsTemplate`을 생성자로 주입 받아서 사용한다. 전송방식은 이전에 보여줬던 시나리오랑 조금 다른데, `convertAndSend`함수에 `MeesagePostProccessor` 인자를 추가하여 지연 시간을 설정하였다. 해당 튜토리얼에서는 3초로 설정하였다.

```kotlin
// MessageSender.kt

@Component
class MessageSender(private val jmsTemplate: JmsTemplate) {
    private val logger = LoggerFactory.getLogger(this::class.java)

    fun send(message: OrderMessage) {
        logger.info("Producer Message -> [$message]")
        jmsTemplate.convertAndSend("ORDER", message) {
            it.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, 3000)
            it
        }
    }
}
```

어플리케이션이 실행될 때 메시지를 발송하도록 어플리케이션 실행 코드에 추가 코드를 작성하자. 메시지는 한번만 전송하며 전송시간을 체크하기 위해 `payload`에 시간을 추가했다.

```kotlin
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
    val app = runApplication<DemoApplication>(*args)
    val sender = app.getBean(MessageSender::class.java)
    
    sender.send(OrderMessage(payload = "test-message[${LocalDateTime.now()}]"))
}
```

**Consumer**

메시지 수신을 위한 Consumer 서버를 추가해 보자. 기존에 작성한 Queue Model과 다른 설정은 없다.

먼저 프로젝트를 생성한다. ([프로젝트생성](#프로젝트생성) 참고)

```yml
server:
  port: 8081
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
```

메시지 수신을 위한 빈을 추가한다. Spring Boot에서 Active MQ의 메시지를 수신하기 위한 여러가지 방법이 있는데 이 글에서는 어노테이션을 이용한 방법으로 진행하였다. 다른 방법에 대해 궁금하다면 [공식 문서 링크](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#jms-receiving){: target="\_blank" }를 참고하자.

```kotlin
@Component
class MessageListener {
    private val logger = LoggerFactory.getLogger(this::class.java)

    @JmsListener(destination = "ORDER")
    fun listen(message: OrderMessage) {
        logger.info("Consumer Listen - > $message")
    }
}
```

**Run**

먼저 Consumer 서버 부터 실행 시킨 후 Producer 서버를 실행 시키자. Application 코드에 메시지를 전송하는 코드를 넣어 두었기 때문에 서버가 기동되면 자동으로 1개의 메시지를 발송한다.

[![run queue-model](/assets/images/posts/active-mq-tutorial-by-scenario/run-delay-delivery.gif)](/assets/images/posts/active-mq-tutorial-by-scenario/run-delay-delivery.gif)

영상에서 볼 수 있듯이 Producer가 메시지를 전송한 후 3초가 지난 뒤 Consumer가 메시지를 수신하는 것을 볼 수 있다.

**Monitoring**

`Delay and Schedule Message Delivery`기능으로 전송된 Message들은 Active MQ의 Web Console의 [Scheduled](http://localhost:8161/admin/scheduled.jsp){: target="\_blank" } 메뉴에서 확인이 가능하다.

[![admin queue history](/assets/images/posts/active-mq-tutorial-by-scenario/admin-scheduled-history.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-scheduled-history.png)

**GIT**

Queue Model에 대한 코드는 아래 Github 주소를 참고하자.

- Producer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/delay-delivery/activemq-sender](https://github.com/veluxer62/activemq-tutorial/tree/delay-delivery/activemq-sender){: target="\_blank" }

- Consumer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/delay-delivery/activemq-receiver](https://github.com/veluxer62/activemq-tutorial/tree/delay-delivery/activemq-receiver){: target="\_blank" }

<br/>

### 전송 실패한 메시지 관리

만약에 전송한 주문을 수신하는 서버가 버그로 인해서 제대로 메시지를 수신하지 못하면 어떻게 될까? MQ를 사용하지 않는 서비스인 경우에는 해당 주문은 실패로 끝나버리고 말것이고 별도로 로그 처리를 하지 않는다면 해당 메시지가 전송되었는지 조차 알지도 못하는 상황이 발생하게 될 것이다. Active MQ에서는 Dead Letter Queue라고 해서 전송 시 어떠한 이유로 인해 전송이 실패한 메시지들을 저장해둔다. 우리는 이러한 메시지들을 Web Console에서 확인할 수 있으며 설정을 통해 재전송이 가능하도록 할 수도 있다.

**Default**

기본적으로 Active MQ에서는 하나의 Dead Letter Queue를 두어 관리하도록 설정되어 있고 재전송 기능을 제공하지 않는다.

Web Console의 [Queue](http://localhost:8161/admin/queues.jsp){: target="\_blank" }메뉴에서 Dead Letter Queue 목록을 확인 할 수 있다.

[![admin dead letter queue list](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-default-history.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-default-history.png)

Queue Name을 클릭하면 해당 큐 이름을 가진 Queue들을 볼 수 있다. 이곳에서는 전송된 큐들의 추가 정보 및 메시지를 삭제하는 기능을 제공한다.

[![admin dead letter queue history](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-default-history-view.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-default-history-view.png)

Message ID를 클릭하면 해당 메시지의 좀더 자세한 정보를 볼 수 있다.

[![admin dead letter queue history](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-default-history-view-each.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-default-history-view-each.png)

**Dead Letter Queue Strategy**

기본 설정을 하게 되면 Dead Letter Queue는  `ActiveMQ.DLQ`에 모든 Queue가 쌓이게 된다. 물론 Dead Letter Queue가 쌓이지 않도록 하는게 맞지만 하나의 Queue에 모든 Queue가 담기는 것은 추후 메시지 처리에 어려움이 생길 수 있다. 그래서 Active MQ에서는 Dead Letter Queue Strategy 설정을 통해 Destination 마다 Dead Letter Queue가 나뉘어서 쌓이도록 할 수 있으며 재전송 기능을 활성화 할 수도 있다. Dead Letter Queue 설정과 관련해서는 [공식 홈페이지](http://activemq.apache.org/message-redelivery-and-dlq-handling.html){: target="\_blank" }를 참고하자.

`activemq.xml`파일의 `broker`의 `destinationPolicy` 태그에 아래와 같이 설정값을 추가하자

`queuePrefix`의 값은 Dead Letter Queue를 의미하는 Queue의 머릿글자를 설정하는 것이다. `useQueueForQueueMessages`속성 값은 `true`로 하는 경우 Dead Letter Queue가 Web Console의 `Queues`에 `false`로 하는 경우 `Topics`에 쌓인다.

```xml
<policyEntry queue=">">
  <deadLetterStrategy>
      <individualDeadLetterStrategy queuePrefix="DLQ." useQueueForQueueMessages="true"/>
  </deadLetterStrategy>
</policyEntry>
```

설정 완료 후 MQ를 재시작 해주자.

설정 완료 후 메시지 전송이 실패하면 Web Console에서 아래와 같이 Destination 명칭별로 Dead Letter Queue가 생성됨을 확인할 수 있다.

Web Console의 [Queue](http://localhost:8161/admin/queues.jsp){: target="\_blank" }메뉴를 보면 Dead Letter Queue의 이름이 바뀌었고, 다른 Destination Name을 가진 Queue의 경우 별도로 쌓인다는 것을 볼 수 있다.

[![admin dead letter queue stratege](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-strategy-queue.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-strategy-queue.png)

추가로 Queue Name의 이름을 클릭하고 들어가면 해당 Destination의 Dead Letter Queue의 이력을 볼 수 있는데, 그곳에서 기본 설정에서는 보이지 않던 `retry` 버튼이 생성된 것을 확인할 수 있다. 해당 버튼을 통해 우리는 실패한 메시지를 다시 전송할 수 있다.

[![admin dead letter queue stratege retry](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-strategy-retry.png)](/assets/images/posts/active-mq-tutorial-by-scenario/admin-dlq-strategy-retry.png)

**GIT**

Dead Letter Queue를 테스트 하기 위한 코드는 아래 링크에서 참고하자.

- Producer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/dlq-demo/activemq-sender](https://github.com/veluxer62/activemq-tutorial/tree/dlq-demo/activemq-sender){: target="\_blank" }

- Consumer Server

  [https://github.com/veluxer62/activemq-tutorial/tree/dlq-demo/activemq-receiver](https://github.com/veluxer62/activemq-tutorial/tree/dlq-demo/activemq-receiver){: target="\_blank" }

<br/>

## Wrap up

지금까지 시나리오별로 Active MQ의 사용 사례를 튜토리얼을 통해 설명하였다. 앞서 소개한 기능 외에도 Active MQ는 다양하고도 유용한 기능들을 제공해준다. 지금까지 소개한 시나리오는 꼭 Active MQ만으로 해결가능한 문제는 아니다. 어느 Message Queue를 사용해도 상관없다. 다만 Active MQ는 Spring Boot와 호환성이 좋고 최근에 AWS에서 Amazon MQ를 제공하면서 클라우드 환경에서도 Active MQ를 손쉽게 사용할 수 있게 되면서 Mesage Queue 서버 관리에 대한 부담감도 줄일 수 있다. 또, 인메모리 서버도 제공하면서 테스팅 또한 용이하다. 어플리케이션을 개발하면서 Message Queue를 사용하면 기존의 구현방식에서 훨씬 더 높은 신뢰성과 확장성을 누릴 수 있다. 만약 현재 구현하는 어플리케이션의 기능이 위에서 설명한 시나리오와 비슷한 점이 있다면, Active MQ로 한번 시도해 보는건 어떨까?

> [https://brewinstall.org/install-activemq-on-mac-with-brew/](https://brewinstall.org/install-activemq-on-mac-with-brew/){: target="\_blank" }
>
> [http://activemq.apache.org/](http://activemq.apache.org/){: target="\_blank" }
>
> [https://spring.io/guides/gs/messaging-jms/](https://spring.io/guides/gs/messaging-jms/){: target="\_blank" }
>
> [https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#jms-receiving](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#jms-receiving){: target="\_blank" }
>
