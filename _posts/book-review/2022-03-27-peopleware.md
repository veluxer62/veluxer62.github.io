---
layout: single
title: "Peopleware"
categories:
  - BOOK-REVIEW
tags:
  - "톰 드마르코"
  - "티모시 리스터"
  - "박재호"
  - "이해영"
  - "3판"
  - "인사이트"
toc: true
toc_sticky: true
---

[![피플웨어](/assets/images/books/peopleware.jpeg)](http://image.yes24.com/momo/TopCate385/MidCate002/38416928.jpg){: target="\_blank" }

리펙터링을 다 읽고 난 후 다음에 읽을 책을 찾기 위해 서점에 들렀다가 눈에 띄여서 구매해 보았다. 사실 이전엔 눈에 띄지 않다가 이제서야 눈에 띈 이유는 현재 백엔드 챕터 리드를 맡고 있는데 팀원간의 관계와 좀더 나은 개발 문화, 환경을 조성할 수 있는 방법을 조금씩 알아가고 싶었기 때문이었다.

피플웨어는 개발자들 사이에서 그리 유명한 책은 아닌 듯하다.(~~내가 몰랐을 수도 있다.~~) 그래서 이 책이 3판까지 나왔다는 사실에 조금 놀래긴 했다. 이 책을 다 읽고나서 왜 3판까지 나왔는지 이해할 수 있었는데, 2014년에 초판이 발행 되었음에도 불구 하고 현재 현업에서 발생하는 문제점들을 잘 꼬집고 있고 그에 따른 좋은 해결방안들을 제시하고 있다. 어쩌면 8년이 지난 지금까지 많은 기술의 발전과 회사 문화가 바뀌었지만 근본적으로 우리가 가진 문제와 해결점들은 동일한게 아닌가 라는 생각이 든다.

300페이지 정도로 그리 두껍지 않지만 책을 느리게 읽는 내가 일주일만에 읽을 정도로 개인적으로 참 잘 읽히는(?) 책이었다. 내용도 그렇게 어렵지 않았고 책에서 다루는 사례에 나를 빗대어서 읽다보니 좀 더 재밌으면서 이해가 쏙쏙 되었던 것 같다.

이 책의 구성은 인사팀이 읽어보면 좋을만한 인적관리에서부터 사례를 통한 개발자의 생산성을 높이기 위한 방법들을 소개해 준다. 이중에 기억나는 몇가지를 적어본다.

## 망한 프로젝트 중 대다수는 기술적인 문제가 없었다.

책의 저자는 실패하는 프로잭트를 조사하여 원인을 분석하였고 그결과 망한 프로젝트의 대다수는 기술적인 문제라기보다는 소위말해 정치적인 문제였다는 결론을 얻었다. 정치적이라는 말은 많은 의미를 내포하고 있는데 좀더 구체적으로 말하자면 사람간의 관계의 문제였다는 것이다.

관리자가 구성원들을 정해진 틀에 맞게 동작하는 부품으로 대하거나 실수를 용납하지 않거나 무조건 열심히 일하도록 강요하고 이를 위한 프로세스와 평가방식을 도입한다면 단기적으로는 좋아 보일 순 있으나 결국 모든 구성원들이 도망가거나 수동적이게 되면서 활력넘치고 생산적인 팀은 어디에서도 찾아보기 힘들게 될것이다.

최근 주변의 스타트업 소식을 듣거나 뉴스 기사를 보면 MZ세대는 근속기간이 짧고 회사에 대한 충성심보다 자신의 성장에 좀더 초점을 맞추어서 회사생활을 한다라는 이야기를 많이 볼 수 있다. 즉, 구성원들이 좀더 주도적이고 성취감을 느낄 수 있도록, 그리고 스스로가 발전해 가면서 회사에 이바지할 수 있도록 관리자는 고민하고 또 고민해야 할것이다.

이야기의 논지가 살작 벗어난 것 같은데, 결국 말하고자 하는것은 개발팀 관리자는 문제의 원인을 프로그래밍 언어, 개발 프로세스, 자동화 등 기술적인 포인트에 집중할 것이 아니라 우리 팀원이 함께 일하고 싶어하고 서로를 신뢰하면서 함께 발전할 수 있는 역할에 집중해야 한다라는 것이다.

## 저는 아침 일찍, 다른 사람들이 출근하기 전에, 업무효율이 가장 높습니다.

이전에도 많이 느껴왔고 지금도 많이 공감하고 있는 말이다. 업무외 시간에 코드를 작성하면 하루종일 작업한 양보다 훨씬 많은 업무를 수행하는 경우가 많다. 책에서는 사무실의 환경이 열악하다라고 말해주고 있는데, 왜 열악하다라고 표현했냐면 업무의 방해요소가 너무 많기 때문이다라고 말하고 있다.

요즘 코로나로 인해서 재택을 도입한 회사가 아주 많아졌다. 그러면서 좀 더 원격으로 일하기 좋은 환경을 만들기 위해 노력하고 있는데, 개인적으로 나는 이러한 움직임이 계속 이어져서 좀 더 보편화 되었으면 한다. 예를 들면 회의는 미리 일정을 잡아두어서 참가자가 미리 회의준비를 할 수 있도록 한다거나, 메신저는 급한 상황이 아니라면 바로 답변하지 않는것을 전제로 한다거나, 비대면으로 대화를 하다보니 자연스럽게 휘발성 대화보다 문서화의 비율이 높아진다거나 하는 것들이 있을 것이다.

우리 팀원중에 한명도 재택으로 인해서 출근할때보다 업무효율이 몇배이상 높아졌다고 얘기한다. 회의하는 시간이 어느정도 정해져있기 때문에 업무 집중 시간동안은 누구의 방해도 받지않고 개발에만 전념할 수 있는 환경이기 때문이라고 한다. 그리고 자가에 그러한 환경을 마음대로 구성할 수 있었기 때문이기도 했다.

경영진이 아니라면 한계가 있는 부분도 물론 존재하지만 관리자는 팀원들이 업무에 몰입할 수 있는 환경을 조성해주기 위해 끊임없이 노력하고 고민해야한다. 공간적으로 몰입할 수 있는 환경을 조성해주면 가장 좋겠지만 만약 경영진의 결정에 의해 이것이 힘들어졌다면 업무 프로세스에서 몰입을 방해하는 요소는 없는지, 메신저나 회의에서 불편한 점은 없는 지 등을 자주 물어보면서 개선해 나가야 겠다.

## 한 번이라도 단결된 팀에서 즐겁게 일해 본 사람은 단결된 팀의 가치를 잘 안다.

> 단결된 팀은 밥맛이고, 재수 없고, 배타적이고, 자기들끼리 논다. 하지만 어떤 교환 가능한 부속품의 조합보다 관리자의 진짜 목표를 멋지게 달성한다.

아쉽게도 나는 과거에 여기서 말하는 단결된 팀에 일해본 기억이 없는 것 같다. 지금 팀은 어느정도 그 가능성을 보이고 있는데, 그래서 인지 그 단결력을 내가 깨뜨리지 않을까 조바심이 나기도 하다.

단결된 팀을 만들어 간다는 것은 참 어렵다고 느껴진다. 책에서도 단결된 팀을 만들기 위한 처방전을 제시하려고 하였으나 실패했다고 말하고 있다. 억지로 단결할 수 없기 때문에 단결하길 바랄 수 밖에 없고 운에 맡길 뿐이라고 말한다. 다만, 단결된 팀이 아닌 팀 형성을 방해하는 요소를 설명해 주면서 길을 제시해 준다.

- 방어적인 관리
- 관료주의
- 물리적인 격리
- 시간 파편화
- 제품 품질 타협
- 가짜 일정
- 파벌 관리

그러면서 마지막에 아래와 같은 말을 해준다. 참 가슴아픈 말이라 생각했다.

> 대다수 조직은 의식적으로 팀을 죽이겠다고 나서지 않는다. 단지 그렇게 행동할 뿐이다.

## 관리자의 궁극적인 죄는 사람들의 시간을 낭비하는 행위다.

참으로 맞는 말이다. 관리자는 지위가 생기면서 팀원들의 시간을 뺏을 가능성이 높아진다. 보고를 위한 회의소집, 실패한 인력분배, 불필요한 문서 요청 등 관리자가 원한다면 구성원들이 불필요하다고 생각되는 작업들을 요청함으로써 시간을 낭비하도록 만든다.

관리자는 자기도 모르게 이러한 낭비를 만들어내고 있지 않는지 끊임없이 경계해야한다. 만약 이러한 일들이 이미 발생하고 있다는 빠르게 이 문제점을 개선하기 위한 행동을 취함으로써 개선하려는 의지를 구성원들에게 보여주어야 한다. 누구나 완벽할 순 없다는건 모두가 알기에 빠르게 개선한다면 구성원들의 불만도 해결하면서 노력하는 관리자로 보여지는 효과도 가져갈 수 있지 않을까.

## Wrap Up

이 글을 적으면서 다시한번 내 모습들을 돌아보며 반성하는 것들이 많이 있다. 이 책을 사길 잘했다고 생각이 드는게 나중에 다시 읽으면 그때 또 다른 배움이 있을 것 같다라는 생각이 들기 때문이다. 관리자에 생각이 있거나 현재 관리 직책을 맡고 있는 개발자라면 읽어두면 두고두고 도움이 많이 되리라 생각된다.

그나저나 아무래도 오랜만에 팀원들과 면담을 해야겠다 ^^;;
