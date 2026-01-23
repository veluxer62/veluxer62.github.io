---
layout: single
title: "GIT Actions을 이용한 CI/CD 적용기 - CD편"
categories:
  - TUTORIALS
tags:
  - CD
  - continuous deployment
  - GIT Actions
  - GIT
  - secrets
toc: true
toc_sticky: true
excerpt: 지난 GIT Actions을 이용한 CI/CD 적용기 - CI편에 이어 이번 글은 CD에 대한 내용을 적어보겠다. CD가 Trigger 되는 시점은 master 브런치에 병합이 될때 실행되며, OCCUPYING프로젝트에서 CD는 Continuous Delivery가 아닌 Continuous Deployment로 하였다.
---

## Introduction

지난 [GIT Actions을 이용한 CI/CD 적용기 - CI편](/tutorials/tutorial-of-continuous-integration-with-git-actions/){: target="\_blank" }에 이어 이번 글은 CD에 대한 내용을 적어보겠다. CD가 Trigger 되는 시점은 master 브런치에 병합이 될때 실행되며, [OCCUPYING](https://github.com/veluxer62/occupying){: target="\_blank" }프로젝트에서 CD는 [Continuous Delivery](https://en.wikipedia.org/wiki/Continuous_delivery){: target="\_blank" }가 아닌 [Continuous Deployment](https://en.wikipedia.org/wiki/Continuous_deployment){: target="\_blank" }로 하였다.

## CD

CD는 Continuous Delivery와 Continuous Deployment로 나뉠 수 있다. Continuous Delivery는 운영환경에 릴리스하기 위한 코드의 변경이 자동으로 준비되는 방식을 말한다. 즉 병합된 코드가 빌드 및 테스트된 후 스테이징 환경에 푸시된다. 그 후 여러 테스트를 거친 후 배포 담당자는 업데이트에 대한 배포 승인을 수동으로 하게 된다. Continuous Deployment는 Continuous Delivery와는 달리 병합된 코드가 명시적 승인 없이 자동으로 운영환경에 배포되는 방식을 말한다.

## How to

그럼 이제 **GIT Actions**를 이용해서 어떻게 CD를 적용하였는지 알아보자.

지난 CI에서 적용하였던 설정이 많이 중복된다. 도커 환경에서 소스를 빌드한 후 그 빌드한 파일을 다은 도커환경에서 사용하려고 하기 보다 다시한번 빌드한 후 그 파일을 배포를 위해 사용하는 것이 효율적인것 같아서 CI 설정을 CD에서 다시 사용하였다.(~~중복되는 부분을 제거하여 재사용할 수 있는지는 좀더 알아보고 있는 중이다.~~)

### Workflows 생성

먼저 Repository에서 **Actions** 탭을 선택한다.

![Click-Action-Tab](/assets/images/posts/tutorial-of-ci-with-git-actions/click-actions-tab.png)

화면에 이동을 하면 현재 사용된 언어를 참고하여 **workflows**들을 추천해준다. 만약 원하는 항목이 없으면 `Workflows for Python, Maven, Docker and more...` 버튼을 클릭하여 다른 workflows를 찾아보자.

![Recommend-workflows](/assets/images/posts/tutorial-of-ci-with-git-actions/recommend-workflows.png)

**Occupying** 프로젝트는 Spring Boot를 사용하고 있으며 Gradle로 빌드를 하기 때문에 **Gradle workflow**를 선택하였다.

![Gradle-workflows](/assets/images/posts/tutorial-of-ci-with-git-actions/gradle-workflows.png)

`Set up this workflow`버튼을 클릭하면 아래와 같이 템플릿이 적용된 YAML 파일이 생성되면서 코드 작성화면으로 이동한다.

![Gradle-template](/assets/images/posts/tutorial-of-ci-with-git-actions/gradle-template.png)

### Workflows 작성

**Occupying**의 CD 시나리오는 아래와 같다.

1. Default Branch (현재 사용하고 있는 프로젝트에서는 _master_ 브런치 이다.)에 코드 병합 시 CD를 실행한다.
2. 실행은 ubuntu OS에서 실행한다.
3. Default Branch에 병합된 코드를 checkout한다.
4. JDK를 설치한다
5. secret 파일을 복사한다.
6. 소스를 build 한다.
7. build한 소스 파일을 서버에 전달한다.
8. 전달한 소스파일을 이용하여 서버를 실행한다.

#### 실행조건 설정

Default Branch인 _master_ 브런치에 코드가 병합될 때 CD가 동작하도록 설정하였다. [`on` syntax 문서 참고](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#on){: target="\_blank" }

```yaml
on:
  push:
    branches:
      - master
```

#### 실행 Job 설정

위에서 정의한 실행 조건에 맞게 동작할 Job을 설정한다. Job에서는 어떤 OS 환경에서 여러개의 step의 설정하여 동작하도록 설정할 수 있다.
[`jobs` syntax 문서 참고](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobs){: target="\_blank" }

##### 실행 OS 환경

Gradle Workflow Template에서 기본적으로 제공해주는 Ubuntu를 그대로 설정하였다.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

##### Checkout

*master*에 병합된 코드를 Checkout 한다. [actions/checkout 참고](https://github.com/actions/checkout#checkout-pull-request-head-commit-instead-of-merge-commit){: target="\_blank" }

**Gradle Template**에서 기본적으로 제공해주는 step으로 추가적으로 변경하진 않았다.

```yaml
jobs:
  deploy:
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
```

##### JDK 설치

**Gradle Template**에서 제공해주는 step으로 추가적으로 변경하진 않았다.

```yaml
jobs:
  build:
    steps:
      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
```

##### secret 파일 복사

**secret**파일을 checkout한 코드에 복사한다. (해당 코드가 필요한 이유는 이전에 글의 [secret 파일 복사](/tutorials/tutorial-of-continuous-integration-with-git-actions/#secret-파일-복사){: target="\_blank" }에서 자세히 볼 수 잇다.)

```yaml
jobs:
  build:
    steps:
      - name: Copy secret
        env:
          OCCUPY_SECRET: {% raw %}${{ secrets.OCCUPY_SECRET }}{% endraw %}
          OCCUPY_SECRET_DIR: src/main/resources
          OCCUPY_SECRET_DIR_FILE_NAME: secret.yml
        run: echo $OCCUPY_SECRET | base64 --decode > $OCCUPY_SECRET_DIR/$OCCUPY_SECRET_DIR_FILE_NAME
```

##### gradlew 실행 권한 설정

**Gradle Template**에서 제공해주는 step으로 추가적으로 변경하진 않았다. gradlew 파일에 실행권한을 줘서 실행할 수 있도록 하는 작업이다.

```yaml
jobs:
  build:
    steps:
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew
```

##### gradlew build

**Gradle Template**에서 제공해주는 step으로 추가적으로 변경하진 않았다. gradlew의 명령어를 이용해 어플리케이션을 빌드한다.

```yaml
jobs:
  build:
    steps:
      - name: Build with Gradle
        run: ./gradlew build
```

##### deliver file

build에 성공한 파일을 **scp**를 이용하여 서버로 전송한다. [**appleboy/scp-action** 문서 참고](https://github.com/appleboy/scp-action){: target="\_blank" }

전송할 서버의 *host*와 _username_, _key_, _port_ 정보는 민감한 정보이므로 Github Secret 설정으로 적용하였다.
Github Secret은 아래 그림과 같이 Settings > Secrets 에서 추가할 수 있다.
![Github-Secret](/assets/images/posts/tutorial-of-ci-with-git-actions/github-secret.png)

```yaml
jobs:
  deploy:
    steps:
      - name: Deliver File
        uses: appleboy/scp-action@master
        with:
          host: {% raw %}${{ secrets.OCCUPY_SSH_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.OCCUPY_SSH_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.OCCUPY_SSH_KEY }}{% endraw %}
          port: {% raw %}${{ secrets.OCCUPY_SSH_PORT }}{% endraw %}
          source: "build/libs/*.jar"
          target: "source"
          rm: true
```

전송 시 서버의 White List IP 설정이 필요할 수 있다. 관련 설정이 필요한 경우 [링크](https://help.github.com/en/actions/reference/virtual-environments-for-github-hosted-runners){: target="\_blank" }를 참고하기 바란다.

##### deploy

SSH로 서버에 접속하여 전송된 소스파일을 이용하여 서버를 실행한다. [appleboy/ssh-action 참고](https://github.com/appleboy/ssh-action){: target="\_blank" }

전송할 서버의 *host*와 _username_, _key_, _port_ 정보는 민감한 정보이므로 Github Secret 설정으로 적용하였다.
Github Secret은 아래 그림과 같이 Settings > Secrets 에서 추가할 수 있다.
![Github-Secret](/assets/images/posts/tutorial-of-ci-with-git-actions/github-secret.png)

만약 서버가 이미 실행중이라면 실행중인 서버를 중지하고 다시 시작한다.
~~현재 운영중인 서버가 1대 밖에 없고 가용성을 많이 고민하지 않았다. 만약 사용유저가 많아지고 가용성을 고민해야 한다면 ECS나 Elastic Beanstalk을 고려해볼 생각이다.~~

```yaml
jobs:
  deploy:
    steps:
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: {% raw %}${{ secrets.OCCUPY_SSH_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.OCCUPY_SSH_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.OCCUPY_SSH_KEY }}{% endraw %}
          port: {% raw %}${{ secrets.OCCUPY_SSH_PORT }}{% endraw %}
          script: |
            SOURCE_DIR=source/build/libs
            FILE_NAME=`find $SOURCE_DIR/*.jar -printf "%f\n"`
            PID=`ps -ef | grep occupying | grep sudo | grep -v "bash -c" | awk '{print $2}'`

            if [ -z "$PID" ]; then
                    echo "#### THERE IS NO PROCESS ####"
            else
                    echo "#### KILL $PID ####"
                    sudo kill $PID
            fi

            echo "#### RUN $SOURCE_DIR/$FILE_NAME ####"

            sudo java -jar $SOURCE_DIR/$FILE_NAME > /dev/null 2>&1 &
```

전송 시 서버의 White List IP 설정이 필요할 수 있다. 관련 설정이 필요한 경우 [링크](https://help.github.com/en/actions/reference/virtual-environments-for-github-hosted-runners){: target="\_blank" }를 참고하기 바란다.

### 전체 소스

```yaml
name: CD

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8

      - name: Copy secret
        env:
          OCCUPY_SECRET: {% raw %}${{ secrets.OCCUPY_SECRET }}{% endraw %}
          OCCUPY_SECRET_DIR: src/main/resources
          OCCUPY_SECRET_DIR_FILE_NAME: secret.yml
        run: echo $OCCUPY_SECRET | base64 --decode > $OCCUPY_SECRET_DIR/$OCCUPY_SECRET_DIR_FILE_NAME

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build

      - name: Deliver File
        uses: appleboy/scp-action@master
        with:
          host: {% raw %}${{ secrets.OCCUPY_SSH_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.OCCUPY_SSH_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.OCCUPY_SSH_KEY }}{% endraw %}
          port: {% raw %}${{ secrets.OCCUPY_SSH_PORT }}{% endraw %}
          source: "build/libs/*.jar"
          target: "source"
          rm: true

      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: {% raw %}${{ secrets.OCCUPY_SSH_HOST }}{% endraw %}
          username: {% raw %}${{ secrets.OCCUPY_SSH_USERNAME }}{% endraw %}
          key: {% raw %}${{ secrets.OCCUPY_SSH_KEY }}{% endraw %}
          port: {% raw %}${{ secrets.OCCUPY_SSH_PORT }}{% endraw %}
          script: |
            SOURCE_DIR=source/build/libs
            FILE_NAME=`find $SOURCE_DIR/*.jar -printf "%f\n"`
            PID=`ps -ef | grep occupying | grep sudo | grep -v "bash -c" | awk '{print $2}'`

            if [ -z "$PID" ]; then
                    echo "#### THERE IS NO PROCESS ####"
            else
                    echo "#### KILL $PID ####"
                    sudo kill $PID
            fi

            echo "#### RUN $SOURCE_DIR/$FILE_NAME ####"

            sudo java -jar $SOURCE_DIR/$FILE_NAME > /dev/null 2>&1 &
```

### 실행 확인

위에 작성된 코드로 실행해 보면 정상적으로 테스트 및 빌드가 성공한 것을 확인 할 수 있다.

![Deploy-Result](/assets/images/posts/tutorial-of-cd-with-git-actions/cd-success-result.png)

## Wrap up

지금까지 **Git Actions**를 이용하여 자동화된 CD를 하는 방법에 대해 알아보았다. 여기에서는 EC2에 배포를 하는 거라 **SSH**를 이용하여 서버에 코드를 배포하였지만 AWS CLI를 이용하여 ECS에 배포하거나 Elastic Beanstalk에 배포하는 템플릿도 있으니 사용해보면 좋을 것 같다.

> [https://en.wikipedia.org/wiki/Continuous_delivery](https://en.wikipedia.org/wiki/Continuous_delivery){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Continuous_deployment](https://en.wikipedia.org/wiki/Continuous_deployment){: target="\_blank" }
>
> [https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#on){: target="\_blank" }
>
> [https://github.com/actions/checkout#checkout-pull-request-head-commit-instead-of-merge-commit](https://github.com/actions/checkout#checkout-pull-request-head-commit-instead-of-merge-commit){: target="\_blank" }
>
> [https://github.com/appleboy/scp-action](https://github.com/appleboy/scp-action){: target="\_blank" }
>
> [https://help.github.com/en/actions/reference/virtual-environments-for-github-hosted-runners](https://help.github.com/en/actions/reference/virtual-environments-for-github-hosted-runners){: target="\_blank" }
>
> [https://github.com/appleboy/ssh-action](https://github.com/appleboy/ssh-action){: target="\_blank" }
