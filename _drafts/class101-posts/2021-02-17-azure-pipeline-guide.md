---
layout: single
title: "에저 파이프라인 가이드"
categories:
  - POST
---

# Azure 파이프라인 사용가이드

# 사전 설정

- 서비스 권한설정

    Azure Devops에서 다른 서비스에 대해 권한 설정을 하려면 Service connections에서 권한 설정을 할 수 있다.

    **권한설정이 되어 있지 않으면 파이프라인 구성을 할 수 없으므로 반드시 사전에 권한 설정을 해두자.**

    단, 서비스 권한은 프로젝트 관리자 권한이 있어야 하니 권한을 확인하자.

    ## AWS 권한 설정

    ### Service Connection 추가

    Project settings → Service connections → New service connection 버튼을 눌러서 Service Connection 추가 화면으로 이동하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.42.01_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.42.01_PM.png)

    ### AWS 서비스 선택

    AWS 권한을 추가할 것이므로 AWS를 선택하고 Next 버튼을 누르자

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.47.38_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.47.38_PM.png)

    ### 필수 정보 입력

    `Access Key ID` 와 `Secret Access Key` `Service connection name` 을 입력하고 Save를 누르자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.52.46_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.52.46_PM.png)

    Access 정보는 Bitwarden에 개발자 공용 계정 폴더에 `Azure 파이프라인 슬렉 봇 Token` 아이템에 기입되어 있으니 복사해서 사용하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.51.11_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.51.11_PM.png)

    ### 확인

    저장 후 목록에서 Connection 정보를 확인하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.55.26_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.55.26_PM.png)

    ## Github 권한 설정

    ### Service Connection 추가

    Project settings → Service connections → New service connection 버튼을 눌러서 Service Connection 추가 화면으로 이동하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.42.01_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.42.01_PM.png)

    ### Github 서비스 선택

    Github 권한을 추가할 것이므로 Github를 선택하고 Next 버튼을 누르자

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.57.55_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_4.57.55_PM.png)

    ### Github 인증

    Azure pipelines를 선택하고 Authorize 버튼을 클릭한다.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.00.58_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.00.58_PM.png)

    ### 이름 설정 및 저장

    인증이 완료되었으면 Service connection name 을 입력하고 Save 버튼을 눌러 저장하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.03.00_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.03.00_PM.png)

    ### 저장 확인

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.04.35_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.04.35_PM.png)

# Pipeline

파이프라인은 직접 하나하나 설정해주어도 되지만 현재 가이드에서는 이미 구성되어있는 파이프라인에 개인 설정을 변경해주는 방식으로 진행하려고 한다. 상세 구성방법에 대해서는 추가적으로 작성할 예정이고 궁금한 점이 있다면 @ 케이 를 태그해서 물어보자.

![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.49.06_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.49.06_PM.png)

- PR 파이프라인

    PR 단계에서 테스트 코드를 실행한다던지, 빌드 테스트를 실행시켜서 오류가 발생하는 코드가 master 브런치에 병합되는 것을 막는 용도로 파이프라인을 만들 수 있다. 해당 파이프라인을 생성하는 방법을 살펴보자

    ## Pipeline 메뉴 이동

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-06_at_12.20.27_AM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-06_at_12.20.27_AM.png)

    ## Import pipeline

    만약 처음 파이프라인을 만든다면 아래와 같이 화면이 나올 것이다. Create pipeline 버튼 옆에 `:` 버튼을 누른 후 Import Pipeline을 누르자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.15.49_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.15.49_PM.png)

    기존에 파이프라인이 있다면 아래와 같이 화면이 나올것이다. New pipeline 버튼 옆에 `:` 버튼을 누른 후 Import Pipeline을 누르자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.21.17_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.21.17_PM.png)

    PR 파이프라인에서 사용할 파일은 아래 링크에서 다운로드 하면된다.

    [Build Test.json](/assets/images/draft/class101-posts/azure-pipeline/Build_Test.json)

    파일 선택 후 Import 버튼을 누르자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.24.36_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.24.36_PM.png)

    ## Pipline 설정

    먼저 Pipeline에서 name과 Agent pool을 설정해 주자.

    이름은 본인이 하고 싶은 이름을 하면되고 Agent pool은 화면과 같이 `Azure pipelines`와 `ubuntu-18.04`를 선택해 주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.27.23_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.27.23_PM.png)

    ## Get sources 설정

    Github에서 소스를 가져올 Repository를 선택하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.30.10_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.30.10_PM.png)

    ## Cache 설정

    캐시는 빌드의 속도를 좀더 빠르게 도와 준다. 다만 Key가 Azure Pipeline에서 글로벌로 되어 있는 것처럼 보여서 Key을 각 파이프라인마다 다르게 설정해주어야 한다. 나름대로 규칙을 정해봤는데 변경하고 싶으면 의견주면 좋겠다. [참고링크](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/caching?view=azure-devops)

    규칙 - `gradle | {서비스명} | {파이프라인명}`

    예시 - `gradle | inventory | build-test`

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.34.28_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.34.28_PM.png)

    ## Gradle 설정

    특별히 변경할건 없는데 Advanced 에서 JDK 버전과 메모리 설정은 체크하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.37.45_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.37.45_PM.png)

    ## 저장

    저장 후 바로 테스트 하려면 Save & queue를 저장만 하려면 Save를 선택하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.39.45_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.39.45_PM.png)

    ## 테스트 방법

    지정된 Repository에서 PR을 올려보자. Git Check에서 Pipeline이 실행 되면 정상적으로 적용된 것이다.

- CI 파이프라인

    CI 파이프라인은 코드가 마스터가 병합한 후에 동작하는 파이프라인으로 쿠버네티스에서 헬름차트를 통해 배포를 하기 위해서 ECR에 코드를 푸시하는 단계이다. PR 파이프라인과 아주 유사하지만 PR 파이프라인은 PR시 동작하고 CI 파이프라인은 코드가 마스터에 병합할 시에 동작한다는 차이점이 있다. 

    **현재 가이드라인에서의 CI는 JIB를 이용한 ECR push까지를 CI로 판단하였다. 만약 쿠버네티스에 배포하지 않고 ECR을 사용하지 않는 서비스라면 다르게 구성해야 할것이다.**

    ## Pipeline 메뉴 이동

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-06_at_12.20.27_AM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-06_at_12.20.27_AM.png)

    ## Import pipeline

    만약 처음 파이프라인을 만든다면 아래와 같이 화면이 나올 것이다. Create pipeline 버튼 옆에 `:` 버튼을 누른 후 Import Pipeline을 누르자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.15.49_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.15.49_PM.png)

    기존에 파이프라인이 있다면 아래와 같이 화면이 나올것이다. New pipeline 버튼 옆에 `:` 버튼을 누른 후 Import Pipeline을 누르자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.21.17_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.21.17_PM.png)

    CI 파이프라인에서 사용할 파일은 아래 링크에서 다운로드 하면된다.

    [Gradle CI.json](/assets/images/draft/class101-posts/azure-pipeline/Gradle_CI.json)

    파일 선택 후 Import 버튼을 누르자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.47.57_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.47.57_PM.png)

    ## Pipline 설정

    먼저 Pipeline에서 name과 Agent pool을 설정해 주자.

    이름은 본인이 하고 싶은 이름을 하면되고 Agent pool은 화면과 같이 `Azure pipelines`와 `ubuntu-18.04`를 선택해 주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.48.43_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.48.43_PM.png)

    ## Get sources 설정

    Github에서 소스를 가져올 Repository를 선택하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.49.44_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.49.44_PM.png)

    ## AWS ECR 로그인 설정

    JIB를 사용하기 위해서 AWS ECR 권한설정이 필요하다.
    사전 설정에서 추가해준 AWS Credentials를 선택해주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.05.49_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.05.49_PM.png)

    ## Cache 설정

    캐시는 빌드의 속도를 좀더 빠르게 도와 준다. 다만 Key가 Azure Pipeline에서 글로벌로 되어 있는 것처럼 보여서 Key을 각 파이프라인마다 다르게 설정해주어야 한다. 나름대로 규칙을 정해봤는데 변경하고 싶으면 의견주면 좋겠다. [참고링크](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/caching?view=azure-devops)

    규칙 - `gradle | {서비스명} | {파이프라인명}`

    예시 - `gradle | inventory | ci`

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.50.28_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.50.28_PM.png)

    ## Gradle 설정

    특별히 변경할건 없는데 Advanced 에서 JDK 버전과 메모리 설정은 체크하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.51.39_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.51.39_PM.png)

    ## 헬름차트 템플릿 복사

    일반적으로 빌드된 파일을 Artifacts로 만들어서 Realese에서 사용하지만 우리는 JIB를 사용하므로 쿠버네티스에 서버를 패포하기 위한 헬름 차트 템플릿만 Artifacts로 복사한다.

    헬름 차트 템플릿 폴더의 위치는 프로젝트마다 다를 것이기 때문에 복사할 컨텐츠의 경로를 프로젝트에 맞게 변경해주어야 한다.

    템플릿 디렉토리 명과 위치는 각 프로젝트마다 다르므로 Contents 경로를 잘 잡아주어야 한다. 하위 디렉토리및 파일을 모두 복사해야 하므로 와일드카드 문자열을 사용했다.

    예를 들면 Inventory Service는 헬름 차트 템플릿 폴더가 root 경로에 위치하므로 아래와 같이 경로를 주었다.

    `class101-inventory-service/**/*`

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.57.54_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_5.57.54_PM.png)

    실제 프로젝트 경로를 보면 아래와 같다.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-06_at_1.10.36_AM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-06_at_1.10.36_AM.png)

    ## 아티펙트 이름 변경

    빌드된 아티펙트는 Release에서 사용된다. 아티펙트 명은 Release에서 어떤 빌드를 트리거 할건지 선택할 수 있으므로 구별 가능한 이름으로 변경해주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_8.00.22_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_8.00.22_PM.png)

    ## 저장

    저장 후 바로 테스트 하려면 Save & queue를 저장만 하려면 Save를 선택하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_8.01.28_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_8.01.28_PM.png)

    ## 테스트 방법

    지정된 Repository에서 PR을 병합해 보자. CI 파이프라인이 실행되면 정상적으로 적용된 것이다.

# Release

릴리즈도 직접 설정 해줄 수 있지만 이미 구성되어있는 Task group을 추가하고 개일 설정을 변경해주는 방식으로 진행하려고 한다. 상세 구성방법은 추가적으로 작성할 계획이다.

릴리즈 시나리오는 아래 그림과 같이 CI 파이프라인이 완료되면 개발서버에 배포되고, 운영서버는 담당자의 승인 후에 배포가 되는 방식으로 진행되도록 하였다.

![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.09.23_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.09.23_PM.png)

## Task Groups

- Helm 설치 및 EKS 설정

    ### Import

    Pipelines → Task groups 메뉴로 이동한 후 Import a task group 버튼을 눌러주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.20.36_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.20.36_PM.png)

    Helm 설치 및 EKS 설정을 위한 Import 파일은 아래 링크에서 받자.

    [Install Helm And Config Kube.json](/assets/images/draft/class101-posts/azure-pipeline/Install_Helm_And_Config_Kube.json)

    Import 후 OK 버튼을 눌러 업로드 하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.23.09_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.23.09_PM.png)

    ### 이름 변경

    임시로 설정된 Task group의 이름을 변경하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.25.26_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.25.26_PM.png)

    ### 권한 설정

    EKS에 접근하기 위한 권한 설정을 해주자. 사전 설정에서 추가해준 AWS Service connection을 선택해주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.26.04_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.26.04_PM.png)

    ### 저장

    Save 버튼을 눌러 저장하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.28.31_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.28.31_PM.png)

- Helm 배포 설정

    ### Import

    Pipelines → Task groups 메뉴로 이동한 후 Import 버튼을 눌러주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.31.48_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.31.48_PM.png)

    Helm 설치 및 EKS 설정을 위한 Import 파일은 아래 링크에서 받자.

    [Helm Upgrade And Check Status.json](/assets/images/draft/class101-posts/azure-pipeline/Helm_Upgrade_And_Check_Status.json)

    Import 후 OK 버튼을 눌러 업로드 하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.32.50_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.32.50_PM.png)

    ### 이름 변경

    임시로 설정된 Task group의 이름을 변경하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.33.42_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.33.42_PM.png)

    ### Helm Upgrade 스크립트 권한 설정

    Helm upgrade 명령어를 실행하기 위한 권한설정을 해주자. 사전 설정에서 추가해준 AWS Service connection을 선택해주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.34.39_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.34.39_PM.png)

    ### Status Check 스크립트 권한 설정

    kubectl status 명령어를 실행하기 위한 권한설정을 해주자. 사전 설정에서 추가해준 AWS Service connection을 선택해주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.35.32_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.35.32_PM.png)

    ### 저장

    Save 버튼을 눌러 저장하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.37.49_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.37.49_PM.png)

## Release

- Artifact 추가

    ### 파이프라인 생성

    Pipelines → Release → New pipeline 버튼 클릭

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.39.52_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.39.52_PM.png)

    ### Empty job 생성

    템플릿 선택 창에서 아래 화면과 같이 Empty job 링크를 클릭하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.41.17_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.41.17_PM.png)

    ### Artifact 추가

    Add an artifact 를 선택하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.42.51_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.42.51_PM.png)

    Artifact 정보 입력을 해준 후 Add 버튼을 눌러 추가한다.

    Project는 현재 프로젝트를 선택하면 되고 Source는 CI 파이프라인을 선택하면 된다.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.44.02_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.44.02_PM.png)

    ### Trigger 설정

    추가한 Artifact에 번개 모양 아이콘을 눌러보자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.46.24_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.46.24_PM.png)

    Continuous deployment trigger 를 Enable 하자. (해당 설정을 해주지 않으면 CI 파이프라인 이후 자동으로 Release가 실행되지 않는다.)

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.48.26_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.48.26_PM.png)

- 개발 Stage 추가

    ### Tasks 설정

    처음 생성되어 있는 Stage의 Task를 설정해주자.

    상단 탭의 Tasks → Stage 1을 선택하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.53.00_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.53.00_PM.png)

    ### Stage 이름 설정

    원하는 스테이지 이름으로 변경하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.55.39_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_9.55.39_PM.png)

    ### Agent specification 변경

    Agent를 ubuntu 환경에서 실행하도록 변경하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.10.37_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.10.37_PM.png)

    ### Task 추가

    Agent job 에서 `+` 버튼을 누른 후 `helm` 으로 검색 하여 이전에 추가했던 Task group 들을 추가하자.

    순서는 아래와 같이 추가하면 된다.

    1. Install Helm And Config Kube
    2. Helm Upgrade And Check Status

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.13.08_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.13.08_PM.png)

    ### Variable 설정

    `Helm Upgrade And Check Status` Task group에 변수를 설정해주자.

    - artifact-name : CI 파이프라인에서 설정해준 artifact 명을 적어주면된다.
    - helm-directory: 헬름 차트 템플릿이 저장되어 있는 디렉토리명을 적어주면 된다. 이전에 CI 파이프라인에서 우리는 헬름 차트 템플릿만 artifact로 만들었으므로 디렉토리명은 현재 디렉토리를 의미하는 `.` 으로 적어주면 된다.
    - helm-value-file: 헬름 차트의 변수값들을 담고 있는 파일을 지정해준다. 개발 서버와 운영서버를 잘 구분하자.
    - k8s-deployment-name: 쿠버네티스에 배포할 서비스의 deployment 이름을 적어주면된다.
    - k8s-namespace: 쿠버네티스에 배포할 서비스의 네임스페이스 이름을 적어주면 된다.
    - k8s-service-name: 쿠버네티스에 배포할 서비스의 이름을 적어주면 된다.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.16.27_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.16.27_PM.png)

    ### 저장

    Save 버튼을 눌러서 저장하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.23.19_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.23.19_PM.png)

- 운영 Stage 추가

    ### Stage 추가

    Artifacts를 클릭한 후 Add 버튼을 눌러 새로운 스테이지를 추가하자. (Artifacts를 먼저 눌러주지 않으면 Deploy development의 다음 Stage가 생성되니 주의하자.)

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.26.14_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.26.14_PM.png)

    Template 선택 없이 Empty job 링크를 눌러주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.28.53_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.28.53_PM.png)

    ### Tasks 설정

    상단 탭의 Tasks → 새롭게 생성된 스테이지명을 선택하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.31.36_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.31.36_PM.png)

    ### Stage 이름 설정

    원하는 스테이지 이름으로 변경하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.32.11_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.32.11_PM.png)

    ### Agent specification 변경

    Agent를 ubuntu 환경에서 실행하도록 변경하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.10.37_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.10.37_PM.png)

    ### Task 추가

    Agent job 에서 `+` 버튼을 누른 후 `helm` 으로 검색 하여 이전에 추가했던 Task group 들을 추가하자.

    순서는 아래와 같이 추가하면 된다.

    1. Install Helm And Config Kube
    2. Helm Upgrade And Check Status

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.13.08_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.13.08_PM.png)

    ### Variable 설정

    `Helm Upgrade And Check Status` Task group에 변수를 설정해주자.

    - artifact-name : CI 파이프라인에서 설정해준 artifact 명을 적어주면된다.
    - helm-directory: 헬름 차트 템플릿이 저장되어 있는 디렉토리명을 적어주면 된다. 이전에 CI 파이프라인에서 우리는 헬름 차트 템플릿만 artifact로 만들었으므로 디렉토리명은 현재 디렉토리를 의미하는 `.` 으로 적어주면 된다.
    - helm-value-file: 헬름 차트의 변수값들을 담고 있는 파일을 지정해준다. 개발 서버와 운영서버를 잘 구분하자.
    - k8s-deployment-name: 쿠버네티스에 배포할 서비스의 deployment 이름을 적어주면된다.
    - k8s-namespace: 쿠버네티스에 배포할 서비스의 네임스페이스 이름을 적어주면 된다.
    - k8s-service-name: 쿠버네티스에 배포할 서비스의 이름을 적어주면 된다.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.16.27_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.16.27_PM.png)

    ### 승인 절차 추가

    파이프라인 화면에서 `Deploy production` 스테이지의 번개,사람 모양을 선택하면 `Pre-deployment conditions` 화면이 열리고 그 화면에서 Pre-deployment approvals 를 Enable 시킨 후 담당자를 할당해주자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.33.45_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.33.45_PM.png)

    ### 저장

    Save 버튼을 눌러서 저장하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.23.19_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.23.19_PM.png)

## Notification

빌드의 성공 및 배포 완료, 담당자 승인요청을 Slack으로 받아 보기위해서 Service Hooks를 설정해주어야 한다. (해당 설정을 위해서는 관리자 권한이 필요하다.)

- Subscription 생성

    ### Subscription 생성

    Project settings → Service hooks 메뉴로 이동한 후 Create subscription 버튼을 클릭하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.55.03_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.55.03_PM.png)

    ### Slack 서비스 선택

    slack 서비스를 선택한 후 Next 버튼을 누르자

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.56.38_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.56.38_PM.png)

    ### Trigger Type 선택

    원하는 트리거 타입을 선택한 후 Next 버튼을 누르자.

    제공하는 트리거 타입 중 우리가 추가할만한 것은 아래와 같다.

    - Build completed : 빌드 완료 알림
    - Release deployment approval pending: 릴리즈 승인 대기 알림

        해당 타입은 승인 타입을 다시 선택해야 하는데 `Pre-deployment` 를 선택하면 아래와 같이 승인 버튼이 있는 슬렉 알림을 받을 수 있다.

        - 이미지 보기

            ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.01.41_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.01.41_PM.png)

    - Release deployment completed: 릴리즈 완료 알림

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.58.03_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_10.58.03_PM.png)

    ### Slack hook url 설정

    Slack hook url은 아래와 같다. 입력 후 Finish 버튼을 눌러주자

    - Slack hook url

        [https://hooks.slack.com/services/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](https://hooks.slack.com/services/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)

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.03.25_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.03.25_PM.png)

- Slack에서 알림 받기

    원하는 슬렉 채널에서 `/azpipelines subscribe [pipeline url]` 을 입력해서 원하는 알림을 구독하면 된다. 

    별도로 생각한 채널이 없으면 `chap-dev-ci-cd` 채널을 생성해두었으니 여기에서 이용해보자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.09.18_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.09.18_PM.png)

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.09.47_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.09.47_PM.png)

    ### Build 링크 찾는 방법

    Pipelines → Pipelines 메뉴에서 원하는 파이프라인을 선택하자

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.11.01_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.11.01_PM.png)

    브라우저의 링크를 복사하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.12.02_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.12.02_PM.png)

    ### Release 링크 찾는 법

    Pipelines → Release 메뉴로 이동 후 상단의 URL을 복사하자.

    ![/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.13.48_PM.png](/assets/images/draft/class101-posts/azure-pipeline/Screen_Shot_2020-12-07_at_11.13.48_PM.png)

    # Troubleshooting

    ## The system does not have docker-credential-ecr-login CLI

    메시지 그대로 gradle 이미지에 docker-credential-ecr-login 명령어가 없어서 그렇다. 

    아래 내용들을 수행하면 해결된다.

    - build.gradle에 아래와 같은 설정이 되어있다면 제거한다

    ```groovy
    jib {
        to {
            image = 'some.repository.url/image'
    				credHelper = 'ecr-login' // <- 이친구를 제거합시다.
            project.afterEvaluate {
                tags = [version]
            }
        }
    }
    ```

    - jib 버전을 최신화