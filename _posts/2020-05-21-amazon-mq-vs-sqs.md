---
layout: single
title: "Amazon MQ vs Amazon SQS"
categories:
  - EXPLANATION
tags:
  - message queue
  - message brocker
  - rabbit mq
  - amazon sqs
  - amazon mq

toc: true
toc_sticky: true
excerpt: 이전에 작성한 글 Message Queue Comparison에서 새로운 프로젝트에 도입할 Message Queue들 중 Amazon MQ와 Amazon SQS를 후보군으로 채택하였었다. 이번 글은 Amazon MQ와 Amazon SQS 중에서 Amazon MQ를 선정한 배경 및 이유에 대해서 작성해 보고자 한다.
---

## Intro

이전에 [작성한 글](/reference/message-queue/){: target="\_blank" }에서 새로운 프로젝트에 도입할 Message Queue들 중 Amazon MQ와 Amazon SQS를 후보군으로 채택하였었다. 이번 글은 Amazon MQ와 Amazon SQS 중에서 Amazon MQ를 선정한 배경 및 이유에 대해서 작성해 보고자 한다.

## Amazon MQ

### 특징

#### 높은 신뢰성

관리형 Apache ActiveMQ로 Active/Stand-By Broker를 지원하면서 여러 가용영역에 걸치 메시지를 저장하므로 높은 신뢰성을 보장한다.

#### 높은 호환성

다양한 API, Cross Language(Java, C, C++, C#, Ruby, Perl, Python, PHP, Node.js, GO) 및 프로토콜(JMS, NMS, MQTT, AMQP, STOMP, WebSocket)을 지원한다. 이는 간편하게 어플리케이션을 마이그레이션을 할 수 있도록 도와주며 Cloud에 종속적이지 않도록 도와준다.

#### 다양한 기능

- Queue, Topic (FIFO 지원)을 지원해준다.
- 일시적 & 지속적인 메시지 전송이 가능하다.
- 합성 & 가상 목적지 전송 기능을 제공한다.
- 메시지 재전송, Dead Letter Queue를 지원한다.
- 와일드카드, 메시지 필터링 기능을 제공한다.
- 풍부한 메시지 크기와 메시지 보관을 지원한다.
- 제한 없는 메시지 타이머 기능을 제공한다.

#### 종량 요금제

인스턴스의 사용시간에 따라 요금이 부과되며 Persistence 모드를 사용하는 경우 스토리지 사용량에 따라 추가적인 요금이 부과된다.

### 제한

- 리전당, AWS 계정당 브로커 수는 최대 20개로 제한된다.
- 브로커당 메시지를 저장 용량은 20GB ~ 200GB 이다. (인스턴스 유형에 따라 다름)
- TOPIC : 초당 21-22,000 메시지(1~2KB 메시지 크기), Durable Queue : 초당 2000 메시지 (서버 인스턴스 크기에 따라 다를 수 있음)

## Amazon SQS

### 특징

#### 클라우드 네이티브

AWS에서 사용되는 완전 관리형 MOM(Message Oriented Middleware)이다. 여러 가용영역에 걸친 메시지를 저장하므로 높은 신뢰성과 수백만 메시지를 발송하고 수신할 수 있는 확장성과 처리능력을 보여준다.

#### 표준 대기열

- Queue에 메시지 PUT, GET에 대한 초당 처리량 제한이 없어 거의 무제한에 가까운 처리량을 보여준다.
- 표준 대기열은 메시지 순서에 대한 보장이 없다. 즉, 저장된 순서로 메시지를 꺼낼 수 있다는 보장이 없는 느슨한 FIFO 기능을 제공한다.
- 적어도 한번은 메시지 전달이 보장되나 2회 이상 동일한 메시지가 전달될 수 있다. 이는 가용성을 위해 메시지가 여러 서버에 저장어 일부 서버가 사용할 수 없는 경우 메시지가 삭제되지 않고 다시 메시지를 수신할 수 있도록 설계되어져 있기 때문이다. 그러므로 어플리케이션 설계시 멱등성을 보장할 수 있도록 설계되어야 한다.

#### FIFO 대기열

- 저장된 순서로 메시지를 꺼낼 수 있도록 메시지의 순서를 보장한다.
- FIFO는 표준대기열과 다르게 정확히 1번만 메시지가 전달되도록 보장한다.
- 초당 300 TPS 처리량을 제공한다. 이는 초당 300번의 전송, 수신, 삭제가 가능함을 의미한다. 만약 Batch 처리(최대 10회)를 하는 경우 최대 3000개 까지 메시지 처리 가능하다.

#### 아키텍처

Producer는 메시지를 Queue로 전송하고 Queue에 있는 메시지가 수신측으로 PUSH 되는 방식이 아닌 Consumer가 별도로 Pull한 후에 Queue의 메시지를 삭제하도록 구성되어 있다.

#### 사용당 요금제

매월 첫 100만건은 무료로 사용이 가능하다. 그 이후에는 요청 백만건당 요금이 부과된다.

### 제한

- 메시지 보존기간

  메시지를 수신 후 삭제하지 않으면 기본적으로 메시지는 4일간 보관하며 보관기간은 60초에서 14일 사이에 변경이 가능하다

- In Flight (메시지 소비자가 수신했으나 아직 큐에서 삭제하지 않은) 메시지

  - 표준 Queue - Queue 당 최대 120,000 In Flight 메시지 (soft limit)
  - FIFO Queue - Queue 당 최대 20,000 In Flight 메시지 (hard limit)

- 요청 메시지의 최대 크기는 256KB

  Batch로 최대 10개 메시지를 모아서 전송 시 전체 함의 256KB를 초과할 수 없다. 만약 Extended Client Library를 사용하면 2GB의 메시지를 송수신 가능하나 백엔드로 S3에 저장하고 메시지 1당 1회 S3 엑세스와 데이터량 비용 발생한다

- Publish / Subscribe 모델 미지원

  Pub/Sub 모델이 없으므로 Pub/Sub 모델을 구현하기 위해서는 Amazon SNS를 함께 사용해야 한다.

- 메시지 타이머

  SQS의 메시지 타이머의 최대값은 15분이다. FIFO 대기열에서는 개별 메시지에 대한 타이머를 지원하지 않는다.

## Amazon SQS vs Amazon MQ

| Amazon SQS                                                                  | Amazon MQ                                           |
| :-------------------------------------------------------------------------- | :-------------------------------------------------- |
| Cloude 기반 태생                                                            | 어플리케이션 마이그레이션에 적합                    |
| 거의 무한의 처리량(표준 대기열의 경우)                                      | 업계 표준 방식, Protocol 지원                       |
| Queue 지원, Pub/Sub 미지원(Topic은 Amazon SNS 활용)                         | 풍부한 기능(Queue, Topic 등)                        |
| Text 메시지만 지원                                                          | Binary 메시지 지원                                  |
| 메시지 사이즈 최대 256KB(Extended Client Library로 2GB까지 가능)            | 메시지 사이즈 제약 없음(브로커당 200GB limit까지)   |
| 요청 수 + 데이터 전송 기반 과금                                             | Instance(초 단위) + 스토리지 + 데이터 전송기반 과금 |
| 메시지가 수신측으로 PUSH 되지 않음. Consumer가 별도로 Pull 하도록 처리 필요 | 메시지가 수신측으로 PUSH                            |
| 메시지 보존기간이 존재 (60초~15일)                                          | 영구 저장 (Persistence mode)                        |

## Amazon MQ 선정 사유

### 높은 호환성

ActiveMQ는 완벽하게 자바와 호환성을 가진다. 예약 시스템이 구현하려는 Spring Boot와 호환성이 좋다. 또한 클라우드에 의존적이지 않기 때문에 추후 AWS를 사용하지 않더라도 다른 클라우드 환경으로 마이그레이션하기 용이하다.

### Publish / Subscribe 모델 사용

현재 개발하려고 하는 예약 시스템에서 발행-구독 모델의 사용이 필요하다. (예약 이벤트 발생 시 파트너시스템, 정산시스템, 알림시스템 발송 등)

### 전달 보증

Amazon MQ로 전송하는 경우 수신 서버가 이용 불가였다가 다시 서비스 가능 상태로 변경 되었을 경우 메시지 전달을 보증해준다. 이는 예약 시스템에서 메시지의 전달 신뢰성이 중요하므로 중요한 요소 중 하나이다.

### Message Push 방식

Amazon MQ는 Push형식으로 메시지를 수신하여 처리한다. 반면 SQS의 경우 Polling 형식으로 데이터를 수신하는 방식을 사용한다. 그리고 수신한 후 Queue의 데이터를 삭제하는 API를 또 다시 호출함으로써 메시지 수신을 완료한다. 이런 방식은 수신자가 메시지 수신 시 관리해야 할 포인트가 증가하므로 개발의 복잡도가 증가할 수 있는 단점이 있다.

### 테스트 용이성

ActiveMQ는 Embedded Broker를 제공해준다. 하지만 SQS의 경우 Embedded환경을 구성할 수 없으므로 개발자마다 개별 인스턴스 또는 가상환경을 제공해 주어야 한다. 그래서 ActiveMQ가 로컬환경에서 자동화된 테스트를 하기 좀더 용이하다고 판단하였다.

### 메시지 지연 시간 단축

Amazon MQ는 10 밀리초 미만의 메시지 지연 시간을 제공한다. (이부분은 Amazon 홈페이지에서 참고) 이는 예약이 발생하는 경우 메시지 지연으로 인한 운영 문제를 고려하지 않을 수 있다.

### 메시지 타이머

Amazon SQS의 경우 최대 15분이라는 타이머 제한이 있다. 하지만 Amazon MQ의 경우 ScheduleSupport 기능을 사용하여 제한 없는 지연 전송, Cron 설정을 통한 특정 일시에 대한 전송을 제공한다.

## Wrap up

지금까지 Amazon MQ와 Amazon SQS를 비교해 보고 Amazon MQ를 선정하게 된 배경에 대해서 알아보았다. 개발 시 발행-구독 모델을 사용해야 한다는 점과 Message 전달 방식이 Amazon MQ를 선택하게된 주된 이유라고 생각한다. SQS도 Amazon SNS를 이용해서 발행-구독 모델을 사용할 수 있으나 이는 인프라 관리부담이 증가할 수 있어서 매력적으로 다가오지 않았다. 이제 Amazon MQ를 이용하여 구현할 수 있는 메시지 전달 방식들을 알아보고 상황에 따라 사용방법을 알아보아야 겠다.

> [https://www.slideshare.net/awskorea/utilizing-amazon-mq-fully-managed-active-mq-service-changsu-lee](https://www.slideshare.net/awskorea/utilizing-amazon-mq-fully-managed-active-mq-service-changsu-lee){: target="\_blank" }
>
> [https://www.slideshare.net/awskorea/aws-aws-devday2018-122965087](https://www.slideshare.net/awskorea/aws-aws-devday2018-122965087){: target="\_blank" }
>
> [https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html){: target="\_blank" }
>
> [https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html](https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html){: target="\_blank" }
