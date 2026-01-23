---
layout: single
title: "How to update forked repository master"
categories:
  - TUTORIALS
tags:
  - git
toc: true
toc_sticky: true
excerpt: 현재 운영중인 블로그는 Minimal Mistakes Jkeyll theme라는 깃 블로그 테마를 사용해서 운영중이다. **Minimal Mistakes Jkeyll theme**를 사용하는 방법은 ZIP파일을 다운로드 받아서 사용하는 방법과 fork를 이용하여 사용하는 방법이 있다. 나는 이전에 작업했던 commit 내용을 볼 수 있다는 점과 원본 브런치의 업데이트를 병합하기 용이하다는 점을 장점으로 생각 되어 fork를 사용하는 방법을 선택하였다. (~~물론 Pull Request시 브런치를 일일히 선택해줘야 한다는 단점도 있다~~) 이 글에서는 **Minimal Mistakes Jkeyll theme**를 fork한 내가 작성중인 블로그에 오리지널 브런치의 업데이트 내용을 병합하기 위한 방법을 설명하고자 한다.
---

## Introduction

현재 운영중인 블로그는 [Minimal Mistakes Jkeyll theme](https://github.com/mmistakes/minimal-mistakes){: target="\_blank" }라는 깃 블로그 테마를 사용해서 운영중이다. **Minimal Mistakes Jkeyll theme**를 사용하는 방법은 ZIP파일을 다운로드 받아서 사용하는 방법과 fork를 이용하여 사용하는 방법이 있다. (자세한 설명은 [링크](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#installing-the-theme){: target="\_blank" }를 참고)
나는 이전에 작업했던 commit 내용을 볼 수 있다는 점과 원본 브런치의 업데이트를 병합하기 용이하다는 점을 장점으로 생각 되어 fork를 사용하는 방법을 선택하였다. (~~물론 Pull Request시 브런치를 일일히 선택해줘야 한다는 단점도 있다~~)

이 글에서는 **Minimal Mistakes Jkeyll theme**를 fork한 내가 작성중인 블로그에 오리지널 브런치의 업데이트 내용을 병합하기 위한 방법을 설명하고자 한다.

## How to

### Register upstream

원본 원격 브런치를 upstream 이라고 하는데 처음에 원본 브런치는 fork하면 원본 원격 브런치의 위치는 등록되어 있지 않다. 그래서 원본 원격 브런치의 위치가 설정되어 있는 지 확인한 후 없다면 새롭게 등록해 주자.

현재 등록되어 있는 원격 브런치 위치 목록을 확인한다. GIT의 `remote`명령어를 사용하며 `-v`옵션을 주어 URL정보도 함께 확인한다.

```
$ git remote -v
origin  https://github.com/veluxer62/veluxer62.github.io.git (fetch)
origin  https://github.com/veluxer62/veluxer62.github.io.git (push)
```

나의 원격 브런치만 확인이 되니 원본 원격 브런치를 추가해 주자. (여기서 원격 원본 브런치의 경로는 `https://github.com/mmistakes/minimal-mistakes` 이다.)

```
$ git remote add upstream https://github.com/mmistakes/minimal-mistakes
```

제대로 추가되었는지 확인해 보자. 아래와 같이 upstream 경로가 생성되어 있으면 정상적으로 등록된 것이다.

```
$ git remote -v
origin  https://github.com/veluxer62/veluxer62.github.io.git (fetch)
origin  https://github.com/veluxer62/veluxer62.github.io.git (push)
upstream        https://github.com/mmistakes/minimal-mistakes.git (fetch)
upstream        https://github.com/mmistakes/minimal-mistakes.git (push)
```

### Fetch

이제 원본 원격 저장소의 업데이트 정보들을 가져와 보자. 데이터를 가져오는 방법은 `fetch` 명령어를 사용하는 것인데, 나의 원격저장소의 정보를 가져오는것과 유사하다. 단지 `upstream` 명령어만 추가하면 된다.

```
$ git fetch upstream
```

### Merge

원본 원격 저장소를 최신상태로 하였다면 이제 나의 브런치에 변경사항을 병합하면 된다. 업데이트 방법은 나의 원격 저장소를 병합하는 방법과 유사하다.

```
$ git merge upstream/master
```

## Wrap up

지금까지 fork한 나의 repository에 원본 원격 브런치의 업데이트 항목을 병합하는 방법에 대해서 알아보았다. Contribution이나 나와 같이 Git theme를 사용할 때 fork하는 방식으로 repository를 생성한 경우 최신 상태로 원본 원격 브런치의 내용을 유지하기 위해서 유용한 방법이니 꼭 숙지하고 주기적으로 업데이트하면 좋을 듯 하다.

> [https://github.com/mmistakes/minimal-mistakes](https://github.com/mmistakes/minimal-mistakes){: target="\_blank" }
>
> [https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#installing-the-theme](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#installing-the-theme){: target="\_blank" }
>
> [https://jybaek.tistory.com/775](https://jybaek.tistory.com/775){: target="\_blank" }
>
> [https://parksb.github.io/article/28.html](https://parksb.github.io/article/28.html){: target="\_blank" }
