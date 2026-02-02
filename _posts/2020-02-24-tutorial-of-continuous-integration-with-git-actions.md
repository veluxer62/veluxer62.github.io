---
layout: single
title: "GIT Actions을 이용한 CI/CD 적용기 - CI편"
categories:
  - TUTORIALS
tags:
  - CI
  - continuous integration
  - GIT Actions
  - GIT
  - secrets
toc: true
toc_sticky: true
excerpt: 이 글은 현재 개인프로젝트로 개발하고 있는 OCCUPYING Repository에 GIT Actions을 이용하여 CI/CD를 적용해 보면서 겪었던 것들을 적은 글이다.
---

## Introduction

이 글은 현재 개인프로젝트로 개발하고 있는 [OCCUPYING](https://github.com/veluxer62/occupying){: target="\_blank" } Repository에 [GIT Actions](https://help.github.com/en/actions){: target="\_blank" }을 이용하여 CI/CD를 적용해 보면서 겪었던 것들을 적은 글이다.

내용이 길어질것 같아 먼저 CI(Continuous Integration)에 대한 내용을 먼저 적고 후속 글로 CD(Continuous Deployment)에 대한 글을 적어보도록 하겠다.

[GIT Actions을 이용한 CI/CD 적용기 - CD편](/tutorials/tutorial-of-continuous-deployment-with-git-actions/){: target="\_blank" }

## GIT Actions

GIT Actions은 2019년 11월에 정식으로 출시되면서 많은 개발자들이 CI/CD로 이용하고 있다. CI/CD로 [Jenkins](https://jenkins.io/){: target="\_blank" }, [Travis CI](https://travis-ci.org/){: target="\_blank" } [Azure DevOps](https://azure.microsoft.com/ko-kr/services/devops/){: target="\_blank" }등이 있긴 하지만 Github를 사용하고 있다면 GIT Action을 사용하지 않을 이유가 없다고 생각한다.

**GIT Actions**는 Public Repository에 한해서는 무료이며, Private Repository에서도 제한적으로 무료로 사용할 수 있다. 사용에 대해서도 제약사항도 존재하는데 제약사항에 대해서는 [링크](https://help.github.com/en/actions/getting-started-with-github-actions/about-github-actions#usage-limits){: target="\_blank" } 통해 확인할 수 있다.

## CI

CI(Continuous Integration)이란 개발자가 작성한 변경사항을 지속적으로 자동화퇸 테스트 및 빌드 도구가 실행되어 중앙 브런치에 병합하는 개발 방식을 말한다. 지속적 통합은 장기간에 걸친 개발 후 통합 과정에서의 불필요한 작업을 최소한으로 하고 버그의 빠른 발견을 위해 활용된다. 그래서 자동화된 테스트가 반드시 존재해야 한다.

## How to

그럼 이제 **GIT Actions**를 이용해서 어떻게 CI를 적용하였는지 알아보자.

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

**Occupying**의 CI 시나리오는 아래와 같다.

1. pull request 시 모든 브런치에서 CI를 실행한다.
2. 실행은 ubuntu OS에서 실행한다.
3. pull request 내용을 checkout 한다.
4. JDK를 설치한다
5. secret 파일을 복사한다.
6. 소스를 build 한다.

#### 실행조건 설정

**Complete**의 필수 조건 중 상태 체크가 성공해야 함을 설정하였다. 이는 혹시나 master에 잘못된 소스가 병합됨을 방지하기 위해서이다.

**Require status check 설정방법은 [링크](https://help.github.com/en/github/administering-a-repository/about-required-status-checks){: target="\_blank" }를 참고하기 바란다\*\***

Pull Request시 모든 브런치에 대해 CI가 동작하도록 설정하였다. [`on` syntax 문서 참고](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#on){: target="\_blank" }

```yaml
on:
  pull_request:
    branches:
      - "**"
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

Pull Request한 코드를 Checkout 한다. [actions/checkout 참고](https://github.com/actions/checkout#checkout-pull-request-head-commit-instead-of-merge-commit){: target="\_blank" }

**Gradle Template**에서 제공해주는 step에 `name` 속성과 `with`속성을 추가하였다.

```yaml
jobs:
  build:
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          ref: {% raw %}${{ github.event.pull_request.head.sha }}{% endraw %}
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

CI 코드를 만들면서 삽질을 많이한 부분중에 하나이다.
secret.yml 파일에는 테스팅을 위한 계정정보가 입력되어 있다. 해당 부분은 소스에 따로 커밋은 하지 않고 .gitignore로 처리해서 필요한 서버에 새롭게 생성해서 사용하도록 했다. 그러다보니 최초 Gradle template에서 제공해주는 설정으로만 CI를 실행하면 테스트 실행시 해당 파일을 찾을 수 없어 오류가 발생하였다. 이 문제를 해결 할 수 있는 방안은 3가지 정도로 생각할 수 있었다.

1. git-secret
2. git-crypt
3. Github Secret

이중 간단히 설정할 수 있고 git-secret이나 git-crypt도 결국 public key를 github secret에 설정을 해주어야 해서 Github Secret을 사용해보기로 결정하였다. ~~windows 환경에서 1번과 2번을 설정하고 테스트해보았는데 불편해서 못해먹겠더라~~

Github Secret은 아래 그림과 같이 Settings > Secrets 에서 추가할 수 있다.

![Github-Secret](/assets/images/posts/tutorial-of-ci-with-git-actions/github-secret.png)

Secret의 value값은 반드시 암호화해서 저장한 후 저장된 값을 사용할때 복호화 해서 사용해야 한다. [Github 문서 참고](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#about-encrypted-secrets){: target="\_blank" } 데이터를 복호화 하지 않고 사용하려고 하면 값을 자동으로 제거해서 아래와 같이 제대로 사용할 수 없게된다.

secret 값을 plain text로 저장하여 사용하는 바람에 아래 그림은 테스트가 계속 실패해서 secret.yml 값을 확인해본 결과를 캡쳐한 것이다. `*** *** *** *** *** ***`로 표시되는 것을 볼 수 있다.

![Github-Secret](/assets/images/posts/tutorial-of-ci-with-git-actions/test-secret-data.png)

그래서 Secret는 Base64로 encrypt 한 후 값을 저장하였다.

Github의 Secret을 Git Actions에서 가져올 때에 사용되는 변수는 `secrets.KEY 값`이다. 그 변수를 이용하여 아래와 같이 설정하였다.

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

`env`로 사용할 변수를 할당하였고 `run`을 통해 스크립트를 실행하여 secret.yml파일을 생성하였다.

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
`gradlew build` 명령어는 테스트틀 포함하고 있다. 만약 테스트를 생략하고 싶다면 `gradlew build -x test`명령어를 사용하면 되지만 test가 없는 CI는 완전한 CI라고 할 수 없으니 권장하지 않는다.

```yaml
jobs:
  build:
    steps:
      - name: Build with Gradle
        run: ./gradlew build
```

### 전체 소스

```yaml
name: CI

on:
  pull_request:
    branches:
      - "**"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          ref: {% raw %}${{ github.event.pull_request.head.sha }}{% endraw %}

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
```

### 실행 확인

위에 작성된 코드로 실행해 보면 정상적으로 테스트 및 빌드가 성공한 것을 확인 할 수 있다.

![Build-Result](/assets/images/posts/tutorial-of-ci-with-git-actions/ci-success-result.png)

## Wrap up

**Git Actions**를 이용하여 자동화된 CI를 하는 방법에 대해 알아보았다. 예상외의 문제에서 시행착오를 좀 겪었지만 좋은 경험이었다. 다음 글에서는 중앙 브런치 병합 시 동작할 CD에 대한 글을 적어보고자 한다.

[GIT Actions을 이용한 CI/CD 적용기 - CD편](/tutorials/tutorial-of-continuous-deployment-with-git-actions/){: target="\_blank" }

> [https://help.github.com/en/actions](https://help.github.com/en/actions){: target="\_blank" }
>
> [https://help.github.com/en/github/administering-a-repository/about-required-status-checks](https://help.github.com/en/github/administering-a-repository/about-required-status-checks){: target="\_blank" }
>
> [https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions){: target="\_blank" }
