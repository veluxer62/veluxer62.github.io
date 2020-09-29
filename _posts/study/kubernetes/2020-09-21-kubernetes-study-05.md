---
layout: single
title: "쿠버네티스 스터디 5일차"
categories:
  - STUDY
tags:
  - kubernetes
use_math: true
---

이번 글에서는 쿠버네티스의 Pod들이 AWS의 특정 권한을 가지기위해서 어떻게 하면 되는 지에 대해 배운것을 정리하였다. 보안을 위해서 AWS의 서비스를 특정 롤만 사용가능하도록 제한하는 경우가 많은데, (예를 들어 S3의 조회는 public으로 되어있으나 upload는 특정 권한만 사용가능하도록 한다.) 이런 경우 쿠버네티스 서비스에 롤을 부여해야 하는 경우가 있다. 아래 내용을 보면서 어떻게 하면 되는 지 보자.

## Pod에 AWS Role 추가하기

### OpenID Connect Provider URL 확인

Pod에 Role을 추가하기 위해서는 먼저 롤을 추가하여야 하는데 그러기 위해서 OpenID를 확인하여야 한다. OpenID Connect Provider URL은 EKS 서비스 메뉴에 있다.

EKS 서비스 메뉴로 이동한 후에

![aws-eks-list](/assets/images/study/view-eks-list.png)

원하는 Cluster를 선택하면 상세화면에서 OpenID Connect Provider URL을 확인할 수 있다. (비교를 위해서 따로 복사해두자)

![aws-eks-list](/assets/images/study/select-eks-cluster.png)

### Role 추가

이제 Role을 추가해보자. IAM 서비스 메뉴에서 Role 메뉴를 선택하고 Create Role 버튼을 클릭해서 Role 추가를 진행한다.

![aws-eks-list](/assets/images/study/aws-role-list.png)

Truest Entity를 `Web Identity`로 선택한 후 Identity provider와 Audience를 선택한다. Identity provider는 이전에 확인했던 OpenID Connect Provider URL과 동일한지 꼭 확인한다. 그런다음 원하는 Permission을 추가한 후 Role 추가작업을 마무리한다.

### Role에 Sub condition 추가

방금 추가한 Role을 선택한 후 Trust relationships 탭을 선택하고 Edit relationships 버튼을 클릭하여 Sub condition을 추가해 보자.

Sub Condition을 추가하는 방법은 Trust Relationship yaml 설정 파일의 `Statement.Condition.StringEuals`에 아래와 같은 규칙을 가진 값을 추가하면 된다.

1개의 서비스를 추가하는 경우

```yml
"{provider url}:sub": "system:serviceaccount:{namespace}:{cluster name}"
```

N개의 서비스를 추가하는 경우

```yml
"{provider url}:sub": [
  "system:serviceaccount:{namespace}:{cluster name}",
  "system:serviceaccount:{namespace}:{cluster name}",
  "system:serviceaccount:{namespace}:{cluster name}"
]
```

ex)

```yml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::1234567:oidc-provider/oidc.eks.ap-northeast-2.amazonaws.com/id/QWERTYTUIOPASDFFGGHJKLL"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.ap-northeast-2.amazonaws.com/id/QWERTYTUIOPASDFFGGHJKLL:aud": "sts.amazonaws.com",
          "oidc.eks.ap-northeast-2.amazonaws.com/id/QWERTYTUIOPASDFFGGHJKLL:sub": [
            "system:serviceaccount:playground:a-server",
            "system:serviceaccount:playground:b-server",
            "system:serviceaccount:playground:c-server",
            "system:serviceaccount:kube-system:d-server"
          ]
        }
      }
    }
  ]
}
```

## Role 적용

### kubectl

Pod에 위에서 추가한 롤이 적용되도록 하기 위해서는 ServiceAccount에 role-arn을 추가해주어야 한다. kubectl 명령어를 실행해서 적용이 정상적으로 되었는지 확인할 수 있다. kubectl로 하는 방법은 단순히 확인용도로만 하자. 운영 환경에 대한 배포는 Helm chart를 사용하는 것이 좋다.

```bash
kubectl run -n {namespace} awscli -it --rm --image=amazon/aws-cli --serviceaccount='{servicea account name}' --command -- sh
```

### Helm chart

차트 템플릿 파일에 Service Account의 Annotation 설정하는 부분을 찾아서 Role의 Arn을 추가해주자.

```yml
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: 'arn:aws:iam::123456789:role/a-server'
```

## 추가이슈

위 설정대로 하였는데 롤이 정상적으로 적용되지 않았다. 확인해보니 아래 링크와 같이 Webhook 설정을 해줘야 했다. 한번 설정하고 나면 다음부턴 위에서 처럼 롤만 추가하고 어노테이션만 추가하면 된다.

[Amazon EKS Pod Identity Webhook](https://github.com/aws/amazon-eks-pod-identity-webhook){: target="\_blank" }

[데브시스터즈 블로그](https://tech.devsisters.com/posts/pod-iam-role){: target="\_blank" }에도 잘 설명이 되어 있다.