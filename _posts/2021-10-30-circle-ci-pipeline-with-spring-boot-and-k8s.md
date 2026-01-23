---
layout: single
title: "Circle CI로 쿠버네티스에 Spring Boot 웹 어플리케이션 배포하기"
categories:
  - TUTORIALS
tags:
  - Circle CI
  - Spring Boot
  - gradle
  - JIB
  - Kubernetes
  - Helm
  - AWS
  - ECR
  - EKS

toc: true
toc_sticky: true
excerpt: 자동화된 배포는 어쩌면 현재 시점에서는 당연하게 받아 들이는 행위라 생각된다. 손으로 하는(자동화되지 않은) 배포는 실수를 유발할 위험성을 가지고 있으며 자동화된 배포에 비해 느리고 번거롭다. 이러한 장점으로 인해 많은 회사들이 자동화된 배포 파이프라인을 가지고 있으며 Jenkins, Github Action, Circle CI, Azure Devops 등 많은 도구를 이용하여 배포 과정을 수행하고 있다. 이글은 그중에 Circle CI를 이용하여 Spring Boot로 만들어진 웹 어플리케이션을 쿠버네티스환경에 배포하기 위한 파이프라인을 만들어보는 과정을 적어보려고 한다. 파이프라인을 작성하는 것에 집중하여 적기위해서 Spring Boot, Gradle, AWS, Kubernetes, Helm, Github, Slack에 대한 기본적인 이해도는 가지고 있다는 것을 전제로 한다. 그래서 Spring Boot로 만들어진 웹 어플리케이션과 Helm Chart, AWS의 Kubernetes 서비스인 EKS가 준비되어 있다고 가정하고 글을 이어가려고 한다. 만약 사전 지식이 부족하다면 아래 링크들을 참고해보길 바란다.
---

## Intro

자동화된 배포는 어쩌면 현재 시점에서는 당연하게 받아 들이는 행위라 생각된다. 손으로 하는(자동화되지 않은) 배포는 실수를 유발할 위험성을 가지고 있으며 자동화된 배포에 비해 느리고 번거롭다. 이러한 장점으로 인해 많은 회사들이 자동화된 배포 파이프라인을 가지고 있으며 [Jenkins](https://www.jenkins.io/){: target="\_blank" }, [Github Action](https://docs.github.com/en/actions){: target="\_blank" }, [Circle CI](https://circleci.com/){: target="\_blank" }, [Azure Devops](https://azure.microsoft.com/en-us/services/devops/){: target="\_blank" } 등 많은 도구를 이용하여 배포 과정을 수행하고 있다.

이글은 그중에 Circle CI를 이용하여 Spring Boot로 만들어진 웹 어플리케이션을 쿠버네티스환경에 배포하기 위한 파이프라인을 만들어보는 과정을 적어보려고 한다. 파이프라인을 작성하는 것에 집중하여 적기위해서 [Spring Boot](https://spring.io/projects/spring-boot){: target="\_blank" }, [Gradle](https://gradle.org/){: target="\_blank" }, [AWS](https://en.wikipedia.org/wiki/Amazon_Web_Services){: target="\_blank" }, [Kubernetes](https://en.wikipedia.org/wiki/Kubernetes){: target="\_blank" }, [Helm](https://helm.sh/){: target="\_blank" }, [Github](https://en.wikipedia.org/wiki/GitHub){: target="\_blank" }, [Slack](<https://en.wikipedia.org/wiki/Slack_(software)>){: target="\_blank" }에 대한 기본적인 이해도는 가지고 있다는 것을 전제로 한다. 그래서 Spring Boot로 만들어진 웹 어플리케이션과 Helm Chart, AWS의 Kubernetes 서비스인 EKS가 준비되어 있다고 가정하고 글을 이어가려고 한다. 만약 사전 지식이 부족하다면 아래 링크들을 참고해보길 바란다.

- [Getting started with Spring Boot](https://spring.io/guides/gs/spring-boot/){: target="\_blank" }
- [Getting started with Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html){: target="\_blank" }
- [Quick start helm](https://helm.sh/docs/intro/quickstart/){: target="\_blank" }
- [Getting started with Github](https://docs.github.com/en/get-started){: target="\_blank" }

## 들어가기 전에

파이프라인을 만들기전에 준비되어야 할 것들을 다시한번 정리하고 본격적으로 글을 이어나가려고 한다.

- Circle CI

  계정 및 프로젝트 설정이 되어 있어야 한다. 그리고 [Github과 통합환경 설정](https://circleci.com/docs/2.0/gh-bb-integration/){: target="\_blank" }도 함께 되어있어야 한다.

- Github

  계정 및 저장소 설정이 되어 있어야 한다.

- 어플리케이션

  Spring Boot로 만들어진 웹 어플리케이션이 필요하다. 배포 후 간단히 테스트 해볼 수 있는 정도의 어플리케이션이면 된다. 이 글에서는 Kotlin DSL Gradle을 사용했다.

- Helm Chart

  배포 시 사용할 Helm Chart가 준비되어 있어야 한다. 차트 파일들 폴더 위치는 `{root}/helm-chart`로 하였다.

- AWS

  AWS 서비스들을 이용하기에 계정 등록이 되어 있어야한다.

- ECR

  Artifact 저장소로 ECR를 사용할 것이기 때문에 저장소를 만들어두어야 한다.

- EKS

  배포 환경으로 EKS를 사용할 것이기 때문에 Cluster를 생성해 두어야 한다.

## 배포를 위한 절차

배포를 위한 절차를 보면 아래 다이어그램과 같다.

![diagram](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/pipeline-diagram.png)

- 빌드

  작성된 코드가 원격 저장소에 push되면 gradle build 스크립트가 실행 된다. (Pull Request도 작업 브랜치에 push를 통해 이루어지므로 이에 포함된다.) 만약 빌드가 실패하면 파이프라인은 그대로 종료되고 성공한다면 다음 과정으로 넘어간다.

- 브랜치 체크

  정의된 브랜치 외에 다음 파이프라인을 실행하지 않도록 브랜치 정책을 정하였다. 작업 브랜치에서 코드를 작성한 후 Pull Request를 통해 병합요청을 하게 되는데 작업 브랜치는 배포할 브랜치가 아니므로 빌드 파이프라인 이후에는 실행하지 않아야 하기 때문이다. 그래서 정의된 배포 브랜치가 아닌 경우 파이프라인은 그대로 종료되고 배포 브랜치라면 다음 과정으로 넘어간다.

- Artifact 업로드

  앱을 실행하기 위한 빌드(또는 컴파일)된 이미지를 [Artifact](<https://en.wikipedia.org/wiki/Artifact_(software_development)>){: target="\_blank" }라고 한다. 소스코드가 원격 저장소에 push될 때마다 새로운 버전의 Artifact가 생성될 것이고 해당 Artifact는 배포에 사용될 것이기 때문에 원격 저장소에 업로드한다.

- 배포 준비 알림

  배포를 위한 준비가 될때까지 매번 담당자는 파이프라인을 바라보고 있을 순 없다. 그래서 배포할 준비가 완료되었다면 배포가 준비되었다고 특정 채널(메일, 슬렉 등)에 알림을 주어 배포 승인을 할 수 있도록 한다.

- 배포 승인

  자동화된 배포라고 하더라도 담당자의 승인 없이 무조건 배포해 버리는 것은 위험하다. 그래서 배포 승인 절차가 필요하다. 배포 승인 담당자는 개발자가 될 수도 있지만 배포에 대한 책임과 권한을 가진 프로젝트 관리자, QA Engineer 등이 될 수도 있다.

- 배포

  업로드된 Artifact를 가져와서 정의된 환경(여기선 EKS)에 어플리케이션을 배포한다.

## Circle CI 용어

Circle CI의 Pipeline을 작성할때 언급될 용어들을 간단하게 정의를 설명하고 진행하면 이해가 좀더 쉬울 것 같아 적어본다. 좀더 자세히 알고 싶으면 [Circle CI Docs](https://circleci.com/docs/2.0/glossary/){: target="\_blank" }를 참고하자

- Orbs

  설정이나 공유자원의 재사용성을 단순화 하기 위한 설정들의 묶음을 말한다.

- Steps

  실행 명령의 모음을 말한다.

- Jobs

  `Step`들의 모음을 말한다.

- Workflow

  작업의 실행 순서와 규칙을 정의한 `Job`의 모음을 말한다.

- Context

  프로젝트들간의 환경변수를 공유하는 메커니즘을 제공한다.

- Environment Variables

  환경변수는 프로젝트에서 사용될 사용자 데이터 정보를 저장한다.

- Executor

  작업을 실행하기 위한 기반 환경을 정의한다.

## 파이프라인 작성

이제 본격적으로 파이프라인을 작성해보자. 먼저 빌드부터 작성하자.

### 빌드

빌드는 웹 어플리케이션으로 실행하기 위해 작성된 코드를 독립적인 Artifact로 변환하는 과정을 말한다. Gradle에서는 `build`명령어를 통해서 실행이 가능하다. 기본적으로 `test` 단계를 포함하고 있기 때문에 테스트 코드를 작성하였다면 코드가 어플리케이션을 정상적으로 실행하기 위한 조건을 충족하는지도 함께 검증할 수 있다.

여기서 `Pull Request` 시에는 `test`만 실행해도 되지 않는지에 대한 질문이 있을 수 있다. Spring Boot로 작성된 어플리케이션과 관련된 이야기이긴 하지만 Gradle의 `test`는 성공하지만 `build`에는 실패하는 경우가 발생할 수 있다. 실제로 어플리케이션을 배포하기 위해서는 `build`된 Artifact가 필요하기 때문에 `test`를 포함하고 있는 `build`명령어를 사용하는 것이다.

파이프라인을 작성해보자. Root 폴더에서 `.circleci` 폴더를 생성하고 `config.yml`파일을 생성한다. 그리고 아래와 같이 적어주자.

```yml
version: 2.1

jobs:
  checkout-and-build:
    docker:
      - image: cimg/openjdk:16.0.2
    steps:
      - checkout
      - run:
          command: ./gradlew build
```

Gradle은 JDK가 있어야 하니 기반환경으로 docker에 openjdk가 설치된 이미지를 가져오도록 하였다. 여기서는 Java 16를 사용하였다. 그런다음 예약된 명령어인 `checkout`을 사용하여 소스파일을 가져온 후 build 명령어를 실행하도록 하였다.

커밋 후 리모트 브랜치에 push하면 아래와 같이 Circle CI의 workflow가 실행되는 것을 볼 수 있다.

![build-pipeline-result](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/build-pipeline-result.png)

하지만 현재 실행되는 리소스의 사양이 `Large`(4 CPU, 8 GB RAM)인데 필요한 것에 비해 너무 크지 않나 생각된다. Gradle이 실행될 때 필요한 리소스를 생각해서 `Medium`정도로 낮추면 좋을 것 같다. [resource_class](https://circleci.com/docs/2.0/configuration-reference/#resourceclass){: target="\_blank" }를 추가해 주자.

```yml
version: 2.1

jobs:
  checkout-and-build:
    docker:
      - image: cimg/openjdk:16.0.2
    resource_class: medium
    steps:
      - checkout
      - run:
          command: ./gradlew build
```

![build-pipeline-result-medium](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/build-pipeline-result-medium.png)

그런데 왠지 Circle CI가 제공해주는 Gradle Orb가 있을거 같다. 찾아보니 [Gradle Orb](https://circleci.com/developer/orbs/orb/circleci/gradle){: target="\_blank" }가 있다. Orb를 사용하면 코드의 재사용성이 높아지므로 이를 사용하도록 변경해보자.

```yml
version: 2.1

orbs:
  gradle: circleci/gradle@2.2.0

workflows:
  checkout-and-build:
    jobs:
      - gradle/run
```

Gradle Orb의 `run` job은 기본적으로 command를 `build`로 실행해 준다. 그래서 별다른 매개변수가 없어도 `build`를 실행해 준다.
하지만 이렇게 실행하면 기본적으로 Java 13 이미지가 기본적으로 설정되고 좀전에 변경하였던 가상머신의 사양이 다시 `Large`로 되돌아 갔다.

![build-pipeline-gradle-run](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/build-pipeline-gradle-run.png)

그래서 원하는 환경에서 실행되도록 `executors`를 설정해주어서 다시 Java 16과 가상환경 크기를 다시 `Medium`으로 변경해주자.

```yml
version: 2.1

orbs:
  gradle: circleci/gradle@2.2.0

executors:
  my-executor:
    docker:
      - image: cimg/openjdk:16.0.2
    resource_class: medium

workflows:
  checkout-and-build:
    jobs:
      - gradle/run:
          executor: my-executor
```

![build-pipeline-gradle-run-with-executor](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/build-pipeline-gradle-run-with-executor.png)

이정도면 어느정도 충분한것 같다. 다음 파이프라인으로 넘어가자.

### Artifact 업로드

Spring Boot 어플리케이션은 Gradle의 [JIB](https://github.com/GoogleContainerTools/jib){: target="\_blank" }를 통해 원격 저장소에 빌드된 이미지를 손쉽게 업로드 할 수 있다. 이미지 업로드는 AWS의 ECR에 하기로 하였다.

#### JIB 설정

먼저 Gradle에서 JIB설정을 해보자.

```kotlin
plugins {
    // 생략...
    id("com.google.cloud.tools.jib") version "3.1.4"
}

// 생략...

fun getBuildVersion(): String {
    val stdout = ByteArrayOutputStream()
    exec {
        commandLine = listOf("git", "rev-parse", "--short", "HEAD")
        standardOutput = stdout
    }
    return stdout.toString().trim()
}

jib {
    from {
        image = "adoptopenjdk/openjdk16"
    }
    to {
        image = "000000000000.dkr.ecr.ap-northeast-2.amazonaws.com/test-service"
        tags = setOf(getBuildVersion())
    }
}
```

JIB 플러그인을 추가해주고 업로드할 이미지 경로를 지정해주면 된다. 여기서 눈여겨 볼 부분은 `tags`이다. 만약 `tags`를 설정하지 않으면 기본적으로 이미지는 업로드할 때마다 `latest`로 업로드가 된다. 매번 마지막 버전으로 배포를 하려면 `latest`로 해도 상관없다. 하지만 운영을 하다보면 특정 버전을 배포해야 하는 경우가 생긴다. 그런경우에는 배포시 버전을 특정할 수 있으면 좋을 것 같다.

GIT을 이용한다면 Commit hash가 이 요구사항에 적합하다.(사람이 하는 버전관리는 불편할 뿐 아니라 실수할 위험성이 있다.) GIT 명령어 중에 HEAD의 짧은 해시값을 가져올 수 있는 명령어가 있는데 이를 이용하면 좋을거 같아서 적용하였다.

#### Job 추가

JIB 설정을 마쳤으니 Pipeline에서 Gradle JIB를 실행하는 Job을 추가해 주자. 이전에 `build`시에 추가해두었던 Gradle orb를 이용하면 될것 같다. 동일한 Job이름이 헷갈릴 수 있으므로 `name`설정을 해줘서 쉽게 구분할 수 있도록 해주자.

```yml
version: 2.1

orbs:
  gradle: circleci/gradle@2.2.0

executors:
  my-executor:
    docker:
      - image: cimg/openjdk:16.0.2
    resource_class: medium

workflows:
  build-and-deploy:
    jobs:
      - gradle/run:
          name: build
          executor: my-executor
      - gradle/run:
          name: push
          executor: my-executor
          command: jib
```

하지만 위 설정은 세가지의 문제가 있다.

- Artifact를 업로드하는 Job은 배포 브랜치에서만 동작해야 하지만 모든 브랜치에서 실행된다.

  앞서 보여준 다이어그램에서는 특정 브랜치에서만 Artifact 업로드 Job이 실행되도록 적어 두었는데 그렇지 않은 것이다.

  ![running-build-and-push](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/running-build-and-push.png)

  Circle CI는 [filter](https://circleci.com/docs/2.0/configuration-reference/#filters){: target="\_blank" }라는 기능을 통해서 특정 브랜치에서 실행되도록 할 수 있다. `filter`를 적용해보자. `main`, `dev`, `qa`브랜치에서만 해당 Job이 실행되도록 하였다.

  ```yml
  version: 2.1

  orbs:
    gradle: circleci/gradle@2.2.0

  executors:
    my-executor:
      docker:
        - image: cimg/openjdk:16.0.2
      resource_class: medium

  workflows:
    build-and-deploy:
      jobs:
        - gradle/run:
            name: build
            executor: my-executor
        - gradle/run:
            name: push
            executor: my-executor
            command: jib
            filters:
              branches:
                only:
                  - main
                  - dev
                  - qa
  ```

  이제 다시 파이프라인을 실행해보면 `test`브랜치에서는 더이상 Artifact 업로드 Job이 실행되지 않는것을 볼 수 있다.

  ![filter-apply-result](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/filter-apply-result.png)

- 빌드 Job과 Artifact 업로드 Job이 동시에 실행된다.

  `build`를 통해 어플리케이션을 배포해도 되는지 체크한 후 Artifact를 원격저장소에 업로드하여야 하는데 현재는 `build`와 `push`가 동시에 실행되기 때문에 불완전한 어플리케이션이 업로드될 수 있다. Job의 순서를 보장하기 위해서 [requires](https://circleci.com/docs/2.0/configuration-reference/#requires){: target="\_blank" }를 사용하면 된다.

  ```yml
  version: 2.1

  orbs:
    gradle: circleci/gradle@2.2.0

  executors:
    my-executor:
      docker:
        - image: cimg/openjdk:16.0.2
      resource_class: medium

  workflows:
    build-and-deploy:
      jobs:
        - gradle/run:
            name: build
            executor: my-executor
        - gradle/run:
            name: push
            executor: my-executor
            command: jib
            filters:
              branches:
                only:
                  - main
                  - dev
                  - qa
            requires:
              - build
  ```

  아래 이미지를 보면 Artifact 업로드 Job이 실행되지 않고 빌드 Job이 끝나기를 기다리는 것을 볼 수 있다.

  ![running-build-and-push-sequentially](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/running-build-and-push-sequentially.png)

- Artifact 업로드 Job이 실패한다.

  workflow 실행 결과를 보면 Artifact 업로드 Job이 실패한 것을 볼 수 있다.

  ![push-pipeline-failed](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/push-pipeline-failed.png)

  로그를 보니 ECR에 이미지를 업로드 하려고 할때 권한이 없어서 발생하는 것으로 보인다.

  ```
  * What went wrong:
  Execution failed for task ':jib'.
  > com.google.cloud.tools.jib.plugins.common.BuildStepsExecutionException: Build image failed, perhaps you should make sure your credentials for '000000000000.dkr.ecr.ap-northeast-2.amazonaws.com/test-service' are set up correctly. See https://github.com/GoogleContainerTools/jib/blob/master/docs/faq.md#what-should-i-do-when-the-registry-responds-with-unauthorized for help
  ```

  [Circle CI의 ECR Orb](https://circleci.com/developer/orbs/orb/circleci/aws-ecr){: target="\_blank" }를 추가해주고 로그인 step을 추가해 주자. [ecr-login](https://circleci.com/developer/orbs/orb/circleci/aws-ecr#commands-ecr-login){: target="\_blank" }을 통해 credentials을 받으려면 Project의 Environment Variables를 등록해야 한다. (Key 값은 default 값으로 하였다.)

  ![ecr-project-environment-variable](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/ecr-project-environment-variable.png)

  ```yml
  version: 2.1

  orbs:
    gradle: circleci/gradle@2.2.0
    aws-ecr: circleci/aws-ecr@7.2.0

  executors:
    my-executor:
      docker:
        - image: cimg/openjdk:16.0.2
      resource_class: medium

  workflows:
    build-and-deploy:
      jobs:
        - gradle/run:
            name: build
            executor: my-executor
        - push:
            name: push
            executor: my-executor
            steps:
              - aws-ecr/ecr-login
              - gradle/run:
                  command: jib
            filters:
              branches:
                only:
                  - main
                  - dev
                  - qa
            requires:
              - build
  ```

  하지만 위 스크립트는 제대로 실행되지 않는다. workflow에서 `push` Job이 잘못 정의되었다는 메시지를 아래와 같이 확인할 수 있다.

  ![push-workflow-failed](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/push-workflow-failed.png)

  찾아보니 `gradle/run`은 `steps`에서 정의할 수 없는듯 하였다. 그래서 아래와 같이 `run` 명령어를 통해 `jib`를 직접 실행하도록 변경하였다. `steps`도 workflow에서 정의하면 정상적으로 실행되지 않아 `jobs`를 별도로 추출하였다.

  ```yml
  version: 2.1

  orbs:
    gradle: circleci/gradle@2.2.0
    aws-ecr: circleci/aws-ecr@7.2.0

  executors:
    my-executor:
      docker:
        - image: cimg/openjdk:16.0.2
      resource_class: medium

  jobs:
    push:
      executor: my-executor
      steps:
        - checkout
        - aws-ecr/ecr-login
        - run:
            command: ./gradlew jib

  workflows:
    build-and-deploy:
      jobs:
        - gradle/run:
            name: build
            executor: my-executor
        - push:
            filters:
              branches:
                only:
                  - main
                  - dev
                  - qa
            requires:
              - build
  ```

  위에서 적힌대로 실행해보니 workflow가 모두 성공적으로 종료된것을 확인할 수 있었다.

  ![push-pipeline-success](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/push-pipeline-success.png)

  그리고 ECR에 이미지도 업로드 된것을 확인하였다.

  ![check-ecr-image-uploaded](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/check-ecr-image-uploaded.png)

### 배포 알림

Artifact가 업로드되고나면 배포준비는 완료되었다. 그래서 배포 담당자에게 배포준비가 완료되었음을 알리는 Job을 추가하자. 이 글에서는 Slack으로 배포 알림을 보내려고 한다.

[Circle CI의 Slack Orb](https://circleci.com/developer/orbs/orb/circleci/slack){: target="\_blank" }를 통해서 손쉽게 슬랙 전송 기능을 사용할 수 있다. 많은 알림방식 중에 [on-hold](https://circleci.com/developer/orbs/orb/circleci/slack#jobs-on-hold){: target="\_blank" } 알림을 사용하였다.

```yml
version: 2.1

orbs:
  gradle: circleci/gradle@2.2.0
  aws-ecr: circleci/aws-ecr@7.2.0
  slack: circleci/slack@4.4.4

executors:
  my-executor:
    docker:
      - image: cimg/openjdk:16.0.2
    resource_class: medium

jobs:
  push:
    executor: my-executor
    steps:
      - checkout
      - aws-ecr/ecr-login
      - run:
          command: ./gradlew jib

workflows:
  build-and-deploy:
    jobs:
      - gradle/run:
          name: build
          executor: my-executor
      - push:
          filters:
            branches:
              only:
                - main
                - dev
                - qa
          requires:
            - build
      - slack/on-hold:
          mentions: "@user"
          requires:
            - push
```

배포 알림도 Artifact 업로드와 마찬가지로 배포 브랜치에서만 실행되어야 하지만 이 Job에서는 `filter`가 없다. 그이유는 바로 `requires` 때문인데 `push` Job에 의존적이기 때문에 만약 `push` Job이 실행되지 않으면 함께 실행되지 않기 때문에 별도로 `filter`를 추가하지 않아도 된다. 만약 `requires`가 없다면 병렬로 실행되기 때문에 `filter`를 추가해 주어야 할것이다.

한번만에 성공하였으면 좋았겠지만 이번에도 배포 알림 Job이 실패하였다.

![slack-noti-pipeline-failed](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/slack-noti-pipeline-failed.png)

로그를 보니 `SLACK_ACCESS_TOKEN` 환경 변수가 정의되어 있지 않아서인듯 하다.

```
In order to use the Slack Orb (v4 +), an OAuth token must be present via the SLACK_ACCESS_TOKEN environment variable.
```

`SLACK_ACCESS_TOKEN` 환경 변수를 프로젝트에 추가해 주자.

![slack-token-environment-variable](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/slack-token-environment-variable.png)

workflow를 다시 실행해주면 성공적으로 배포알림이 실행되는 것을 확인할 수 있다.

![slack-noti-pipeline-success](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/slack-noti-pipeline-success.png)

![slack-noti-check](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/slack-noti-check.png)

### 배포 승인

담당자가 배포 여부를 결정할 수 있도록 [manual approval](https://circleci.com/docs/2.0/workflows/#holding-a-workflow-for-a-manual-approval){: target="\_blank" }을 추가해 주자.

```yml
version: 2.1

orbs:
  gradle: circleci/gradle@2.2.0
  aws-ecr: circleci/aws-ecr@7.2.0
  slack: circleci/slack@4.4.4

executors:
  my-executor:
    docker:
      - image: cimg/openjdk:16.0.2
    resource_class: medium

jobs:
  push:
    executor: my-executor
    steps:
      - checkout
      - aws-ecr/ecr-login
      - run:
          command: ./gradlew jib

workflows:
  build-and-deploy:
    jobs:
      - gradle/run:
          name: build
          executor: my-executor
      - push:
          filters:
            branches:
              only:
                - main
                - dev
                - qa
          requires:
            - build
      - slack/on-hold:
          mentions: "@user"
          requires:
            - push
      - manual-approval:
          type: approval
          requires:
            - push
```

배포 승인 Job의 특이점은 `requires`가 `push` Job이라는 것이다. 즉 배포 알림 Job과 병렬로 실행된다는 것이다. 배포알림의 목적을 생각하면 어찌보면 당연하게 보일 수 있는데, 만약 담당자가 workflow를 모니터링하고 있거나 배포알림이 지연되는 경우 혹은 실패하였을 경우에도 배포 승인을 통해서 배포 파이프라인을 이어갈 수 있다.

배포 승인이 준비되면 담당자는 workflow dashboard에서 승인 버튼을 눌러 다음 pipeline을 진행할 수 있다. 만약 다음 pipeline을 진행하지 않으려면 해당 workflow를 무시하면 된다.

![approval-pipeline](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/approval-pipeline.png)

### 배포

helm upgrade 명령어를 통해서 kubernetes에 웹 어플리케이션을 배포할 수 있다. 이 글에서는 AWS의 EKS에 환경을 세팅해두었으므로 [Circle CI의 EKS Orb](https://circleci.com/developer/orbs/orb/circleci/aws-eks){: target="\_blank" }와 [Circle CI의 helm](https://circleci.com/developer/orbs/orb/circleci/helm){: target="\_blank" }을 추가한다.

helm 명령어를 통해 EKS에 배포하기 위해서는 접근 권한이 필요하다. 그래서 AWS EKS의 [update-kubeconfig-with-authenticator](https://circleci.com/developer/orbs/orb/circleci/aws-eks#commands-update-kubeconfig-with-authenticator){: target="\_blank" }를 통해 접근권한을 얻고 helm upgrade를 실행하도록 구성해야한다.

```yml
version: 2.1

orbs:
  gradle: circleci/gradle@2.2.0
  aws-ecr: circleci/aws-ecr@7.2.0
  slack: circleci/slack@4.4.4
  aws-eks: circleci/aws-eks@1.1.0
  helm: circleci/helm@1.2.0

executors:
  my-executor:
    docker:
      - image: cimg/openjdk:16.0.2
    resource_class: medium

jobs:
  push:
    executor: my-executor
    steps:
      - checkout
      - aws-ecr/ecr-login
      - run:
          command: ./gradlew jib
  deploy:
    executor: my-executor
    steps:
      - checkout
      - run:
          command: |
            echo 'export SHORT_GIT_HASH="$(echo $CIRCLE_SHA1 | cut -c -7)"' >> $BASH_ENV
      - aws-eks/update-kubeconfig-with-authenticator:
          cluster-name: "my-cluster"
          cluster-authentication-role-arn: "arn:aws:iam::000000000000:role/k8s-authenticator"
          install-kubectl: true
      - helm/upgrade-helm-chart:
          helm-version: "v3.1.2"
          release-name: "my-release"
          chart: "./helm-chart"
          values: "./helm-chart/values.yaml"
          values-to-override: "image.versionTag=$SHORT_GIT_HASH"
          atomic: true

workflows:
  build-and-deploy:
    jobs:
      - gradle/run:
          name: build
          executor: my-executor
      - push:
          filters:
            branches:
              only:
                - main
                - dev
                - qa
          requires:
            - build
      - slack/on-hold:
          mentions: "@user"
          requires:
            - push
      - manual-approval:
          type: approval
          requires:
            - push
      - deploy:
          requires:
            - manual-approval
```

`deploy` Job을 순서대로 살펴보자.

- checkout

  먼저 소스코드를 받는다. 빌드된 이미지는 이미 ECR에 업로드 되어있는데 왜 받는지 의문이 생길 수 있는데, 바로 helm chart 파일들 때문이다. helm chart 파일들을 별도 저장소에 보관할 수도 있지만 소스코드와 함께 보관하면 쉽게 받아올 수도 있고 컨텍스트가 분산되지 않기 때문에 여러모로 장점이 많다.

- 환경변수 설정

  작업 브랜치, 클러스터, Commit Hash 등 실행 환경에 따라 변수를 다르게 하고 싶은 경우 환경변수를 설정할 수 있다. helm chart를 어떻게 구성하였는지에 따라 환경변수는 각자 다를 수 있으니 본인의 상황에 맞게 구성하자. 여기서는 앞서 말한 특정 Artifact를 가져오기 위해서 Git의 짧은 Hash값을 환경변수로 지정하였다.

- EKS 인증

  Config를 적용할 Cluster 명과 인증을 위한 AWS 계정의 ARN을 설정해준다. 그리고 kube-ctl을 사용하기 위해 설치 옵션을 함께 준다.

  그리고 인증에서 눈치챘겠지만 `update-kubeconfig-with-authenticator`도 환경변수를 필요로 한다. (`AWS_ACCESS_KEY_ID`와 `AWS_SECRET_ACCESS_KEY`는 `ecr-login`에서 사용된 변수와 동일하기 때문에 해당 계정의 권한을 체크해 보길 바란다.)

  ![eks-environment-variable](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/eks-environment-variable.png)

- Helm upgrade

  Helm upgrade 명령어를 실행해 준다. 다만 기본 버전이 `2.x.x` 버전이므로 3버전을 사용하고 싶다면 버전을 명시해 주어야 한다. chart 파일의 위치를 지정해주고 그외에 helm 명령어를 수행하기 위한 필수 설정을 해주자. `atomic` 옵션은 helm upgrade 실패 시 배포 이전 버전으로 롤백해주는 옵션이다. kubernetes pod들이 모두 배포될때까지 기다리는 `--wait` 옵션이 함께 있으므로 유용하게 사용할 수 있다.

workflow 설정에 문제가 없고 helm chart 설정을 잘 하였다면 파이프라인이 성공적으로 종료된 것을 확인할 수 있다. (다이어그램에 정의한 순서와 조금 다른건 배포알림은 병렬로 비 동기적으로 실행되기 때문이다.)

![deploy-pipeline-success](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/deploy-pipeline-success.png)

### context 설정

만약 하나의 project만 관리하는 것이 아니라 여러 프로젝트를 관리한다면 환경변수를 매 프로젝트마다 설정해주는 것은 번거로운 일이 될 수 있다.(물론 각 프로젝트마다 다른 환경 변수 값을 가진다면 이야기가 달라지겠지만...) 그래서 Circle CI에서는 [Context](https://circleci.com/docs/2.0/contexts/){: target="\_blank" }라는 매커니즘을 통해서 공유 환경변수를 제공한다.

![context](/assets/images/posts/ciecle-ci-pipeline-with-spring-and-k8s/context.png)

위에서 Proejct environment variable로 정의해두었던 환경변수들을 context에 등록하고 해당 context를 사용하도록 workflow를 바꿔보자.

```yml
version: 2.1

orbs:
  gradle: circleci/gradle@2.2.0
  aws-ecr: circleci/aws-ecr@7.2.0
  slack: circleci/slack@4.4.4
  aws-eks: circleci/aws-eks@1.1.0
  helm: circleci/helm@1.2.0

executors:
  my-executor:
    docker:
      - image: cimg/openjdk:16.0.2
    resource_class: medium

jobs:
  push:
    executor: my-executor
    steps:
      - checkout
      - aws-ecr/ecr-login
      - run:
          command: ./gradlew jib
  deploy:
    executor: my-executor
    steps:
      - checkout
      - run:
          command: |
            echo 'export SHORT_GIT_HASH="$(echo $CIRCLE_SHA1 | cut -c -7)"' >> $BASH_ENV
      - aws-eks/update-kubeconfig-with-authenticator:
          cluster-name: "my-cluster"
          cluster-authentication-role-arn: "arn:aws:iam::000000000000:role/k8s-authenticator"
          install-kubectl: true
      - helm/upgrade-helm-chart:
          helm-version: "v3.1.2"
          release-name: "my-release"
          chart: "./helm-chart"
          values: "./helm-chart/values.yaml"
          values-to-override: "image.versionTag=$SHORT_GIT_HASH"
          atomic: true

workflows:
  build-and-deploy:
    jobs:
      - gradle/run:
          name: build
          executor: my-executor
      - push:
          context: aws-ecr
          filters:
            branches:
              only:
                - main
                - dev
                - qa
          requires:
            - build
      - slack/on-hold:
          context: slack
          mentions: "@user"
          requires:
            - push
      - manual-approval:
          type: approval
          requires:
            - push
      - deploy:
          context: aws-eks
          requires:
            - manual-approval
```

## 마치며

Circle CI를 처음 사용하는 사람들을 위해 실제 작업할때처럼 적용해보고 이슈가 있으면 대응하는 형식으로 글을 적어보았다. 그러다보니 내용이 좀 장황해진 느낌이 있기는 하다. 하지만 왜 Pipeline에 적혀진 설정이 적히게 되었는지 이해하는데에는 도움이 되지 않았을까 하는 기대를 해본다.

Spring Boot 어플리케이션을 구성하는 것에서 부터 AWS 서비스 설정, Kubernetes 환경구성과 Helm chart 작성까지 모두 하였다면 완전 초보도 따라하면서 웹 어플리케이션을 배포해 볼 수 있는 튜토리얼을 만들 수 있었을 텐데라는 아쉬움이 있다. 하지만 그걸 모두 적기에는 글이 너무나도 길어질 것 같아서 CI 작성에만 집중을 해보았다.

모쪼록 이 글을 읽는 분들에게 도움이 되었길 바란다.
