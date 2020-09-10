---
layout: single
title: "쿠버네티스 스터디 2일차"
categories:
  - STUDY
tags:
  - kubernetes
use_math: true
---

## Node를 생성해보자

이번시간에는 간단히 쿠버네티스에 Node를 생성해보는 것을 목표로 공부하였다.

## Managed Node

관리되는 노드 그룹은 AWS에서 관리 할 수 있는 노드 그룹이라고 생각하면된다. 관리되는 노드 그룹과 관리되지 않는 노드 그룹의 두드러진 차이점은 관리되는 노드는 [Taint](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/taint-and-toleration/)를 제공하지 않는다는 점이다. 그리고 스팟 인스턴스에서는 관리되는 노드를 생성할 수 없다.

## Unmanaged Node

관리되지 않는 노드 그룹은 AWS에 의해 관리되지 않는 노드 그룹이다. Taint를 설정할 수 있다.

## Kubectl 설치

kubectl은 쿠버네티스 CLI 명령어 도구이다.

```
$ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.15.10/2020-02-22/bin/darwin/amd64/kubectl

$ chmod +x ./kubectl

$ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

$ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bash_profile

$ echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)" >> ~/.zshrc

$ source ~/.zshrc
```

또는

Brew를 통해 설치

```
brew install kubectl
```

### kubectl 치트시트

https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/

## Lens 설치

kubernetes UI 도구이다. 내부적으로 kubectl을 사용하는것처럼 보인다.

설치방법은 아래 링크 참고하자

https://github.com/lensapp/lens

## Node 생성방법

```
$ kubectl create -f FILENAME
```

생성하는 파일의 스키마는 아래 링크 참고

https://eksctl.io/usage/schema/


## 헬름 설치

아래 링크를 통해 헬름 CLI를 설치한다.

https://helm.sh/ko/docs/intro/install/

## Chart repository 추가

```
$ helm repo add {repo-name} {repo-uri}
```

비트나미 추가 예제

```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

## Maria DB Node 생성해보기

### 헬름 values 받기

```
helm pull bitnami/mariadb
```

### values 값 변경

필요한 설정값들을 변경해 주자

https://github.com/bitnami/charts/tree/master/bitnami/mariadb 참고

### 인스톨

```
helm install -n playground mariadb ./mariadb -f ./mariadb/values.yaml
```

참고: namespace를 설정하지 않으면 `default`로 추가된다.

### 업데이트

```
helm upgrade -n playground mariadb ./mariadb -f ./mariadb/values.yaml
```

만약 업데이트 하려고 할때 설치되어 있지 않는 경우 설치하도록 하려면 아래와같이 `--install` 옵션을 주면된다.

```
helm upgrade --install -n playground mariadb ./mariadb -f ./mariadb/values.yaml
```


## 다음시간에는...

오늘은 간단한 MariaDB를 설치해보면서 Node를 생성하는 방법을 공부했다.
다음에는 Spring Boot앱을 띄워보자.

> [https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/managed-node-groups.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/managed-node-groups.html){: target="\_blank" }
>
> [https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/){: target="\_blank" }