---
layout: single
title: "Tunneling vs Port Forwarding"
categories:
  - EXPLANATION
tags:
  - tunneling
  - port forwarding
toc: true
toc_sticky: true
excerpt: tunneling과 port forwarding에 대해 알아보자.
---

**Tunneling**과 **Port Forwarding**이라는 말은 비슷한 의미로 혼용해서 쓰는 경우가 많이 있다. 그리고 실제로 검색을 해봐도 동일한 의미인 것 처럼 작성된 글들을 종종 볼 수 있어 두 개념의 의미를 정리해보고 그 차이점을 비교해보고자 한다. ~~두 개념에 대한 설명은 Wikipedia에서 제공되는 정보를 그대로 번역해서 사용하였다.~~

## Tunneling

컴퓨터 네트워크에서 [Tunneling protocol](https://en.wikipedia.org/wiki/Tunneling_protocol){: target="\_blank" }은 다른 트워크에서 온 데이터를 이동할 수 있도록 하는 커뮤니케이션 프로토콜이다.
캡슐화라고 불리는 과정을 통해 공용 네트워크(인터넷 등)를 통해 사설 네트워크 통신을 할 수 있도록 하는 것을 포함한다.

**Tunneling**은 트래픽 데이터를 다른 형태로 다시 포장하는 것을 포함하기 때문에, 아마도 암호화를 표준으로 하여 터널을 통과하는 트래픽의 특성을 숨길 수 있다.

**Tunneling protocol**은 실제로 서비스를 제공하는 패킷을 운반하기 위해 패킷의 데이터 부분(Payload)을 사용하여 작동한다. 터널링은 OSI나 TCP/IP 프로토콜 제품군의 것과 같은 계층화된 프로토콜 모델을 사용하지만, 일반적으로 네트워크가 정상적으로 제공하지 않는 서비스를 운반하기 위해 페이로드(payload)를 사용할 때는 계층화를 위반한다. 일반적으로, 전달 프로토콜은 페이로드 프로토콜보다 계층화된 모델에서 동일하거나 더 높은 수준에서 동작한다.

### Common tunneling protocols

- IP in IP (Protocol 4): IP in IPv4/IPv6
- SIT/IPv6 (Protocol 41): IPv6 in IPv4/IPv6
- GRE (Protocol 47): Generic Routing Encapsulation
- OpenVPN (UDP port 1194): Openvpn
- SSTP (TCP port 443): Secure Socket Tunneling Protocol
- IPSec (Protocol 50 and 51): Internet Protocol Security
- L2TP (Protocol 115): Layer 2 Tunneling Protocol
- VXLAN (UDP port 4789): Virtual Extensible Local Area Network.

## Port forwarding

컴퓨터 네트워킹에서 [Port forwarding](https://en.wikipedia.org/wiki/Port_forwarding){: target="\_blank" } 또는 **port mapping**은 패킷이 라우터나 방화벽과 같은 네트워크 게이트웨이를 통과하는 동안 통신 요청을 한 주소와 포트 번호 조합에서 다른 주소로 다시보내는 NAT(네트워크 주소 변환)의 애플리케이션이다.

이 기법은 통신의 대상 IP 주소와 포트 번호를 내부 호스트로 다시 매핑하여 게이트웨이(외부 네트워크) 반대편의 호스트에서 보호되거나 위장된 (내부) 네트워크에 상주하는 호스트에서 서비스를 제공하는 데 가장 일반적으로 사용된다.

**Port forwarding**에는 아래와 같은 타입이 존재한다.

### Local port forwarding

**Local port forwarding**은 **Port forwarding**의 가장 일반적인 유형이다. 이것은 사용자가 로컬 컴퓨터에서 다른 서버로, 즉 SSH(Secure Shell) 클라이언트와 동일한 컴퓨터에서 실행 중인 다른 클라이언트 응용프로그램에서 데이터를 안전하게 전달하기 위해 사용된다. **Local port forwarding**을 사용하면 특정 웹 페이지를 차단하는 방화벽을 우회할 수 있다.

### Remote port forwarding

이 형태의 포트 포워딩은 SSH(Secure Shell) 연결의 서버측에서 SSH 클라이언트측에 상주하는 서비스에 접속하는 애플리케이션을 가능하게 한다. SSH 외에도, 동일한 일반 목적을 위해 원격 포트 포워딩을 활용하는 독점적인 튜닝 체계가 있다. 즉, **Remote port forwarding**을 통해 사용자는 터널의 서버측, SSH나 다른 쪽에서 터널의 클라이언트측에 위치한 원격 네트워크 서비스로 연결할 수 있다.

### Dynamic port forwarding

**Dynamic port forwarding**(DPF)은 방화벽 핀홀을 사용하여 방화벽 또는 NAT을 통과시키는 온디맨드 방식이다. 목표는 클라이언트가 하나 이상의 대상 서버에 데이터를 전송/수신할 목적으로 중개 역할을 하는 신뢰할 수 있는 서버에 안전하게 연결할 수 있도록 하는 것이다.

## Difference

**Port forwarding**과 **Tunneling**의 가장 큰 차이점은 **Port forwarding**은 한 프로토콜의 [PDU](https://en.wikipedia.org/wiki/Protocol_data_unit){: target="\_blank" }를 다른 프로토콜로 캡슐화 하지 않고 **Tunneling**은 일반적으로 보안상의 이슈로 캡슐화를 한다는 것이다. 그래서 **Port forwarding**은 저사양 네트워크 장비로도 사용이 가능하며 단순히 수신 포트/호스트 조합을 검사하고 다시 써서 패킷을 다른 곳으로 전송하는 일만한다. 반면 **Tunneling**은 패킷의 흐름에 논리를 추가해야 하고 로딩이 무겁고 보다 강력한 시스템이 요구된다.

아래에 추가적으로 구별되는 특징이 더 있다.

### Tunneling

- 암호화된 터널을 사용하여 암호화되지 않은 wifi 네트워크 또는 공용 인터넷과 같은 신뢰할 수 없는 네트워크를 통해 데이터를 안전하게 전송할 수 있다.
- 한 Port의 Tunnel을 사용하여 일반적으로 다른 Port에서 전송되는 데이터를 전송할 수 있으며, 클라이언트와 서버 모두에게 네트워크 보안을 투명하게 우회할 수 있다.
- Tunnul을 사용하여 두 장치 또는 네트워크를 연결할 수 있으며, 다른 전송 네트워크의 기본 프로토콜을 사용할 수 있다. 예를 들어 MPLS를 통해 TCP/IP를 실행할 수 있다.
- ~~최소한 Tunnel을 설정하기 위해서는~~ 패킷 내부의 데이터를 검사해야 한다.
- 상태 저장한다.
- 네트워크 프로토콜 외부에서 제어되며, 한 패킷의 데이터는 다른 패킷의 데이터와 관련되거나 다른 패킷의 데이터에 의존한다.

### Port forwarding

- 일반적으로 허용된 Port에서 트래픽을 수신하고, 대상 주소를 구성된 매핑된 IP 주소로 수정한 다음, 나가는 트래픽을 올바르게 다시 라우팅할 수 있도록 이를 기록하는 NAT 기술의 일부분이다.
- 네트워크 패킷에서 이용 가능한 데이터만 사용하여 수행할 수 있다.
- 규칙과 일치하는 모든 데이터가 전달된다는 것에서 '상태가 없는' 것을 의미한다.

<br>

사내에서 네트워크 구간이 다른 서버에의 서비스를 이용하기 위해 **Port forwarding**을 사용하고 있다. 동료들에게 이 방법을 설명할 때 **Tunneling**이라고도 했다가 **Port forwarding**이라고 하기도 하였다. 이번 글을 적으면서 확실히 동료들에게 "**Port forwarding**, ~~아니 **Local Port forwarding**~~을 쓰세요" 라고 말해야 겠다.

> [https://en.wikipedia.org/wiki/Tunneling_protocol](https://en.wikipedia.org/wiki/Tunneling_protocol){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Port_forwarding](https://en.wikipedia.org/wiki/Port_forwarding){: target="\_blank" }
>
> [https://superuser.com/questions/749919/is-there-a-difference-between-port-forwarding-and-tunneling](https://superuser.com/questions/749919/is-there-a-difference-between-port-forwarding-and-tunneling){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Protocol_data_unit](https://en.wikipedia.org/wiki/Protocol_data_unit){: target="\_blank" }
