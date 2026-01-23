---
layout: single
title: "제품 개발을 위한 JIRA 사용 가이드"
categories:
  - TUTORIALS
tags:
  - JIRA
  - Agile
  - Sprint
  - Kanban
  - Scrum

toc: true
toc_sticky: true
excerpt: 이 글은 현재 몸담고 있는 팀에서 제품을 개발할때 JIRA를 활용하는 사례를 소개하고 이를 통해 JIRA를 활용하는 방법을 소개하기 위한 글이다. JIRA의 사용방법은 공식 페이지에 좀더 자세히 나와있으니 참고하고 이 글에서는 사용 시나리오를 기반으로 어떻게 JIRA를 활용하여 제품을 만들어가는지를 봐주면 좋겠다.
---

## Intro

이 글은 현재 몸담고 있는 팀에서 제품을 개발할때 JIRA를 활용하는 사례를 소개하고 이를 통해 JIRA를 활용하는 방법을 소개하기 위한 글이다. JIRA의 사용방법은 [공식 페이지](https://www.atlassian.com/software/jira){: target="\_blank" }에 좀더 자세히 나와있으니 참고하고 이 글에서는 사용 시나리오를 기반으로 어떻게 JIRA를 활용하여 제품을 만들어가는지를 봐주면 좋겠다.

## 프로젝트 운영방식 선정 배경

우리팀은 애자일 소프트웨어 개발 방법론에서 스크럼과 칸반을 혼합한 형태로 제품을 개발하고 있다.

스크럼은 스프린트라고 하는 단기 작업 블록을 통해서 프로젝트를 진행하며 스프린트 기간은 보통 2~4주 사이로 하고 있지만 우리팀은 2주를 채택하고 있다. 스크럼의 가장 큰 특징은 스프린트 기간내에 계획된 작업 외에 다른 작업들은 진행중인 스프린트에 포함시키지 않고 오로지 계획된 이슈들만 작업을 진행한다는 것이다.

칸반은 백로그에 필요한 작업들을 쌓아두고 작업자가 작업할 수 있을 때 우선순위가 높은 작업을 할당하여 작업을 진행하는 방법이다. 칸반의 가장 큰 특징은 작업의 우선순위가 높은 새로운 작업이 발생하는 경우(버그, 급한 운영 이슈 등) 진행 중이던 작업을 잠시 멈추고 우선순위가 높은 작업을 먼저 진행할 수 있다는 것이다.

우리팀이 스크럼과 칸반을 혼합한 이유는 스크럼과 칸반의 장점을 모두 가져가기 위해서이다. 앞서 스크럼의 큰 특징이 스프린트 내 계획되지 않은 작업은 추가하지 않는 것이다. 이는 정확한 목표 수립과 정확한 공수측정을 위해서 인데, 운영중인 서비스가 있어 계획되지 않은 작업이 자주 발생하거나 명확한 목표를 정하기 어려운 상황에서는 이상적으로 스크럼방식을 수행하기 어렵다. 그래서 칸반처럼 계획되지 않은 작업도 스프린트에 포함 시키면서 목표한 작업을 계획해서 스프린트를 운영하는 혼합된 방식을 채택하게 되었다.

## 용어

사례를 보여주기 전에 먼저 기본적인 용어를 정리해보자.

### 프로젝트(Project)

프로젝트는 말그대로 JIRA에서 프로젝트 단위로 이슈를 관리하는 공간이다. 각 프로젝트마다 독립된 설정값을 따르며 이슈카드의 ID도 다른 형태로 생성된다. 회사마다 다르겠지만 우리는 제품단위로 나뉘기도 하고 제품개발이 아닌 업무별로 나누어서 관리하기도 한다.

![project](/assets/images/posts/how-to-use-jira/project.png)

### 보드(Board)

프로젝트내에 여러개의 보드를 운영할 수 있다. JIRA에서는 스크럼과 칸반 2개의 보드 스타일을 제공한다.

![board](/assets/images/posts/how-to-use-jira/board.png)

#### 칸반

칸반형태로 보드를 생성한다면 아래 그림과 같이 메뉴에 Kanban board가 나타나게 되며 칸반형태로 이슈를 관리할 수 있도록 한다.

![kanban](/assets/images/posts/how-to-use-jira/kanban.png)

칸반보드를 보면 생성된 이슈들을 칸반보드 형태로 나타나 있는 것을 볼 수 있다.

![kanban-board](/assets/images/posts/how-to-use-jira/kanban-board.png)

#### 스크럼

스크럼 형태로 보드를 생성한다면 아래 그림과 같이 메뉴에 `Backlog`와 `Active sprints`가 나타나게 된다.

![scrum](/assets/images/posts/how-to-use-jira/scrum.png)

##### Backlog

백로그는 보드 내 현재 생성되어 있는 모든 이슈카드 목록을 보여준다. 현재 진행중인 스프린트와 앞으로 진행할 스프린트 그리고 백로그들을 보여주기 때문에 다음 스프린트 계획 시 활용하면 좋다.

![backlog](/assets/images/posts/how-to-use-jira/backlog.png)

##### Active sprints

Active sprints는 말그대로 현재 진행중인 스프린트내 이슈들만 볼 수 있는 화면이다. 백로그와 달리 칸반형태로 표시하기 때문에 각 이슈의 진행상황을 파악하기에 용이하다.

![active-sprint](/assets/images/posts/how-to-use-jira/active-sprint.png)

### 이슈(Issue)

보드 내 생성된 작업들을 이슈라고 한다. 이슈는 여러가지 타입이 있으며 우리팀에서는 크게 Epic, Story, Task, Bug, Sub-Task를 기본적으로 사용한다.

#### 에픽(Epic)

에픽이란 여러번의 스프린트를 거쳐 완료되는 정도의 작업량을 가진 업무를 말한다. 에픽의 범위는 회사마다 이슈관리 도구마다 다르게 정의할 수 있지만 우리팀에서는 스토리들을 묶은 상위 개념의 기능을 의미한다. 에픽에서는 기능에 대한 정의만하지 상세한 기술은 스토리를 통해 설명한다.

![epic](/assets/images/posts/how-to-use-jira/epic.png)

#### 스토리(Story)

스토리는 비지니스 가치를 제공하는 최소 단위의 요구사항을 말한다. 보통 사용자 입장에서 필요한 내용을 일상적인 언어로 기술한다. 하나의 에픽에 여러개의 스토리가 있을 수 있다.

![story](/assets/images/posts/how-to-use-jira/story.png)

#### 태스크(Task)

태스크는 하나의 스토리를 완성하기 위한 구체적인 작업들을 나타낸다. 하나의 스토리에 여러개의 태스크가 있을 수 있다. 태스트 구성요소는 아래와 같다.

- 디자인
- 기술 검토
- 개발
- 문서화

![task](/assets/images/posts/how-to-use-jira/task.png)

#### 버그(Bug)

버그는 주로 QA 시 요구사항을 충족하지 못하는 경우나 운영 중 발생한 이슈를 리포팅하기 위한 용도로 사용된다.

![bug](/assets/images/posts/how-to-use-jira/bug.png)

#### 서브 태스크(Sub-task)

서브 태스크는 하나의 태스크를 여러개의 세부작업으로 나눌 필요가 있는 경우 태스크의 하위 작업의 용도로 사용한다.

![sub-task](/assets/images/posts/how-to-use-jira/sub-task.png)

#### 컴포넌트(Component)

컴포넌트는 에픽 또는 이슈들을 묶어주는 단위로써 해당 이슈의 목표점 또는 특징을 나타낸다.

![component](/assets/images/posts/how-to-use-jira/component.png)

## 시나리오 1 : 스프린트 관리

이제 스프린트를 진행하는 시나리오를 기반으로 Jira의 프로젝트 관리 방법을 알아보자.

### 스프린트 계획

스프린트 계획은 스프린트를 진행하기 전에 진행할 다음 스프린트 작업을 계획하는 활동이다. 스프린트를 생성하고 기획문서를 바탕으로 관련된 이슈들을 생성하거나 이전에 수행하지 못했던 이슈들을 다음 스프린트로 이동하는 등의 작업을 진행한다.

#### 스프린트 생성

`Backlog`메뉴에서 `Create sprint`버튼을 클릭하여 스프린트를 생성한다.

![create-sprint](/assets/images/posts/how-to-use-jira/create-sprint.png)

#### 스프린트 설정

생성된 스프린트의 이름과 소요일정을 설정한다.

![edit-sprint](/assets/images/posts/how-to-use-jira/edit-sprint.png)

#### 에픽 생성

PO/PM은 기획문서를 기반으로 신규 또는 변경되는 기능에 대한 에픽을 생성힌다. (컨플루언스 문서에서 에픽을 링크해두면 이슈카드에서도 컨플루언스 문서 링크를 함께 확인할 수 있다.)

![epic-doc](/assets/images/posts/how-to-use-jira/epic-doc.png)

![epic-link](/assets/images/posts/how-to-use-jira/epic-link.png)

에픽은 Jira 상단의 `Create`버튼이나 `Backlog`화면 내 EPICS의 `Create epic`버튼을 통해 생성할 수 있다.

![epic-creation-button](/assets/images/posts/how-to-use-jira/epic-creation-button.png)

![create-epic](/assets/images/posts/how-to-use-jira/create-epic.png)

#### 에픽 설정

생성된 에픽의 상세화면으로 이동하여 다음과 같이 설정을 추가해준다.

##### 라벨 설정

각 이슈들은 프로젝트내에서는 공유된다. 그래서 별도의 설정을 하지 않으면 보드내 모든 이슈들이 보이는 문제가 있다. 그래서 특정 보드에 원하는 이슈들만 보이도록 하기 위해서 라벨을 설정한다. 아래는 팀에서 현재 운영중인 라벨 목록이다. (라벨은 하나 이상 설정이 가능하다.)

- 코어 스쿼드: `core-squad`
- 거래처 스쿼드: `tf-vendor-list`

![label](/assets/images/posts/how-to-use-jira/label.png)

##### 컴포넌트 설정

앞서 설명한 바와 같이 컴포넌트는 해당 에픽이 가진 특징을 표현한다. 컴포넌트 설정을 해두면 같은 특징을 가진 작업들을 한번에 볼 수 있기 때문에 설정해 두면 향후 큰 도움이 될 수 있다. (컴포넌트는 하나 이상 설정이 가능하다.)

![set-component](/assets/images/posts/how-to-use-jira/set-component.png)

##### 담당자 할당

주로 에픽은 PO/PM이 관리하기에 담당자는 PO/PM으로 할당한다.

![asignee](/assets/images/posts/how-to-use-jira/asignee.png)

#### 스토리 생성

PO/PM은 기획문서를 기반으로 신규 또는 변경되는 기능에 대한 에픽의 하위 스토리들을 생성한다. (컨플루언스 문서에서 스토리를 링크해두면 이슈 카드에서도 컨플루언스 문서 링크를 함께 확인할 수 있다.)

![story-doc](/assets/images/posts/how-to-use-jira/story-doc.png)

![story-link](/assets/images/posts/how-to-use-jira/story-link.png)

스토리는 Jira 상단의 `Create`버튼이나 에픽내 `Create issue in epic`버튼을 통해 생성한다. (개인적으로는 에픽 내에서 생성하면 자동으로 에픽과 연결을 시켜주기 때문에 에픽 내에서 생성하는 것을 추천한다.)

![story-creation-button](/assets/images/posts/how-to-use-jira/story-creation-button.png)

![create-story-in-epic](/assets/images/posts/how-to-use-jira/create-story-in-epic.png)

만약 Jira 상단의 `Create`버튼을 통해 스토리를 생성한 경우 Epic을 연결시켜주어야 한다.

![story-link-epic](/assets/images/posts/how-to-use-jira/story-link-epic.png)

#### 스토리 설정

생성된 스토리의 상세화면으로 이동하여 다음과 같이 설정을 추가해준다.

##### 라벨 설정

에픽에서 설정했던 방식과 동일하게 라벨을 설정해준다.

![label](/assets/images/posts/how-to-use-jira/label.png)

##### 담당자 할당

스토리도 에픽과 마찬가지로 PO/PM이 관리하기에 담당자는 PO/PM으로 할당한다.

![asignee](/assets/images/posts/how-to-use-jira/asignee.png)

##### 스프린트 설정

스프린트 설정은 백로그 화면에서 백로그 이슈를 해당 스프린트로 드래그 앤 드랍으로 이동하거나 스토리 상세 페이지에서 스프린트를 설정하는 방법으로 설정할 수 있다.

스프린트 설정을 하지 않으면 Active sprints메뉴에서 이슈를 확인할 수 없으니 수행해야할 스프린트에 맞도록 스프린트 설정을 반드시 해주어야 한다.

![set-sprint](/assets/images/posts/how-to-use-jira/set-sprint.png)

##### 스토리 포인트 설정

사실 스토리포인트를 설정해주는것이 이상적이긴 하지만 앞서 팀에서는 이상적인 스프린트를 진행하기 어려운 상황이고 팀원들이 스토리 포인트에 대한 견해차이가 있어 따로 설정하지 않고 운영하고 있다.

#### 태스크 생성

작업자는 스토리를 기반으로 작업할 태스크를 생성한다.

태스크는 Jira 상단의 `Create`버튼이나 에픽내 `Create issue in epic`버튼을 통해 생성한다. (스토리와 마찬가지로 에픽 내에서 생성하는 것을 추천한다.)

![story-creation-button](/assets/images/posts/how-to-use-jira/story-creation-button.png)

![create-task-in-epic](/assets/images/posts/how-to-use-jira/create-task-in-epic.png)

만약 Jira 상단의 Create버튼을 통해 스토리를 생성한 경우 Eplic을 연결시켜주어야 한다.

![story-link-epic](/assets/images/posts/how-to-use-jira/story-link-epic.png)

만약 에픽이 없는 태스크인 경우 별도로 에픽 링크를 설정하지 않고 생성한다. 보통 에픽이 없는 태스트인 경우는 스프린트에 포함되지 않는 운영 이슈인 경우가 많다.

#### 태스크 설정

생성된 태스크의 상세화면으로 이동하여 다음과 같이 설정을 추가해준다.

##### 스토리 링크 설정

생성된 태스크와 관련한 스토리를 링크한다. (에픽이 없거나 관련된 스토리가 없는 태스크의 경우 링크하지 않는다.)

Jira에서는 스토리와 Task가 동일한 작업레벨을 가진다. (서브 태스크가 있긴 하지만 서브 태스크를 이용하여 작업관리를 하는 경우 작업 Estimate에 대한 리포팅이 제대로 되지 않는 등 불편한 점들이 많아 태스크로 작업 관리를 하는 것을 추천한다.)

그래서 태스크가 특정 스토리와 관련이 있다는 것을 알 수 있도록 하기 위해서 링크를 걸어준다. 링크를 걸어주는 작업이 다소 번거로울 수 있지만 이슈 추적에 매우 도움이 되니 링크를 걸어주는 것을 추천한다.

![task-link-story1](/assets/images/posts/how-to-use-jira/task-link-story1.png)

![task-link-story2](/assets/images/posts/how-to-use-jira/task-link-story2.png)

##### 라벨 설정

에픽에서 설정했던 방식과 동일하게 라벨을 설정해준다.

![label](/assets/images/posts/how-to-use-jira/label.png)

##### 담당자 할당

태스크의 담당자는 주로 해당 작업을 수행할 본인 또는 실무자를 할당한다.

![asignee](/assets/images/posts/how-to-use-jira/asignee.png)

##### 스프린트 설정

스프린트 설정은 백로그 화면에서 백로그 이슈를 해당 스프린트로 드래그 앤 드랍으로 이동하거나 스토리 상세 페이지에서 스프린트를 설정하는 방법으로 설정할 수 있다.

스프린트 설정을 하지 않으면 `Active sprints`메뉴에서 이슈를 확인할 수 없으니 수행해야할 스프린트에 맞도록 스프린트 설정을 반드시 해주어야 한다. (스프린트 이슈가 아닌 운영 이슈의 경우 스프린트를 설정하지 않는다.)

![set-sprint](/assets/images/posts/how-to-use-jira/set-sprint.png)

##### Estimate 설정

Task는 스토리포인트를 설정하지 않고 `Original estimate`를 설정하여 예상되는 작업 일정을 기입한다.

![estimate](/assets/images/posts/how-to-use-jira/estimate.png)

### 스프린트 진행

계획된 스프린트를 수행하는 단계이다. 만약 계획되지 않은 태스크가 추가되면 위의 생성규칙을 통해 생성하고 진행한다.

#### 스프린트 시작

`Backlog`화면에서 시작할 스프린트의 `Start sprint`버튼을 클릭하여 스프린트를 시작한다.

![start-sprint](/assets/images/posts/how-to-use-jira/start-sprint.png)

#### 이슈 상태 변경

이슈가 할당된 담당자들은 본인이 할당된 이슈의 진행여부를 파악하셔 이슈상태를 업데이트 한다.

##### 에픽 상태 변경

에픽은 하위 이슈들이 진행중으로 변경되면 진행중으로 상태를 변경한다. 에픽은 여러 스프린트를 걸쳐 수행될 수 있기 때문에 관련된 이슈들이 모두 종료 상태가 되면 종료상태로 변경해 준다.

에픽이 종료되었다면 `Backlog`메뉴의 `EPICS`에서 해당 에픽을 아래 이미지와 같이 최종적으로 완료처리하여 에픽 목록에 더이상 표시가 되지 않도록 한다.

![mark-as-done](/assets/images/posts/how-to-use-jira/mark-as-done.png)

##### 스토리 상태 변경

스토리는 관련된 태스크가 진행중으로 변경되면 진행중 상태로 변경한다. 그리고 QA에 입고된다면 해결됨 상태로 변경하고 QA가 완료되고 작업이 종료된다면 종료상태로 변경한다.

##### 태스크 상태 변경

태스크는 작업자가 진행할 때 진행중으로 변경한다. 프로그래머의 경우 Pull Request를 통한 코드리뷰 진행 시 리뷰 상태로 변경하고 코드가 병합이 된다면 종료상태로 변경한다. 그 외 작업자들도 상황에 따라 리뷰 상태를 거쳐 종료상태로 변경하는 프로세스를 진행하여도 무방하다.

만약 리포터가 자기 자신 또는 작업 담당자가 아니라 다른 사람이라면 작업이 완료되고 해결됨 상태를 둔 후 리포터에게 완료여부를 요청할 수 있다. 이때 리포터는 작업완료를 확인 한 후 이슈를 종료할 수 있다. 해당 프로세스는 운영이슈에서 주로 사용된다.

#### 버그 이슈 처리

QA 또는 운영 이슈 발생 시 버그 이슈가 생성될 수 있다. 버그 리포트가 생성되면 작업자는 위 태스크 생성방법을 통해서 태스크를 생성하고 버그 이슈를 링크해 준다.

![link-bug](/assets/images/posts/how-to-use-jira/link-bug.png)

그리고 작업이 완료되면 태스크는 종료상태로 두고 버그 이슈는 해결된 상태로 변경한다. (리포터에게 넛지를 주면 더 좋겠다)

리포터는 해결됨이 확인되면 버그 이슈를 종료처리한다.

![bug-comment](/assets/images/posts/how-to-use-jira/bug-comment.png)

### 스프린트 종료

QA가 종료되고 작업된 코드를 배포하면서 진행중인 스프린트를 종료한다. 종료 방법은 `Active sprints`메뉴에서 `Complete sprint`버튼을 눌러 종료한다. (만약 여러개의 스프린트가 진행중이라면 Select box에서 종료하고자 하는 스프린트를 선택한 후 `Complete sprint`버튼을 누른다.)

![complete-sprint](/assets/images/posts/how-to-use-jira/complete-sprint.png)

주의해야할 부분은 스프린트 종료 시 반드시 해당 스프린트 내 모든 이슈(서브 태스크 포함)는 종료 상태여야 한다는 것이다. 만약 종료되지 않은 이슈가 있다면 아래와 같이 경고 문구가 표시되고 종료되지 않는다. 다만 종료되지 않은 이슈들은 다음 스프린트나 백로그로 이동시킬 수 있다.

![move-backlog](/assets/images/posts/how-to-use-jira/move-backlog.png)

### 리포트

스프린트가 종료되고 난 후 Reports메뉴를 통해 지난 스프린드 진행 결과를 분석할 수 있다. 이슈 생성 시 Estimate를 잘적어주고 카드의 상태관리를 제때 해주었다면 의미있는 보고서 결과를 얻을 수 있다. 모든 보고서를 다 이해하고 있지 않기 때문에 다루면 좋을 법한 보고서 들만 적어보겠다.

아래에 보여주는 차트들은 동일한 스프린트를 다른 보고서 형태로 표시한 것이다.

#### Burndown Chart

스프린트 기간동안에 계획된 작업 잔여량이 시간이 지남에 따라 어떻게 변화하는지 보여주는 차트이다. 목표 기간내에 작업이 완료되는 추세를 보여주기 때문에 계획된 작업에 대한 완료 여부를 예측하는 용도로 유용하다.

스프린트 기간 내 딱 정해진 작업만을 수행하는 이상적인 스크럼 환경에서 참고하면 유용한 보고서이다.

우리팀에서는 칸반과 스크럼이 혼합된 형태이기 때문에 아래 번다운 차트에서 잔여량이 우하향만 하는 것이아니라 증가하는 추이를 함께 볼 수 있다. 그래서 우리팀에서 참고하기에는 큰 인사이트를 얻기 힘들듯 하다.

![burndown-chart](/assets/images/posts/how-to-use-jira/burndown-chart.png)

#### Burnup Chart

정해진 기간동안 새롭게 생성된 이슈들과 완료된 이슈들의 총량이 시간이 지남에 따라 어떻게 변화하는지 보여주는 차트이다. 목표 기간내에 작업이 완료되는 추세를 볼 수 있다는 점에서는 번다운 차트와 비슷하지만 새롭게 생성되는 이슈들에 대한 추이도 함께 볼 수 있다는 점에서 번다운 차트와 조금 다른 인사이트를 제공한다. 이 점으로 인해 진행중인 스프린트 기간내에 작업의 완료 여부를 예측함과 동시에 새롭게 생성되는 이슈의 추세를 봄으로써 스프린트 진행의 방해요소들을 파악할 수 있는 장점이 있다.

우리팀과 같이 스크럼과 칸반을 혼합해서 프로젝트를 수행하는 경우 참고하면 좋을 보고서 이다.

![burnup-chart](/assets/images/posts/how-to-use-jira/burnup-chart.png)

#### Velocity Chart

각 스프린트에서 예상한 소요시간과 실제 수행완료한 소요시간에 대한 비교결과를 보여준다. 각 스프린트가 진행됨에 따라 이전 스프린트에서 발생했던 문제점에 대한 개선여부를 파악하기에 용의하다.

![velocity-chart](/assets/images/posts/how-to-use-jira/velocity-chart.png)

## 시나리오 2 : 운영 이슈 관리

운영 관리 관점에서 Jira의 이슈 관리 방법을 알아보자.

### 운영 이슈 생성

운영이슈는 운영이슈를 처리하기 위한 칸반 보드에서 생성한다. 별다른 생성공간이 없으므로 Jira 상단의 `Create`버튼을 통해 생성한다.

![create-issue-in-kanban](/assets/images/posts/how-to-use-jira/create-issue-in-kanban.png)

생성 시 특별한 사유가 없으면 이슈 유형을 `태스크`로 설정한다.

![create-task-in-kanban](/assets/images/posts/how-to-use-jira/create-task-in-kanban.png)

작업할 담당자를 할당해 준 후 라벨과 컴포넌트를 설정한다.

![set-label-component](/assets/images/posts/how-to-use-jira/set-label-component.png)

요청 내용은 작업자가 작업내용을 쉽게 파악할 수 있도록 되도록 자세히 적어주시면 좋다.

![task-description](/assets/images/posts/how-to-use-jira/task-description.png)

### 운영 이슈 설정

#### Estimate 설정

Task는 스토리포인트를 설정하지 않고 `Original estimate`를 설정하여 예상되는 작업 일정을 기입한다.

![estimate](/assets/images/posts/how-to-use-jira/estimate.png)

### 상태 관리

작업자는 할당된 작업을 진행할 때 작업을 진행중으로 변경한다. 작업이 완료되면 해결됨 상태로 변경하고 리포터에게 작업완료를 알린다. 리포터는 작업 완료려부를 확인한 후 해당 이슈를 종료처리한다.

## Wrap up

지금까지 두개의 사례를 통해 JIRA를 이용하여 스크럼과 칸반을 운영하는 방법을 알아보았다. 회사마다 팀마다 제품을 만들어가는 방법은 다를 것이기 때문에 앞서 소개한 방법이 이상적이다라고 말할 순 없다. 다만, 위 시나리오별로 도입을 해보고 운영해보면서 자신의 팀에 맞는 JIRA 활용법을 익히기 위한 가이드라인으로써 도움은 되리라 생각된다.
