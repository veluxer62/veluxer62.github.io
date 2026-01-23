---
layout: single
title: "게시글 작성 규칙"
categories:
  - RULE
tags:
  - rule
  - markdown
toc: true
toc_sticky: true
excerpt: 게시글을 작성할때 규칙을 정해두고 작성하면 일관성있는 글을 작성할 수 있을것 같아 아래에 그 규칙들을 정하고 지키고자 한다.
---

게시글을 작성할때 규칙을 정해두고 작성하면 일관성있는 글을 작성할 수 있을것 같아 아래에 그 규칙들을 정하고 지키고자 한다.
규칙은 게시글을 작성하면서 계속 추가될 수 있다.

## 특별한 경우가 아니면 마크다운 언어로 작성한다.

마크다운은 간단하고 일관성있는 글을 작성하기 쉬우므로 되도록이면 마크다운으로 글을 작성한다.

작성규칙은 [링크](https://daringfireball.net/projects/markdown/syntax){: target="\_blank" }를 참고하자.

## 가장 상위 헤더는 `##`으로 한다.

가장 상위 헤더를 `#`으로 할 경우, 현재 사용하고 있는 테마의 타이틀과 동일하게 보여서 `##`을 사용하도록 한다.

## 화면을 접어야 할때는 아래와 같이 작성하자.

```
<details><summary>표시문구</summary>
내용
</details>
```

출처: [joyrexus님의 gist](https://gist.github.com/joyrexus/16041f2426450e73f5df9391f7f7ae5f){: target="\_blank" }

## 적당한 분량의 글을 작성하자.

[Medium에서는 독자의 관심을 끄는 글의 길이는 7분정도라고 소개하고 있다.](https://medium.com/data-lab/the-optimal-post-is-7-minutes-74b9f41509b){: target="\_blank" }
읽는 사람이 지루함을 느끼지 않고, 작성하는 나도 부담감을 많이 느끼지 않는 선에서 5분정도의 읽는 시간을 가질 수 있는 글이 적당할 것 같다.
영어의 경우 1000자, 한글의 경우 500자 정도가 1분 정도라고 하니 5분을 위해서는 한글 기준으로 2,500자 정도를 염두해두고 작성해 보자.

출처: [nacyot님의 글](https://www.44bits.io/ko/post/8-suggestions-for-tech-programming-blog){: target="\_blank" }

## 글 내용은 아래 4가지 항목을 넣도록 하자.

4가지 규칙은 아래와 같다.

- 서론
- 본론
- 결론
- 출처

**서론**에서는 주로 내가 글을 작성한 이유 또는 배경에 대해 기술하자.

**본론**에서는 내가 적고자 하는 글을 유형에 맞게 작성한다. [Daniele Procida님께서 작성하신 글](https://www.divio.com/blog/documentation/){: target="\_blank" }에는 소프트웨어 블로그를 작성할 때 유형별로 어떻게 작성하면 좋은지에 대한 좋은 가이드를 제공해준다.

**결론**은 작성한 글에 대한 내가 가진 생각을 적으면서 글을 마무리 한다.

**출처**에서는 이 글을 작성할때 참고했던 글들에 대한 출처들을 기술하자. 링크는 클릭시 새창으로 표시되게 하기 위해 아래와 작성규칙으로 기술하자.

```
[타이틀](링크){: target="\_blank" }
```

> [https://www.44bits.io/ko/post/8-suggestions-for-tech-programming-blog](https://www.44bits.io/ko/post/8-suggestions-for-tech-programming-blog){: target="\_blank" }
>
> [https://jekyllrb.com/docs/posts/](https://jekyllrb.com/docs/posts/){: target="\_blank" }
>
> [https://gist.github.com/joyrexus/16041f2426450e73f5df9391f7f7ae5f](https://gist.github.com/joyrexus/16041f2426450e73f5df9391f7f7ae5f){: target="\_blank" }
>
> [https://www.divio.com/blog/documentation/](https://www.divio.com/blog/documentation/){: target="\_blank" }
