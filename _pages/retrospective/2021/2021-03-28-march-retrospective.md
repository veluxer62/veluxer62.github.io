---
layout: single
title: "2021년 3월 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2021-march-retrospective/
---

8일 부터 새로운 회사에 출근을 하게 되었다. 설레임반 걱정반의 심정으로 첫 출근을 하였다. 이전 회사와는 다른 절제된 분위기 속에서 구성원들간의 존중이 느껴졌다. 앞으로 화이팅이다.

새로운 회사의 백엔드의 주된언어는 python이다. 이전에 스파르타코딩클럽의 튜터로 활동하면서 python + flask를 사용해 본적이 있어서 간단한 서버정도는 만들어 본적이 있다. 하지만 python을 깊이 사용해 본적이 없고 생태계를 접해본 경험이 부족해서 앞으로 많이 헤매야 하지 않을까 싶다.

현재 팀의 개발 문화를 보면서 가장 놀라웠던 부분이 테스트 코드와 코드 리뷰 그리고 앞의 두개에 대한 비개발직군의 공감대였다. <br/>
 길지않은 경험이지만 지금까지 재직중이었거나 들었던 회사에서 100% 테스트 코드를 심지어 유닛테스트, 기능테스트, E2E 테스트, QA 프로세스까지 존재하는 회사를 보질 못했다. 누구나 알법한 대기업이나 유명 스타트업에서도 테스트 코드를 작성한다고는 하지만 이정도 일거라 않으리라 생각한다. 이러한 코드베이스는 앞으로 제품을 발전시켜갈 원동력이 되리라 생각이 든다. (실제로 한달 가까이 있으면서 1번 정도 장애 상황을 경험하였고 이마저도 예상치 못한 제품계획의 문제였지 코드에서의 결함이 원인은 아니었다.)<br/>
 두번째는 코드리뷰인데 꼼꼼한 코드리뷰는 python을 잘 모르는 나에게 더 나은 코드퀄리티를 유지할 수 있도록 도움을 주었다. 코드리뷰에 공을 많이 들이다보면 리뷰이나 리뷰어 모두 공수를 들여야하고 지치기 마련이다. 하지만 구성원 모두 더 나은 리뷰환경을 구성하기 위해 고민하고 노력하고 있는 모습을 보며 더 나은 제품을 만들 수 있을 것이라는 자신감을 가질 수 있었다. <br/>
 마지막으로 테스트코드와 리뷰에 대한 비개발직군의 공감대였다. 테스트코드와 리뷰는 개발자들의 책임이자 의무이다. 언뜻 보기에 이 행위들은 비개발자 입장에서는 비생산적인 활동이라 느껴질 수 있다. (실제로도 그렇게 느끼고 있거나 그것에 대해 말하는 동료들을 본적이 있다. 심지어 개발자까지도) 하지만 장기적인 관점에서 앞의 활동들은 오히려 개발 속도를 더 낼 수 있는 원동력이 될 수 있는데, 현재 팀의 구성원들은 모두다 테스트코드와 리뷰에 대해 공감하고 스프린트의 공수에 포함시켜가며 그 활동을 이어나갈 수 있도록 지지해 주고 있다.

앞으로 맡게된 프로젝트를 보면서 변경했으면 하는 것들을 대충 정리하였고 팀원들에게 공유하며 앞으로 제품을 어떻게 만들어가면 좋을지 논의하기로 하였다. 마음 같아서는 kotlin + spring을 쓰고 싶은 욕심이 있지만 기존의 코드를 새롭게 바꿔가겠다는 결정은 100%의 자신감이 없다면 하지 않기로 하였다. 대신에 도메인을 정리하고 기존 코드를 점진적으로 개선해 가면서 우리가 제공하는 서비스를 대부분을 이해하는 것을 첫번째 목표로 하였다.

아래는 3월동안 내가 배우거나 겪었던 이슈에 대해서 정리한 내용이다.

## Creator KPI

CTO님께서 제품팀의 핵심 지표에 대해 작성해주신 글이 있는데 인상깊은 부분이 있어서 남긴다.

> 제품팀이라면 '매출을 달성하지 못하겠네'같은 고민이 아닌 '적립율이 안올라가네', '명세표 업로드 수가 떨어지네'와 같은 고민을 해야 합니다.

제품팀의 핵심 지표를 매출이라는 것에 끼워맞추기 보다 더 좋은 제품을 만들기 위한 목표를 가져야 한다는 것으로 해석된다. 정반대 되는 목표를 향해 달려가는 곳에서 느꼇던 답답함을 이 글 하나로 풀 수 있었다.

## 파이썬3 기본버전 설정

Mac OS에서 Homebrew를 이용하여 python의 기본버전을 설정하는 방법을 기억하고자 적어본다.

### install python
```bash
brew update && brew install python3
```

### set alias
아래 스크립트를 zsh를 사용한다면 `~/.zshrc`에 bash를 사용한다면 `~/.bashrc`에 추가한다.
```bash
alias python=/usr/local/bin/python3
```

### apply & check
zsh를 사용한다면 `~/.zshrc`에 bash를 사용한다면 `~/.bashrc`에 추가한 스크립트를 적용 후 파이썬 버전을 확인한다.
```bash
source ~/.zshrc && python --version
```

참고 사이트 : [https://nono.ma/python-3-default-macos](https://nono.ma/python-3-default-macos){: target="\_blank" }

## Graphql Client Tool

이전에 Postman을 Graphql client tool로 사용했었는데 이번에 [AltairGraphQL](https://altair.sirmuel.design/){: target="\_blank" }이라는 도구를 알게 되었다!! GraphQL에 특화된 도구이다 보니 사용성이 무척 좋았다.

## Gather

회사에서 워크샵을 위해서 [Gather](https://gather.town/){: target="\_blank" }를 사용해 보았다. 포켓몬스터 게임과 같은 UI에서 케릭터를 움직일 수 있고 케릭터끼리 가까이가면 화상으로 대화를 할 수 있는 시스템인데 재밌으면서 활용성도 있어서 좋았다. 다만 속도나 네트워크 환경 부분은 개선이 필요해 보였다..20명 가까이 모여서 얘기를 하니 나중에 조금 끊기는 현상이 발생했다.

## Python 기초 공부 사이트

파이썬을 공부하기 위해 이것저것 알아보던 중 '점프 투 파이썬'을 웹사이트에서 무료로 볼 수 있는 페이지가 있어서 적어둔다. [https://wikidocs.net/book/1](https://wikidocs.net/book/1){: target="\_blank" }

## SSM을 로컬에서 사용하기

내 PC에서 현재 운영중인 AWS의 사설망 내의 서버로 접근하기 위해서는 다양한 방법이 있다. 이전엔 VPN을 주로 사용하거나 Bastion 서버를 따로 두어서 접근하는 방식을 사용했었는데 현재 회사에서는 SSM(AWS Systems Manager)을 이용하고 있다. VPN이나 Bastion서버를 두는 경우 별도로 서버를 구성하거나 AWS의 것을 사용해야 하는데 비용이 발생하고 관리를 따로 해주어야 하지만 SSM의 경우 AWS 계정을 이용하여 손쉽게 이용할 수 있다는 점에서 현재까지 사설망 서버로 접근하는 방법 중 가잔 괜찮은 방법이지 않나 생각된다.

SSM을 사용하기 위해서는 AWS Console에서 접속하거나 CLI를 이용하는 방법이 있는데 AWS Console은 번거로우니 로컬에서 CLI를 이용하여 SSM을 사용하는 방법에 대해서 적어본다.

### Install

AWS CLI 설치

```bash
brew install awscli
```

AWS Session Manager Plugin 설치

```bash
brew tap dkanejs/aws-session-manager-plugin
brew install aws-session-manager-plugin
```

### Connect

```bash
aws ssm start-session --target {Instance-ID}
```

참고로 ssm을 이용하여 portforwarding을 사용해 보려고 하였으나 보안상의 문제인지 ssm의 local port에 대해서는 portforwarding을 제공하지만 원격 서버에 대해서는 제공하지 않는다...이부분은 좀 아쉽다.

참고 사이트: [https://musma.github.io/2019/11/29/about-aws-ssm.html#ssm%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B0%A9%EB%B2%95%EA%B0%9C%EC%84%A0](https://musma.github.io/2019/11/29/about-aws-ssm.html#ssm%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B0%A9%EB%B2%95%EA%B0%9C%EC%84%A0){: target="\_blank" }

## 한국의 행정구역 영어 표기법

최근에 주소관련 모델을 정의하면서 한국의 행정구역에 대한 영어 표기법을 어떻게 하면 좋을지 찾아보았고 아래와 같이 적으면 좋을 것 같아서 정리해본다.

### 한국 행정계층

시도 > 시군구 > 읍면동

### 미국 행정계층

Country > Municipality/City,Town,Village

### 한국과 미국 비교

| Region | State | Country, District | City | Town | Township | Village |
|:------:|:-----:|:-----------------:|:----:|:-----:|:-------:|:-------:|
| 지방    | 도     | 특별, 광역시         | 시   | 군, 구 | 동, 읍   | 면, 리   |

## 클라우드 네이티브 환경을 한눈에 볼 수 있는 사이트

클라우드 네이티브 환경에 대해 한눈에 파악할 수 있는 사이트가 있어서 공유해본다.

[CNCF Cloud Native Interactive Landscape](https://landscape.cncf.io/){: target="\_blank" }

## 컬러 팔레트

이번에 워크샵을 진행하면서 디자이너분께서 자사 서비스에 대한 컬러팔레트에 대한 개선방안을 설명해 주셨다. 디자이너와 함께 일할일이 적어서 [컬러팔레트](https://www.netinbag.com/ko/internet/what-is-a-color-pallet.html){: target="\_blank" }라는 것이 있는지 처음 알았다. 색상에 따라 의미를 부여한다는 것도 참 신선하게 다가왔다. 백엔드 개발자라 이런걸 신경을 거의 신경쓸일이 없었다. 그나마 와닿았던건 백오피스 개발을 하면서 [Bootstrap](https://getbootstrap.com/){: target="\_blank" }을 사용했었는데 여기서 색상별 클래스명을 primary, success, warning, danger 등과 같이 사용한 경험이 있어서 였다.

아무튼 색상에 대해서도 의미를 부여할 수 있고 컬러팔레트로 관리할 수 있다는 걸 알게 되었다.

## 부하 테스트 도구

부하테스트 도구로 많은 도구가 있다. 내가 알고 있는 도구로 nGrinder, jMeter 정도가 있다. 최근에 k6에 대해서 알게 되었는데 이것도 테스트 결과에 대한 대시보드를 제공해 주어서 사용해봄직 한것 같다.

공식 페이지: [https://k6.io/docs/](https://k6.io/docs/){: target="\_blank" }

관련 블로그: [https://medium.com/swlh/beautiful-load-testing-with-k6-and-docker-compose-4454edb3a2e3](https://medium.com/swlh/beautiful-load-testing-with-k6-and-docker-compose-4454edb3a2e3){: target="\_blank" }
