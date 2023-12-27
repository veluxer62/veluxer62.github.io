---
layout: single
title: "2020년 10월 3주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-third-week-of-october-weekly-retrospective/
---

월요일 부터 대사때문에 새벽에 집에 갓더니 일주일 내내 컨디션이 좋지 않았던 한주였다. 거기다 이런저런 약속으로 늦게까지 술을 마시고 집에 가니 더더욱 상태가 나빳던 것 같다. 컨디션이 좋지 않으니 일에 집중도가 떨어져서 효율적으로 일을 못하고 정작 중요한 일을 마무리 못하게 되는 상황이 발생했다. 해야할 일이 많은 만큼 우선순위를 잘 정하고 차근차근 일을 해 나가야 겠다.

아래 내용은 이번 한주 동안 내갓 새롭게 배우거나 겪었던 이슈를 정리한 것이다.

## kubectl jsonpath 옵션

kubectl에서 jsonpath 옵션을 사용하면 커멘드 결과에서 원하는 항목들을 보기 좋게 출력할 수 있다.

예시) kubctl pod의 이름과 라벨을 확인하는 명령어

```bash
kubectl get pods -o=jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.metadata.labels}{'\n'}{end}"
```

## kubectl history & undo

- history

디플리먼트에서 이력을 확인하는 명령어는 아래와 같다.

```bash
kubectl rollout history deploy {디플로이먼트 이름} [--revision={버전번호}]
```

- undo

디플로이먼트 배포 버전을 이전으로 되돌리는 명령어는 아래와 같다.

```bash
kubectl rollout undo deploy {디플로이먼트 이름}
```

## kubectl pause & resume & restart

- pause

디플로이먼트를 정지하는 명령어이다.

```bash
kubectl rollout pause deployment/{디플로이먼트이름}
```

- resume

정지되어 있는 디플로이먼트를 재개하는 명령어이다.

```bash
kubectl rollout resume deploy/{디플로이먼트이름}
```

- restart

디플로이먼트를 재시작 하는 명령어이다.

```bash
kubectl rollout restart deployment/{디플로이먼트이름}
```

## 쿠버네티스 배포 상태 체크 스크립트

젠킨스에서 쿠버네티스를 update명령어를 통해 배포하는 경우 명령어 실행 후 바로 success 메시지가 반환된다. 하지만 우리는 서비스가 정상적으로 배포되었는지 확인할 필요가 있고 파드가 모두 성공적으로 업데이트 되었는지 체크할 필요가 있는데 이럴 때 사용하면 좋은 스크립트를 찾아서 적어본다.

```bash
#!/bin/sh

DEPLOY_NAME=sample-service
NAMESPACE=sample-namesapce

CMD="kubectl rollout status deployment $DEPLOY_NAME -n $NAMESPACE"

if ! $CMD; then
    echo "Rolling back deployment!" >&2
    kubectl rollout undo deployment $DEPLOY_NAME -n $NAMESPACE
    exit 1
fi
```

간단히 설명하면 디플로이먼트의 상태를 체크한 후 배포가 완료되면 정상종료하고 그렇지 않고 상태 체크 시 오류가 발생하면 이전 버전으로 롤백하는 명령어이다.

## agenda process 주기 설정

Delay Queue 기능을 사용하기 위해 node 서버에서 agenda 라는 라이브러리를 사용하였다. agenda는 쉽게 말해 MongoDB를 이용하여 스케줄잡, 딜레이큐 등등을 제공해주는 라이브러리 인데, 딜레이 큐를 사용하기 위해서 적용 후 사용하고 있었다. 2~3개월 사용하면서 큰 문제가 없다가 최근에 이슈가 발생하였는데, 내가 DB에 대량의 큐를 넣어버리는 실수를 범했고 agenda 프로세서가 해당 큐 데이터를 모두 읽어 들이면서 DB에 부하를 주게 되었고 서비스의 장애로까지 이어지게 되었다.

해당 장애는 DB의 불필요한 프로세스를 죽이고 기간이 만료된 Queue들을 제거하는 작업을 해주어서 해결하게 되었는데, 근본적으로는 프로세스의 실행 주기를 변경할 필요가 있었다. (더 근본적으로는 DB가 아닌 다른 queue를 사용해야 하는데 이부분은 추후 작업할 예정이다.)

Agenda에는 프로세스의 실행주기 변경과 동시 실행되는 프로세스 최대 개수, 기본 개수, 그리고 잠금 제한등등을 통해 DB의 부하를 조절할 수 있다.
자세한 내용은 [Agenda 공식 문서](https://github.com/agenda/agenda#processeveryinterval){: target="\_blank" }를 참고하자.

## mermaid

회사 동료가 소개해준 다이어그램 도구이다. Gitlab에서 공식적으로 사용하면서 유명해진 도구인데 PlantUML처럼 간단한 텍스트를 입력하면 다이어그램을 시각화 해준다. PlantUML 처럼 다이어그램을 보기 위한 Viewer Server를 따로 만들어주지 않고 크롬브라우저의 플러그인 설치만으로 손쉽게 사용할 수 있어 앞으로 자주 활용해야 겠다.

공식 사이트: [https://mermaid-js.github.io/mermaid](https://mermaid-js.github.io/mermaid){: target="\_blank" }

## detect

이것도 회사 동료가 소개해준 코드 분석 도구인데, ktlint를 wrapping 버전이라 생각하면 될거 같다. 좋은점은 [Intellij Plugin](https://github.com/detekt/detekt-intellij-plugin){: target="\_blank" }이 있다는 것인데 한번 적용해서 써봐야겠다.

공식 사이트: [https://github.com/detekt/detekt](https://github.com/detekt/detekt){: target="\_blank" }

## retrofit

안드로이드 진영에서 많이 쓰고 있다는 Retrofit 이다. RestTemplate을 사용하다보면 정말 불편한게 많은데 나는 그래서 주로 WebFlux에서 사용하는 WebClient를 주로 사용하였다. Retrofit의 장점은 인터페이스로 간단히 API 명세를 정의하면 손쉽게 Client에서 사용할 수 있다는 것이다. Non-blocking 지원이 되지 않는게 조금 아쉽지만, 굳이 Non-blocking을 사용하지 않는 곳에서는 RestTemplate를 사용하는 것보다 훨씬 좋아 보인다.

공식 페이지: [https://square.github.io/retrofit/](https://square.github.io/retrofit/){: target="\_blank" }

## eureka

[Spring Cloud](https://spring.io/projects/spring-cloud-netflix){: target="\_blank" } 에서 제공하는 기능중에 eureka라는 기능이 있다는 것을 알게 되었다. 부하분산을 위한 기능으로 보이는데 쿠버네티스로 대체가 가능한 기능으로 보인다.

