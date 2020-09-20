---
layout: single
title: "쿠버네티스 스터디 3일차"
categories:
  - STUDY
tags:
  - kubernetes
use_math: true
---

## 사용자 추가

Lens에서 `configuration` -> `configMap` 메뉴를 선택한 후 `aws-auth`를 선택한다.

![lens aws auth](/assets/images/study/lens-aws-auth.png)

`mapUsers` 컴포넌트에 아래와 같이 입력한다.

```yml
- groups:
  - system:{권한}
  userarn: {AWS ARN}
  username: {사용자명}
```

## 쿠버네티스 설정 파일 확인 방법

```
~/.kube/config
```

## 쿠버네티스 config 업데이트

```
aws eks --region {리전} update-kubeconfig --name {네임스페이스 명}
```

ex) `aws eks --region ap-northeast-2 update-kubeconfig --name test`

## Drain

[kubctl drain](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/){: target="\_blank" }은 노드를 지우고자 하는 경우 Pod들을 안전하게 제거하기 위해 사용된다. 

```
kubectl drain $NODENAME
```

## Cordon

노드를 예약 불능 상태로 변경하고자 하는 경우 사용한다.

```
kubectl cordon $NODENAME
```

## 간단히 앱을 띄워보자

### DB 설치

- 헬름차트 pull 받기

```
helm pull bitnami/mariadb
```

- values.yml 파일 수정

간단하게 하려면 다른건 별로 손댈건 없고 서비스 타입과 사용자 설정, DB 설정 정도만 해주면 된다.

```yaml
# ...

service:
  ## Kubernetes service type, ClusterIP and NodePort are supported at present
  type: ClusterIP
  # clusterIp:
  #   master: xx.xx.xx.xx
  #   slave: xx.xx.xx.xx
  port: 3306
  ## Specify the nodePort value for the LoadBalancer and NodePort service types.
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
  ##
  # nodePort:
  #   master: 30001
  #   slave: 30002

# ...

rootUser:
  ## MariaDB admin password
  ## ref: https://github.com/bitnami/bitnami-docker-mariadb#setting-the-root-password-on-first-run
  ##
  password: "비밀번호"

# ...

db:
  ## MariaDB username and password
  ## ref: https://github.com/bitnami/bitnami-docker-mariadb#creating-a-database-user-on-first-run
  ##
  user: "사용자"
  password: "사용자 비밀번호"
  ## Database to create
  ## ref: https://github.com/bitnami/bitnami-docker-mariadb#creating-a-database-on-first-run
  ##
  name: 스키마명
```

- 인스톨

```
helm upgrade --install -n $NAMESPACE $SERVICENAME bitnami/mariadb -f values.yaml
```

ex)
```
helm upgrade --install -n dev maria-dev bitnami/mariadb -f values.yaml
```

### WAS 설치

Spring Boot 웹 앱을 설치해보자

- JIB로 빌드 및 이미지 생성

JIB 사용 사례는 아직 정리하진 않아서 참고했던 [글 링크](https://medium.com/@gaemi/spring-boot-%EA%B3%BC-docker-with-jib-657d32a6b1f0){: target="\_blank" }를 공유한다. 해당 글에서는 docker hub에다가 이미지를 올리는데 나는 ECR에다가 이미지를 올려서 사용했다.

- JIB 사용 시 ECR 로그인 방법

아래 커멘드를 실행하거나

```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $ECR_URL
```

credencial-helper를 사용한다.

1. docker-credential-helper-ecr 설치

```bash
brew install docker-credential-helper-ecr
```

2. 설치 후 docker config 변경

```
{
	..
	"credHelpers": {
		"$ECR_URL": "ecr-login"
	}
	..
}
```

3. awscli v2 설치

```
# MacOS
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
aws --version
```

4. aws 인증 세팅

```
$ aws configure
AWS Access Key ID [None]: {자신의 Access Key ID}
AWS Secret Access Key [None]: {자신의 Secret Access Key}
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

5. 확인

```
$ docker-credential-ecr-login list
{"$ECR_URL":"AWS"}
```

- 인스톨

template 파일은 추후 정리

```
helm upgrade --install -n $NAMESPACE $SERVICE_NAME $TEMPLATE_PATH -f $VALUES_FILE --set image.versionTag=$HASH
```