---
layout: single
title: "Message Queue Comparison"
categories:
  - REFERENCE
tags:
  - message queue
  - message brocker
  - rabbit mq
  - apache kafka
  - amazon sqs
  - amazon sns
  - amazon mq

toc: true
toc_sticky: true
excerpt: 사내 예약 시스템을 만들면서 서버간의 데이터 교환에 Message Queue를 사용하기로 결정하였다. Message Queue 기능을 제공하는 다양한 솔루션이 있는데 이중에서 어떤 것을 사용하면 좋을 지 알아보면서 Message Queue란 어떤것이며 종류가 어떤것 들이 있는지 알아보고 우리가 구현하고자 하는 시스템의 요구사항에 맞는 Message Queue를 선택해 보고자 한다.
---

## Intro

사내 예약 시스템을 만들면서 서버간의 데이터 교환에 Message Queue를 사용하기로 결정하였다. Message Queue 기능을 제공하는 다양한 솔루션이 있는데 이중에서 어떤 것을 사용하면 좋을 지 알아보면서 Message Queue란 어떤것이며 종류가 어떤것 들이 있는지 알아보고 우리가 구현하고자 하는 시스템의 요구사항에 맞는 Message Queue를 선택해 보고자 한다.

## Message Queue?

Message Queue란 MOM(Message Oriented Middleware)을 구현한 시스템을 말하는데, 비동기적으로 분산 응용프로그램간에 메시지를 주고 받을 수 있는 기능을 제공한다.

### Usage

- API 서버와의 통신

  MSA나 외부 API를 이용한 통신시 Message Queue를 이용할 수 있다. 물론 API끼리 직접 통신하는 방식이 단순하고 빠를 순 있지만 부하분산이라던지 데이터 손실 방지 등을 위해서 메시지 큐를 사용할 수 있다.

- 어플리케이션간의 비동기 통신

  Message Queue는 비동기적으로 메시지를 전달한다. 비동기적으로 통신함으로써 어플리케이션은 좀더 효율적으로 자원을 사용할 수 있기 때문에 비동기적 통신이 필요한 어플리케이션에서 많이 사용하고 있다.

- 이메일 발송 및 문서 업로드

  Message Queue는 메시지의 전달에 대한 높은 신뢰성을 제공하므로 이메일이나 문서 업로드와 같이 응답을 기다리는 동기성 작업에는 비효율적이면서 메시지 전송에 대한 높은 신뢰를 요하는 기능에 적합하다.

- 다량의 프로세스 처리

  Message Queue는 높은 가용성과 다량의 메시지를 처리하기 위한 메커니즘을 제공하므로 다량의 프로세스를 처리하기 위한 용도로 많이 사용되고 있다.

### 장점

- Asynchronous

  Queue에 넣어서 처리하기 때문에 비동기적으로 처리할 수 있다.

- Decoupling

  어플리케이션과 분리하여 사용가능 하다.

- Resilience

  일부가 실패하더라도 전체에 영향을 미치지 않는다.

- Redundancy

  실패할 경우 재실행이 가능하다.

- Guarantees

  작업의 완료여부를 확인할 수 있다.

- Scalable

  다수의 프로세스들이 큐에 메시지를 보낼 수 있다.

### 단점

- Resource

  큐를 구성하기 위한 별도 자원이 소모된다.

- Loose Coupling

  느슨한 결합은 장점이 될 수도 있지만 시스템에 따라 단점이 될 수도 있다.

- Depend on Middleware

  현재 다양한 Message Queue들이 존재한다. 각 시스템마다 가진 장/단점이 존재하는데 Message Queue를 도입할 때 어플리케이션이 어느 Message Queue에 적합한지 충분한 고려가 필요하다.

## Rabbit MQ

Rabbit MQ는 [AMQP(Advanced Message Queuing Protocol)](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol){: target="\_blank" }을 구현한 오픈소스 메시지 브로커이다. [STOMP(Streaming Text Oriented Messaging Protocol)](https://en.wikipedia.org/wiki/Streaming_Text_Oriented_Messaging_Protocol){: target="\_blank" }와 [MQTT(MQ Telemetry Transport)](https://en.wikipedia.org/wiki/MQTT){: target="\_blank" } 등을 지원하는 플러그인을 통해 확장 가능하다.

### AMQP

Advanced Message Queuing Protocol의 약자로 ISO응용계층의 MOM(Message Oriented Middleware)의 표준 프로토콜이다.
메시지 전달을 아래 3가지 방식 중 하나를 보장한다.

- At-Most-Once: 각 메시지는 한번만 전달되거나 전달되지 않음.
- At-Least-Once: 각 메시지는 최소 한번 이상 전달됨을 보장.
- Exactly-Once: 각 메시지는 딱 한번만 전달됨.

### 특징

- ISO 표준(ISO/IEC 19464) AMQP 구현
- 동기와 비동기 메시징 제공
- 클러스터링 제공
- 고가용성
- Publish / Subscribe 방식
- 다양한 Plugin 지원

### Usage

RabbitMQ는 범용 메시징 솔루션으로, 사용자가 결과를 기다리는 동안 리소스 사용량이 많은 절차를 수행하지 않고 웹 서버가 요청에 신속하게 응답할 수 있도록 하는 데 종종 사용된다. 메시지를 여러 수신자에게 배포하여 소비하거나 부하가 많은 작업자 간에 부하를 분산시키는 데도 유용하다. 요구사항이 처리량을 초과할 경우, RabbitMQ는 신뢰할 수 있는 전달, 라우팅, 연합, HA, 보안, 관리 도구 및 기타 기능 등 다양한 기능을 제공한다.

Rabbit MQ는 아래와 같은 시나리오에서 사용하기에 적합할 수 있다.

- AMQP 0-9-1, STOMP, MQTT, AMQP 1.0과 같은 기존 프로토콜의 조합으로 작업해야 하는 경우
- 메시지별 정합성 관리/보증이 필요한 경우
- 시점별 다양성, 요청/응답, 게시/구독 메시지가 필요한 경우
- 소비자에 대한 복잡한 라우팅, 여러 서비스/앱과 타의 추종을 불허하는 라우팅 논리 통합

## Apache Kafka

Apache Kafka는 LinkedIn이 개발하고 Apache Software Foundation에 기부한 오픈 소스 스트림 프로세싱 소프트웨어 플랫폼이다. 높은 처리량을 요구하는 실시간 데이터 피드 처리나 대기 시간이 짧은 플랫폼을 제공하는 것을 목표로 하며 TCP 기반 프로토콜을 사용한다. 클러스터를 중심으로 Producer와 Consumer가 데이터를 Push하고 Pull하는 구조를 가진다.

### 특징

- Publisher / Subscriber 모델
- 고가용성
- 확장성
- 디스크 순차 저장 및 처리
- 분산 처리 (Partitioning)

### Usage

카프카는 내구성이 뛰어난 메시지 저장소로, 고객들은 메시지가 한 번 배달되면 대기열에서 제거되는 전통적인 메시지 중개업자들과는 달리, 필요에 따라 이벤트 스트림을 "재생"할 수 있다.

Apache Kafka는 아래와 같은 시나리오에서 사용하기에 적합할 수 있다.

- 최대 처리량(100k/sec+)으로 복잡한 라우팅 없이 A에서 B로 스트리밍, 분할된 순서대로 1회 이상 제공
- 응용 프로그램에서 스트림 히스토리에 대한 액세스 권한이 필요한 경우, 분할된 순서대로 한 번 이상 제공
- 내구성이 뛰어난 메시지 저장소로, 고객들은 메시지가 한 번 배달되면 대기열에서 제거되는 전통적인 메시지 브로커와는 달리, 필요에 따라 이벤트 스트림을 다시 재생하여야 하는 경우
- 스트림 처리
- 이벤트 소싱

## Amazon SQS

Amazon SQS(Simple Queue Service)는 2004년말 부터 Amazon.com에서 제공하는 분산 메시지 큐 서비스이다. Amazon SQS를 사용하기 위해 Java, Ruby, Python등 다양한 언어로된 SDK를 지원한다.

### 특징

- Security

  메시지를 주고 받을 수 있는 사용자를 제어할 수 있는 기능을 제공한다.

- Durability

  메시지의 안전을 보장해 주기 위해 메시지를 여러 서버에 저장해 두며 **표준 대기열**은 최소 한 번 이상의 메시지 전달을 보장하며, **FIFO 대기열**은 정확히 한번 메시지 처리를 지원한다.

- Availability

  메시지의 높은 동시 엑세스 제어와 메시지의 생산과 소비를 위한 고가용성 을 제공하기 위해Redundant Infrastructure를 사용한다.

- Scalability

  버퍼 요청을 독립적으로 처리할 수 있으며, 프로비저닝된 지침없이 로딩의 증가또는 급증을 다루는 확실한 확장성을 제공해준다.

- Reliability

  메시지 처리 중 해당 메시지를 잠그므로 동시에 메시지를 전송 및 수신할 수 있다.

- Customization

  Queue에 지연시간을 지정한다거나 256KB가 넘는 대규모 메시지를 전송하기 위해 S3나 DynamoDB를 사용하는 것과 같이 다양한 Queue를 사용할 수 있다.

### Lifecycle

[![sqs-message-lifecycle-diagram](/assets/images/posts/message-queue/sqs-message-lifecycle-diagram.png)](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-message-lifecycle-diagram.png)

1. Publisher(Component 1)는 메시지 A를 Queue로 전송하고 이 메시지는 Amazon SQS 서버에서 중복 분산
2. Consumer(Component 2)는 메시지를 처리할 준비가 되면 Queue에서 메시지를 소비하고 메시지 A가 반환. 메시지 A는 처리되는 동안 대기열에 그대로 남아 있고 제한 시간 초과가 지속되는 동안 후속 수신 요청으로 반환되지 않음.
3. Consumer(Conponent 2)는 Queue에서 메시지 A를 삭제하여 제한 시간 초과가 만료되면 이 메시지가 수신되어 다시 처리되지 못하도록 함.

### Standard Queue vs FIFO Queue

- Standard Queue

  Amazon SQS는 기본적으로 Standard Queue를 사용한다. 거의 무제한의 TPS를 제공할만큼 성능이 좋으며 최소 1회 이상의 전송을 보장합니다. 일반적으로 메시지가 전송한 순서와 동일한 순서로 전송되도록 최선의 정렬을 제공하지만 순서가 맞지 않게 전송 될 수 있다.

- FIFO Queue

  작업및 이벤트의 순서가 중요한 메시지의 경우 FIFO Queue를 사용할 수 있다. FIFO Queue는 정확히 1회 처리를 제공하지만 TPS의 수가 제한적이다. (일괄처리 시 초당 최대 3000개, 개당 처리 시 초당 최대 300개)

### Pricing

매월 처음 백만건은 무료로 사용이 가능하다.

|                | 프리티어 이후 요청 백만건 당 요금(월별) |
| :------------- | :-------------------------------------: |
| Standard Queue |                0.40 USD                 |
| FIFO Queue     |                0.50 USD                 |

가격 책정 시 주의해야 할 점은 Amazon SQS는 메시지의 Consumer가 Polling 방식으로 메시지를 소비한다는 점이다. Amazon SQS의 가격정책에서 보면 모든 API 요청(Sending, Deleting, Receiving)별로 가격을 책정하기 때문에 Queue에 메시지가 존재하던 존재하지 않던 요청 Polling으로 인한 요청 비용이 발생한다는 것이다. <br>
또, 메시지 전송 시 최대 64KB까지만 1개의 요청으로 간주한다는 점이다. 예를 들면 200KB의 메시지를 전송했다고 가정하면 4개의 Send Request와 4개의 Receive Request, 그리고 1개의 Delete Request가 발생한다.

비용 절감을 위해서는 2가지 방법을 사용할 수 있다.

- Long Polling

  메시지가 주기적이지 않고 자주 발생하지 않는다면 Long Polling을 이용하여 (최대 20초) Polling의 횟수를 줄일 수 있으므로 비용을 절감할 수 있다.

- Batch Request

  개별 메시지를 일괄적으로 처리함으로써 API 요청 횟수를 줄일 수 있다. Amazon SQS에서는 Batch 요청을 위한 API (`SendMessageBatch`, `DeleteMessageBatch`)를 제공해준다.

## Amazon MQ

Amazon.com에서 제공하는 Apache ActiveMQ용 관리형 메시지 브로커 서비스이다. 기존에 사용하던 JMS 등과 같은 API 또는 AMQP, MQTT, OpenWire 및 STOMP와 같은 프로토콜과 호환되는 기존 메시지 브로커의 애플리케이션을 마이그레이션할 때 유용하다. Amazon MQ는 2018년 7월부터 아시아 리전에서 지원하기 시작하였다.

### ActiveMQ

Apache ActiveMQ는 JMS(Java Message Service)클라이언트와 함께 Java로 개발된 오픈 소스 메시지 브로커이다. 빠르며, 다양한 언어간의 클라이언트 및 프로토콜을 지원하고, 사용하기 쉬운 엔터프라이즈 통합 패턴 및 많은 고급 기능을 제공하면서 JMS 1.1 및 J2EE 1.4를 완벽하게 지원한다. JMS를 지원하는 클라이언트를 포함하는 브로커, 자바 뿐만 아니라 다양한 언어를이용하는시스템간의 통신을 할 수 있게 해준다. 또한 클러스터링기능 및 DB 그리고 FileSystem을 통해 각 시스템간의 일관성 및 지속성을 유지 시켜준다.

### JMS

JMS(Java Message Service)는 Java의 MOM(Message Oriented Middleware) API 이다. 자바 플랫폼, 엔터프라이즈 에디션(EE) 기반이며, 메시지 생성, 송수신, 읽기를 하며 또한 비동기적이며 신뢰할 만하고 느슨하게 연결된 서로 다른 분산 어플리케이션 컴포넌트 간의 통신을 허용한다. 하지만 다른 MOM(AMQP, SMTP 등)과의 통신은 불가능하다.

### Model

[![activemq-diagram](/assets/images/posts/message-queue/activemq-diagram.png)](https://imgix.datadoghq.com/img/blog/activemq-architecture-and-metrics/activemq_diagram2.png?auto=format&fit=max&w=847&dpr=2)

- Point-to-point model

  Publisher 메시지를 Queue에 전송하면 각 메시지는 하나의 Consumer가 소비하는 것을 보장한다. Queue는 Consumer가 메시지를 소비하는 동안 다른 메시지들을 만료시간 까지 보관한다. 만약 등록된 Consumer가 없는경우 Queue는 Consumer가 등록될 때까지 메시지를 보관한다.

- Publish-and-subscribe model

  Publisher는 **Topic**을 발행하고 0개 이상의 Subscriber가 구독하여 메시지를 소비하는 형태이다. Publisher와 Subscriber는 타이밍에 의존하는데, Subscriber는 구독을 시작한 이후에 Publisher가 발행한 메시지만 소비할 수 있다. 해당 문제를 해결하기 위해서 JMS는 Durable Subscription을 제공한다.

  해당 모델에서는 메시지 중복 처리에 대한 문제가 발생할 수 있는데 Virtual Destinations를 활용하여 문제를 해결 할 수 있다. 해당 문제 해결 사례는 [링크](https://ryan-han.com/post/server/activemq_virtual_destinations/){: target="\_blank" }를 참고하자

### Pricing

Amazon MQ는 브로커 요금과 브로커 스토리지 요금을 별도로 지불해야 한다.

- 브로커 요금

단일 인스턴스

| 모델          | CPU | Memory | 시간당 요금 |
| :------------ | :-: | :----: | :---------: |
| mq.t2.micro   |  1  |  1GB   |  0.037 USD  |
| mq.m5.large   |  2  |  8GB   |  0.354 USD  |
| mq.m5.xlarge  |  4  |  16GB  |  0.708 USD  |
| mq.m5.2xlarge |  8  |  32GB  |  1.416 USD  |
| mq.m5.4xlarge | 16  |  64GB  |  2.832 USD  |
| mq.m4.large   |  2  |  8GB   |  0.37 USD   |

단일 인스턴스 스토리지

| 스토리지 유형             |              GB-월당 요금              |
| :------------------------ | :------------------------------------: |
| 내구성 최적화(Amazon EFS) | 매월 첫 5GB는 무료, 그 이후에 0.33 USD |
| 처리량 최적화(Amazon EBS) |               0.114 USD                |

활성/대기 인스턴스

| 모델          | CPU | Memory | 시간당 요금 |
| :------------ | :-: | :----: | :---------: |
| mq.t2.micro   |  1  |  1GB   |  0.074 USD  |
| mq.m5.large   |  2  |  8GB   |  0.708 USD  |
| mq.m5.xlarge  |  4  |  16GB  |  1.416 USD  |
| mq.m5.2xlarge |  8  |  32GB  |  2.832 USD  |
| mq.m5.4xlarge | 16  |  64GB  |  5.664 USD  |
| mq.m4.large   |  2  |  8GB   |  0.74 USD   |

활성/대기 인스턴스 스토리지

| 스토리지 유형             |              GB-월당 요금              |
| :------------------------ | :------------------------------------: |
| 내구성 최적화(Amazon EFS) | 매월 첫 5GB는 무료, 그 이후에 0.33 USD |

## 선정기준

- 설치 및 운영 손쉽고 인력투입의 최소화

  사내에는 Message Queue를 설치하고 운영할 수 있는 전문 엔지니어가 없다. 그래서 개발자가 손쉽게 Message Queue를 구축하고 운영할 수 있어야 한다.

- 모니터링 용이성

  장애 발생 시 손쉬운 모니터링과 대응을 위한 모니터링 도구나 UI를 제공하면 좀더 운영이 용이할 듯 하다.

- 속도보다는 높은 안정성

  현재 개발하려고 하는 예약 시스템에서 중요한 것은 높은 성능보다는 높은 안정성이다. 네트워크 트래픽 로그와 같이 다량의 메시지를 실시간으로 처리하기 위한 성능보다는 하나의 메시지가 정확하게 다른 어플리케이션에게 잘 전달함을 보장해주는 안정성이 중요하다.

- Spring Boot 환경에서의 개발 용이성

  우리 팀에서 개발 예정인 예약 시스템은 Spring Boot를 이용하여 개발할 예정이다. Spring Boot에서는 위에서 언급한 모든 Message Queue를 손쉽게 사용하기 위한 추상 인터페이스를 지원하므로 개발 환경에 대한 문제는 없을 듯 하다.

- 테스트 용이성

  로컬환경에서 유닛 테스팅을 위한 임베디드 인스턴스를 제공하면 좋을 것 같다. 테스트 환경을 위한 서버를 별도로 구축하는 것은 비용도 비용이지만 개발자마다 별도의 Message Queue 서버를 제공해야 하는 상황이 생길 수 있기 때문에 고려하고 싶지 않고 로컬 환경에서 실행하기 위한 설치 스크립트를 만드는 것도 비용이라 생각하기에 RDB나 MongoDB와 같이 인메모리나 Mock Instance를 사용할 수 있으면 좋겠다.

- 합리적인 비용

  서버 사용에 대한 비용이 합리적이어야 한다. 여기에서 **합리적**은 가격대비 사용에 편리함이 어느 정도인지 이다.

## Wrap up

지금까지 우리 팀에서 사용할 Mesage Queue에 대해 알아보고 어느 것을 사용할지 비교해 보았다. 현재 내가 생각하는 후보군은 **Amazon SQS**와 **Amazon MQ**이다. 아무래도 서버 설치 및 운영에 대한 부담감을 줄일 수 있고 안정성을 높일 수 있다는 점에서 직접 설치 및 관리를 해야하는 **Rabbit MQ**와 **Apache Kafka**는 고려 대상에서 제외하였다.

**Amazon SQS**의 경우 백만건당 월 0.4 USD 또는 0.5 USD라는 저렴한 가격으로 사용할 수 있다는 점에서 **Amazon MQ**에 비해 큰 장점을 가진다.(mq.t2.micro 기준 월 93 USD) 가격적인 부분만 고려한다면 **Amazon SQS**를 선택해야 하지만 동작하는 방식의 다름과 (Polling을 이용한 메시지 소비 방식, 메시지 소비 후 Delete를 해야 함, 순서를 보장하지 않음 - FIFO를 이용하면 해결 가능) 실제 API 요청 개수가 운영하였을 때 어느정도인지 예상하기 힘든 부분(Polling 시 요청횟수는 요금에 반영됨)이 있어 팀원과 좀더 논의가 필요해 보인다.

> [https://docs.oracle.com/cd/E19148-01/820-0532/6nc919fai/index.html](https://docs.oracle.com/cd/E19148-01/820-0532/6nc919fai/index.html){: target="\_blank" }
> [https://12bme.tistory.com/176](https://12bme.tistory.com/176){: target="\_blank" }
>
> [https://skibis.tistory.com/310](https://skibis.tistory.com/310){: target="\_blank" }
>
> [https://tanzu.vmware.com/content/blog/understanding-when-to-use-rabbitmq-or-apache-kafka](https://tanzu.vmware.com/content/blog/understanding-when-to-use-rabbitmq-or-apache-kafka){: target="\_blank" }
>
> [https://thedulinreport.com/2015/09/05/top-ten-differences-between-activemq-and-amazon-sqs/](https://thedulinreport.com/2015/09/05/top-ten-differences-between-activemq-and-amazon-sqs/){: target="\_blank" }
>
> [https://medium.com/@cpackingham1/sqs-pricing-model-tips-alternatives-9292beab21da](https://medium.com/@cpackingham1/sqs-pricing-model-tips-alternatives-9292beab21da){: target="\_blank" }
>
> [https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html){: target="\_blank" }
>
> [https://swiftymind.tistory.com/9](https://swiftymind.tistory.com/9){: target="\_blank" }
>
> [https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html](https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html){: target="\_blank" }
>
> [https://ryan-han.com/post/server/activemq_virtual_destinations/](https://ryan-han.com/post/server/activemq_virtual_destinations/){: target="\_blank" }
>
> [https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=144&dbnum=128400&mode=detail&type=techreport](https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=144&dbnum=128400&mode=detail&type=techreport){: target="\_blank" }
