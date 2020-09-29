---
layout: single
title: "쿠버네티스 스터디 4일차"
categories:
  - STUDY
tags:
  - kubernetes
use_math: true
---

## 로컬환경에서 쿠버네티스 서비스에 접근하는 방법

쿠버네티스에 서비스 배포 시 만약 서비스 타입을 Cluster IP로 지정한 경우 로컬환경에서 서비스로 바로 접근이 불가능하다. 쿠버네티스 서비스끼리는 서비스명을 호스트명으로 지정하여 사용하기 때문에 접근할 수 있지만 로컬환경에서는 IP대역이 다르기 때문에 접근이 불가능하다. 만약 로컬에서 쿠버네티스 서비스로 접근이 필요한 경우 아래와 같이 접근할 수 있는 방법이 있다.

### Ingress를 오픈한다.

Ingress는 쉽게 말하면 방화벽 정책과 비슷한데 외부에서 쿠버네티스 네트워크로 들어오는 트래픽에 대한 정책을 설정할 수 있다. 내 로컬 환경에서 쿠버네티스로 접근할 수 있도록 정책을 오픈해버리면 원하는 서비스에 접근할 수 있는데, 이러한 방법은 보안에 취약할 수 있으므로 추천하지 않는 방식이다.

### Proxy를 통해서 접근한다.

로컬환경에 쿠버네티스 프록시를 띄워서 접근하는 방법이 있다. [링크 참고](https://kubernetes.io/ko/docs/tasks/access-application-cluster/access-cluster/#rest-api%EC%97%90-%EC%A7%81%EC%A0%91-%EC%95%A1%EC%84%B8%EC%8A%A4){: target="\_blank" }

해당 방법은 URL이 복잡하고 Proxy룰이 존재해서 하나하나 설정을 해줘야하는 번거로움이 있다.

### 포트포워딩을 사용한다.

매번 접근이 필요할때마다 포워딩을 해줘야 한다는 불편함이 있지만 손쉽고 빠르게 할 수 있으므로 가장 추천하는 방법이다. 포트 포워딩은 콘솔에 커맨드를 입력해서 실행하는 방법이 있고, 렌즈를 이용하는 방법이 있다.

- Console

  ```bash
  kubectl -n {namespace} port-forward svc/{서비스명} {host-port}:{service-port}
  ```

  ex)

  ```bash
  kubectl -n playground port-forward svc/test-service 5000:80
  ```

- Lens

  Network > Services 메뉴를 선택하여 Services 목록을 오픈한 후 원하는 서비스를 선택한다. 그런다음 Connection 섹션에 Ports 부분의 링크를 클릭하면 브라우저가 열리면서 포워딩 된 포트를 알려준다.

  ![lends port forwarding](/assets/images/study/lens-port-forwarding.png)

## skaffold

[skaffold](https://skaffold.dev/){: target="\_blank" }는 쿠버네티스용 CD 툴이다. 아직 사용은 안해봣는데 좋은 툴이라고 추천받아서 기회가 되면 사용해 봐야겠다. 간단한 튜토리얼은 [조대협 선생님의 블로그](https://bcho.tistory.com/1342){: target="\_blank" }를 참고하자.