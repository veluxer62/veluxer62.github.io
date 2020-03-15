---
layout: single
title: "2020년 3월 2주차 회고"
categories:
  - RETROSPECTIVE
tags:
  - 회고
toc: true
---

## KEEP

[Occupying 개인 프로젝트](https://github.com/veluxer62/occupying){: target="\_blank" }의 코드를 개선하고 새로운 기능을 추가하기 위해서 오픈소스인 [PlantUML](https://plantuml.com/){: target="\_blank" }을 사용하여 uml 다이어그램을 작성해보았다. PlantUML은 사용자가 다이어그램을 그릴 수 있는 편한 GUI를 제공해주지는 않지만 코드를 통해 다이어그램을 그릴 수 있다. 개발자에게는 익숙한 방법이면서 어느 환경에서도 작성가능하다는 점은 큰 장점인 것 같다. 물론 결과물도 나쁘지 않고 다양한 다이어그램을 제공한다는 점도 내가 이 것을 사용하게된 이유이다. <br/>
먼저 현재 제공하고 있는 기능이 어떤 흐름을 가지는지 볼 수 있는 [시퀀스 다이어그램](https://github.com/veluxer62/occupying/wiki/%EC%8B%9C%ED%80%80%EC%8A%A4-%EB%8B%A4%EC%9D%B4%EC%96%B4%EA%B7%B8%EB%9E%A8){: target="\_blank" }을 위키에 그려보았다. 시퀀스 다이어그램을 그리니 프로그램이 어떤 순서를 가지고 동작하는지 한눈에 볼 수 있었다. 이 다이어그램은 다른 사람들에게 클래스 다이어그램을 그릴때 어떻게 동작하면 좋을 지 설명하는데 큰 도움을 주었다. <br/>
그런다음 현재 기능에서 새로운 기능을 추가할 때 어떻게 구조를 바꾸면 좋은지 고민하면서 [클래스 다이어그램](https://github.com/veluxer62/occupying/wiki/%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%8B%A4%EC%9D%B4%EC%96%B4%EA%B7%B8%EB%9E%A8){: target="\_blank" }을 작성해보았다. 클래스 다이어그램을 작성할 때에는 회사 동료들에게 어떻게 인터페이스를 정의하면 좋을 지 많이 물어보고 의견을 구하였다. 혼자서 무언가를 설계할 때보다 훨씬 좋은 설계 의견을 받을 수 있었고, 이번에 디자인 패턴을 공부하면서 막연했던 사용 사례들을 적용할 수 있는 경험을 가질 수 있어서 좋았다. <br/>
현재는 설계는 어느정도 마무리 지었고 구현이 남아 있다. 구현중에 놓쳤거나 변경할 내용이 생길 순 있겠지만 이렇게 먼저 설계를 그려보고 정의한 후에 개발을 시작하는 방식을 많이 연습해보면 기존에 가진 나의 나쁜 개발 습관을 많이 고칠 수 있을 것 같다.

## WATCH

자기전에 휴대폰을 의도적으로 보지 않고 자려고 노력하고 있다. 항상 안보진 않지만 그래도 12시 전에는 휴대폰을 손에서 놓고 잠이 든다. 하지만 아직도 누워서 휴대폰을 보고 싶은 욕구를 떨쳐버리는게 쉽지 않다. 이것도 버릇을 들이면 점차 나아지겠지 하고 있다. 의도적 수련이라는 말이 여기에 어울릴지는 잘 모르겠지만 계속해서 노력하다보면 휴대폰을 안보는 사람이 되어있지 않을까.

다음 교양서적으로 무얼 읽을지 고민중이다. 아니 교양서적을 읽을지 기술서적을 읽을지 고민중이다. 현재는 [클린 아키텍처](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262472&orderClick=LEa&Kc=){: target="\_blank" }를 읽고 있는데, 확실히 관심있는 주제를 읽으니 더 적극적으로 독서를 한다. 하지만 [넛지](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788901227542&orderClick=LAG&Kc=){: target="\_blank" }를 다 읽고 나서도 많은 것을 느꼈고 그 책을 읽은 시간이 아깝다는 시간이 들지 않았다. 다만 다음에 교양서적을 고를 땐 좀더 쉬운 어조로 되어 있는 책을 선택해야 할것 같다. 이 다음 책으로 어떤 책을 읽을 지는 고민을 좀더 해보아야 겠다. 굳이 이걸 나누기 보다 그냥 책을 다읽을때 쯤에 그때 읽고 싶은 책을 선택하는 것도 나쁘지 않는 것 같다.

## CHANGE

이번 주는 결국 운동을 하루도 하지 못했다. 집에서 할 운동 계획을 잡아보고자 하였지만 생각보다 나는 더 나태한 사람이었다 ㅠㅠ. 10일까지 휴관을 하고 11일부터 다시 헬스장이 오픈하니까 그때부터 다시 운동을 열심히 다녀야겠다.

## LAST ACTION ITEMS

- [ ] 집에서 할 운동 계획하기
- [x] 하루에 30분 이상 책 읽기
- [x] 12시에는 잠자리에 들기
- [x] Occupying 프로젝트 인터페이스 설계
- [x] 잠자리에 든 이후에 휴대폰 보지 않기

## ACTION ITEMS

- Occupying 프로젝트 리펙토링
- 하루에 30분 이상 책 읽기
- 12시에는 잠자리에 들기
- 휴대폰 보는 시간 줄이기
- 운동 열심히 하기

## ACHIEVEMENT RATE

영어 공부
<progress value="7" max="7"></progress>
7/7 (<b>100%</b>)

운동
<progress value="0" max="7"></progress>
0/7 (<b>0%</b>)

- [x] 블로그 작성

1. [GIT Actions을 이용한 CI/CD 적용기 - CD편](/tutorials/tutorial-of-continuous-deployment-with-git-actions/){: target="\_blank" }
