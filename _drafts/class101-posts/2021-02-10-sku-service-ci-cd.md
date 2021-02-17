---
layout: single
title: "SKU 서비스 코드 병합 및 배포 방식"
categories:
  - POST
tags:

---

# TL;DR

- SKU 서비스의 코드 병합은 어떻게 하나
- 배포는 어떻게 되나?

# 코드 병합

이전에 [SKU 서비스 제작기](https://www.notion.so/SKU-ff39716d5a294e0f9e6034444e497e3d)에서  얘기한 바와 같이 테스트 커버지리를 충족하지 못하고 코드 컨벤션을 맞추지 못한 코드는 코드 병합을 하지 못하게 설정해 두었다.

로컬 환경에서 테스트를 실행하면 PR 요청 시에 해당 요건을 충족하여 할 수도 있지만 이는 물리적으로 강제할 방법이 없어 Pull Request 시 GIT Action이 동작하게 하여 요구사항을 만족하는 경우에만 병합을 할 수 있도록 설정하였다.

![github-build-check](/assets/images/draft/class101-posts/github-build-check.png)

# 배포

기본 브런치는 `master` 로 하였고 기본 브런치에 코드가 병합이 되면 젠킨스로 Hook을 보내도록 설정하였다.

![github-webhook-setting](/assets/images/draft/class101-posts/github-webhook-setting.png)

젠킨스는 GIT Hub의 Hook을 받아서 `master` 브런치에 대한 이벤트만 설정한 CI와 CD를 실행하도록 하였다.

![jenkins-build-setting](/assets/images/draft/class101-posts/jenkins-build-setting.png)

CI는 단순히 Gradle의 JIB 테스크를 실행하는 것인데, 간단히 말하면 설정한 ECR 경로에 어플리케이션 코드를 업로드 한다고 생각하면 된다.

![jenkins-gradle-setting](/assets/images/draft/class101-posts/jenkins-gradle-setting.png)

```kotlin
jib {
    to {
        image = "$ecrUri/sku-service:$version"
    }

    container {
        creationTime = "USE_CURRENT_TIMESTAMP"
        jvmFlags = mutableListOf(
            "-javaagent:/agents/$apmAgentFileName"
        )
    }
}
```

JIB 명령은 별도 설정이 없으면 테스트 코드를 실행하진 않는다. 설정을 통해서 테스트 빌드를 한 후 이미지를 업로드 할 수 있지만 이미 우리는 코드 병합 시 검증된 코드가 전달되었다고 믿기 때문에 배포 시간을 줄일 겸 별도 테스트 코드를 실행하지 않도록 하였다.

CD는 CI가 끝난 후 다음 스텝에서 바로 실행되는데 어플리케이션 내에서 작성한 배포 스크립트를 단순히 실행하는 방식으로 되어 있다.

![jenkins-script-setting](/assets/images/draft/class101-posts/jenkins-script-setting.png)

배포 스크립트는 헬름 차트 명령어 이며 설정한 네임스페이스의 서비스에 원하는 설정값으로 배포하도록 하였다.

```bash
#!/bin/bash

ENV=$1
VERSION=$2

if [ "$ENV" == "" ] || [ "$VERSION" == "" ]; then
    echo "PARAMETER IS NOT EMPTY: \$1=[$ENV], \$2=[$VERSION]"
    exit
fi

echo "helm upgrade --install -n cell-commerce sku-service-$ENV ./class101-sku-service -f ./class101-sku-service/values-$ENV.yaml --set image.versionTag=$VERSION"
helm upgrade --install -n cell-commerce sku-service-$ENV ./class101-sku-service -f ./class101-sku-service/values-$ENV.yaml --set image.versionTag=$VERSION
```

# 정리

엄밀히 말하면 지금 소개해준 배포 방식은 CI/CD라 하기엔 모자람이 많다. 그냥 편의상 CI/CD라고 이름을 붙였다고 생각하면 좋을 것 같다. 

말하고 싶었던 부분은 앞으로 우리가 코드의 안정성을 높이고 자동화된 배포 설정을 통해서 배포에 대한 리소스를 줄이고 짧은 배포 주기를 가짐으로써 시스템을 점진적으로 발전시키는 전략을 가지면 좋을 것 같다 라는 생각이다.