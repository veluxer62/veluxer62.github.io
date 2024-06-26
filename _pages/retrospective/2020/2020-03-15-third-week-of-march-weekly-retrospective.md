---
layout: single
title: "2020년 3월 3주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-third-week-of-march-weekly-retrospective/
---

## KEEP

이번주에 [English for Developers](http://www.yes24.com/Product/Goods/19992192){: target="\_blank" }책으로 하는 영어공부는 마무리 했다. 이제 영어공부는 [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){: target="\_blank" }를 번역하면서 해보기로 하였다. 책으로 공부할때는 명확히 한 챕터씩 공부하면 되어서 시간안배를 따로 하지 않아도 되었는데, 이제는 그렇지 않으니 시간안배도 잘해야 할 것 같다. 현재 2일 정도 진행해 보았는데, 30분정도 읽고 30분 정도 번역하는 식으로 진행하면 알맞을 것 같다. 책으로 번역된 글을 보다가 실제로 번역 글 없이 독해를 진행하다 보니 쉽지는 않는것 같다. (~~맞게 해석했는 지도 모르겠다.~~) 그래도 내가 읽어 보고싶고 공부해야 할 내용에 대한 번역을 하다 보니 집중도 잘되고 유익한 것 같아 일단 **Spring WebFlux**를 다 번역해 본 후 향후 어떻게 진행할 지 고민해 봐야겠다.

이번주부터 매주 화,목 저녁에 코딩 튜터를 하기 시작했다. 첫 날에는 첫 수업이라는 긴장감과 수강생들의 동공이 흔들리는 모습(?)을 보며 스트레스를 많이 받았는지 집에 오자마자 곯아 떨어졌다 ㅎㅎ. 다행이 두번째 수업때는 수강생분들이 나름 적응하시고 잘 따라와 주셔서 첫날보다는 정신적으로 덜 힘들었다. 누군가에게 내가 알고 있는 지식을 전달해준다는게 쉽지는 않는 것 같다. (~~모든 학교/학원 선생님들 존경합니다.~~) 그래도 이 튜터를 해보겠다고 결심한 이유가 내가 가진 지식을 나누어 주고자 하는 것과 내가 가진 지식의 기초를 좀더 단단히 하는 것, 그리고 조금더 겸손해지기 위함이니 앞으로도 더 좋은 튜터가 되기 위해 노력해야 겠다.

우연히 [어썸데브블로그](https://awesome-devblog.now.sh/){: target="\_blank" }라는 곳을 알게 되었다. 이곳의 컨셉은 개인 또는 팀의 개발 블로그나 관련 글들을 한곳에서 볼 수 있도록 해주는 서비스인데, 평소에 이곳 저곳의 블로그에 드나들며 최신 기술이나 글들을 접하던 나에게 아주 유익한 서비스이다. (~~만들어 주신분 감사하고 존경합니다.~~) 모든 글을 다 읽어볼 순 없지만 제목들만 둘러보아도 최신 트렌드 동향을 볼 수 있을 것 같으니 자주 이용해야 겠다.

## WATCH

[Occupying 개인 프로젝트](https://github.com/veluxer62/occupying){: target="\_blank" }의 리펙토링을 진행하면서 Spring WebFlux의 Document가 많이 변경된 것을 알게 되었다. ~~문서를 본지가 반년정도 지났으니 바뀐게 당연할 수도 있겠다.~~ <br/>
Spring에서 이제 Kotlin의 사용을 점점 확대해 가고 있다고 눈에 띄게 느껴지는게 Kotlin에 대한 언급이 많아지고 예제 코드도 많이 제공하고 있다. <br/>
문서에서 보면 WebFlux에서 Kotlin사용 시 Coroutine 사용을 추가하고 예제코드도 변경한 것을 볼 수 있다. 현재 내가 만든 프로젝트의 언어도 Kotlin으로 되어있기 때문에 Coroutine을 사용할 수 있다. 이전에는 Coroutine을 공식적으로 지원하지 않아서 Reactor를 사용하였다. <br/>
여기에서 나의 고민은 Coroutine을 사용할지 말지 선택하는 것이다. Coroutine은 Kotlin을 사용한다면 반드시 익혀야 할 기술이니 익혀둘겸 해보는 것도 나쁘지 않지만 순차적으로 먼저 기존코드 기반으로 리펙토링을 끝낸 후 Coroutine을 적용할지 바로 적용할지 고민중이다. 유닛 테스팅에 대한 공식적인 테스트 방법이 없어 헤매고 있는 점 (~~내가 못찾은 거일 수도 있다.~~)과 Coroutine 관련 의존 라이브러리를 추가해야 한다는 점은 좀 마음에 들지 않는다.

## CHANGE

일주일의 패턴이 갑작스럽게 바껴서 그런지 지키고자 했던 Action Item들을 많이 지키지 못하였다. 하루 책읽기와 휴대폰 적게 보기, 운동 열심히 하기는 매주 이렇게 회고를 하면서 채찍질을 해야할 필요성이 있어 보인다! 다음주에는 좀더 분발하자.

## LAST ACTION ITEMS

- [x] Occupying 프로젝트 리펙토링
- [ ] 하루에 30분 이상 책 읽기
- [x] 12시에는 잠자리에 들기
- [ ] 휴대폰 보는 시간 줄이기
- [ ] 운동 열심히 하기

## ACTION ITEMS

- Occupying 프로젝트 리펙토링
- 하루에 30분 이상 책 읽기
- 휴대폰 보는 시간 줄이기
- 운동 열심히 하기
- 다이어트! (저녁 간단히 먹기)

## ACHIEVEMENT RATE

영어 공부
<progress value="7" max="7"></progress>
7/7 (<b>100%</b>)

운동
<progress value="1" max="7"></progress>
1/7 (<b>14%</b>)

- [x] 블로그 작성

1. [Introduce GIT GONE](/tutorials/introduce-git-gone/){: target="\_blank" }
