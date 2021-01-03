---
layout: single
title: "2020년 9월 2주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

이번주에 현재 개발중인 SKU 서비스를 배포하는 것을 목표로 하였는데, 결국 배포 실패하였다. 내손으로 서비스를 처음부터 구성하고 개발하려다 보니 욕심이 앞선거 같다. 더 좋게, 더 나은 방법이 있지 않을까 하는 고민들과 피처와 관계없이 어플리케이션의 디자인과 여러 도구들을 적용해 보는데 정신이 팔렸던 것 같기도 하다. 현재 개발하고 있는 서비스 개발 기간이 길어짐에 따라 심리적으로 일정에 대한 압박이 커서 무리한 일정을 얘기한 것일 수도 있다. 이번일을 계기로 일정관리와 개발 사이 어떻게 조절을 해야하는 지에 대해 많은 것을 배울 수 있었다.

쿠버네티스를 배우기로 마음먹고 현재 쿠버네티스 운영을 담당하고 있는 동료에게 많은 것을 배우고 있다. 한창 재미있게 배우고 있는데 갑자기 동료가 관리자 권한을 주는 것이다! 아직은 관리자 권한을 받기엔 부담스럽다고 하니까 ㅋㅋㅋ 받아야 된다고 하더라.....ㅋㅋㅋㅋㅋ 그때 느낌이 딱 왔지만 역시나 조금있으면 퇴사를 하신단다... 하... 배우는건 재밌었는데 막상 운영해야한다니 막막하다. 뭐 많이 배울 수 있을 테니 좋게 생각하기로 했다.

아래는 이번주에 내가 배웠거나 겪었던 이슈들을 정리한 것이다.

## Kotest 이슈 해결 완료

Kotest에서 Spring boot의 `@DataJpaTest` 사용 시 `FreeSpec`을 이용하여 테스트를 BDD로 작성하는 경우 테스트가 정상적으로 동작하지 않는 이슈를 발견했었다.

이슈 링크 : [https://github.com/kotest/kotest/issues/1643](https://github.com/kotest/kotest/issues/1643){: target="\_blank" }

최근 `4.2.4`버전에서 해당 이슈를 해결하였다는 소식을 전해 들었다. 빨리 버전을 변경해서 적용 테스트를 해봐야 겠다.

## 클릭업 오토메이션

클릭업에 오토메이션이랑 기능이 베타버전으로 제공되고 있다. [https://docs.clickup.com/en/articles/3904901-automations](https://docs.clickup.com/en/articles/3904901-automations){: target="\_blank" }

아직 전부다 보진 않았는데 잘 설정하면 편리한 기능들(예를 들면 특정 조건에 맞게 자동으로 테스크의 상태를 변경해 준다던지)이 많을 것 같다. 다만 좀 아쉬운거는 액션 개수에 따라서 과금이 된다는 것이다. JIRA나 Azure Devops에서 기본적으로 제공해주는 기능을 과금모델로 제공한다니... 이해하기가 힘들다.

## 클릭업 커스텀 Task ID

클릭업에서 새로운 기능으로 [Custom Task ID](https://docs.clickup.com/en/articles/4368464-custom-task-ids){: target="\_blank" }를 제공한다.
기존에 해시값으로 제공하던 Task ID를 좀더 기억하기 쉽게 사용자가 Prefix를 줄 수 있고 자동으로 넘버링을 해주는 방식이다. 정확한 매커니즘은 모르겠는데 기존 ID들도 다 바뀌는것으로 봐서는 마스킹을 하는게 아닐까 하는 생각을 해본다. 아쉬운점이라면 이전에 사용하던 ID까지 다 바껴버리기 때문에 처음부터 적용한게 아니라면 바꾸기 이전의 Task를 추적하기가 쉽지 않을 수도 있을 것 같다 라는 생각이 든다.

## slack 런치 트레인

회사에서 Slack을 이용하여 점심이나 저녁을 같이 먹을 동료를 모집하곤 한다. 어느 동료가 [슬렉 런치 트레인](https://lunchtrain.builtbyslack.com){: target="\_blank" }이라는 것을 사용하는 것을 보았는데 신기하고 재미있었다.

## 시멘틱 버저닝

이번주 백엔드 개발자 발표때 시멘틱 버저닝과 관련한 내용의 발표가 있었다. 솔직히 백엔드 서버 개발을 하면서 별도로 버저닝을 하지 않았었는데, 일단 API가 내부에서만 사용되기 때문이고, 버저닝을 할 필요성을 느끼지 못하고 있기 때문이다. 회사 차원에서 버저닝을 하자고 정책이 결정되지 않는 한 사용하지 않을 것 같지만 그래도 알아두면 좋으니 관련 링크를 달아둔다.

https://semver.org/lang/ko/

아래 링크는 치트시트 이다.

https://devhints.io/semver

## 벤치마킹 툴

이전에 부하 테스트를 위한 툴로 (이런 툴을 벤치마킹 툴이라고 하더라) [Ngrinder](http://naver.github.io/ngrinder/){: target="\_blank" }나 [Jmeter](https://jmeter.apache.org/){: target="\_blank" }를 사용해 본적이 있다. Ngrinder는 대시보드 및 관리 서버를 두고 Agent를 심어서 부하테스트를 하는 방식이고 Jmeter는 로컬에서 테스크를 실행시켜서 부하테스트를 하는 방식이다.

회사 동료가 [WRK](https://github.com/wg/wrk){: target="\_blank" }라는 툴을 소개해 주었는데, 콘솔에서 실행하는 방식이긴 하지만 가볍게 테스트 해볼 수 있다는 점에서 좋아 보인다.

## 크롬 확장 다크모드

크롬브라우저에서 테마로 블랙 테마를 사용하고 있는데 확장프로그램으로 다크모드를 제공하는 프로그램이 있다는 것을 알게되어 적용해서 사용중이다.

이름은 [Dark Reader](https://chrome.google.com/webstore/detail/dark-reader/eimadpbcbfnmbkopoojfekhnkhdbieeh){: target="\_blank" } 이다.

## git pre-commit ignore

Git 이용시 코드 컨벤션을 맞추기 위해서 pre-commit을 사용하고 있다. 하지만 잠시 테스트를 위해서 임시로 커밋을 해야할 일이 있었는데 계속 pre-commit에서 컨벤션 오류로 인해 commit이 되지 않는 일이 있었다. 이럴때 사용하면 좋은 옵션이 바로 `--no-verify`이다.

사용방법은 아래와 같다.

```
git commit --no-verify
```

짧게 `-n`으로도 사용이 가능하다

```
git commit -n
```

편리한 기능이지만 특정 상황이 아니면 사용하지 말자 ^^;

## 환율 관리 API

환률 정보를 제공하는 API를 발견해서 링크를 적어본다. 요청횟수에 따라 과금되는 구조이기 때문에 실시간으로 환율이 필요한 경우가 아니라면 환율정보를 일별 배치로 DB화 해두고 사용하는 방법도 좋아보인다.

API 링크 : [https://fixer.io/](https://fixer.io/){: target="\_blank" }

## @PostConstruc 이슈

`@PostConstruc`을 이용하여 Application의 기본 타임존을 설정해서 사용하려고 했었다. 하지만 동료분께서 해당 어노테이션을 이용하여 서버를 기동할 시 간헐적으로 타임존이 제대로 설정이 되지 않는다는 얘기를 해주셨다. 그 이유는 `PostConstruc`가 빈이 모두 로드 된 후 적용되는데 그 시점이 비동기적이어서 어플리케이션에서 타임존을 값을 불러오는 시점에 따라 원하지 않는 타임존을 가져올 수도 있다는 것이었다. 그래서 타임존 설정을 메인 메소드 바로 위에 기입해서 동기적으로 적용되도록 바꾸었다.

```
@SpringBootApplication
class Application

fun main(args: Array<String>) {
    TimeZone.setDefault(TimeZone.getTimeZone(ZONE_ID_ASIA_SEOUL))
    runApplication<Application>(*args)
}
```

## jacoco kotlin inlin function 이슈

현재 테스트 커버리지 검증 도구로 Jacoco를 사용하고 있는데 kotlin에서 inline function을 사용하고 있는데 해당 코드에 대한 테스트 코드를 작성하였음에도 커버리지가 올라가지 않아 확인해보니 검증할 수 없는 이슈가 있었다.

[https://github.com/jacoco/jacoco/issues/654](https://github.com/jacoco/jacoco/issues/654){: target="\_blank" }

그래서 해당 코드들은 모두 제외 처리하였다.
