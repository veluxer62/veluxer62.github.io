---
layout: single
title: "How to resolve GIT conflict using vimdiff"
categories:
  - TUTORIALS
tags:
  - git
  - mergetool
  - vimdiff
toc: true
toc_sticky: true
excerpt: GIT 사용시 발생할 수 있는 코드 충돌 해결 방법을 알아본다.
---

## Introduction

사내에서 동료들과 협업을 하다보면 코드가 충돌이 나는 상황이 가끔 발생한다. ~~내가 작업한 코드에서 충돌이 나기도 한다.~~
[IntelliJ](https://www.jetbrains.com/ko-kr/idea/){: target="\_blank" } 또는 [Visual Studio Code](https://code.visualstudio.com/){: target="\_blank" } 등 IDEA에서는 충돌난 코드들을 해결해주기 위한 보기 좋고 사용하기 좋은 도구를 제공해준다.

이 글에서는 이러한 툴이 아닌 ~~다소 불편할 수도 있는~~ [vimdiff](http://vimdoc.sourceforge.net/htmldoc/diff.html){: target="\_blank" }를 통한 코드 충돌 해결방법에 대해 알아보고자 한다.

**_해당 튜토리얼에서는 GIT의 사용법을 알고 GIT이 설치되어 있다는 가정하에 진행하였다. 만약 GIT의 사용법을 모른다면 [GIT](https://git-scm.com){: target="\_blank" }를 참고하기 바란다._**

## Preparation

먼저 코드 충돌을 발생 시키기 위한 상황을 발생시켜보자.

### 디렉토리 생성

원하는 위치에 디렉토리를 생성한다.

```
$ mkdir workspace
```

### GIT init 실행

해당 디렉토리 내에서 **init**을 실행한다.

```
$ git init
```

### branch 생성

먼저 2개의 브런치를 생성한다.

```
$ git checkout -b branch-1
$ git checkout -b branch-2
```

### branch-1에 Commit 작성

**branch-1**로 이동한다.

```
$ git checkout branch-1
```

**Workspace**에 `test.txt` 파일을 생성한 후 아래와 같이 작성한다.

```
-- test.txt
test1
test2
test3
```

작업 내용을 **commit** 한다.

```
$ git add .
$ git commit -m "commit1"
```

### branch-2에 commit

**branch-2**로 이동한다.

```
$ git checkout branch-2
```

**branch-2**에 **branch-1**의 **commit**을 **rebase**한다.

```
$ git rebase branch-1
```

**test.txt** 파일을 아래와 같이 수정한다.

```
-- test.txt
test1
test2
test3
test4
test5
```

작업 내용을 **commit** 한다.

```
$ git add .
$ git commit -m "commit2-from-branch-2"
```

### branch-1에 commit 추가

**branch-1**로 이동한다.

```
$ git checkout branch-1
```

**test.txt** 파일을 아래와 같이 수정한다.

```
-- test.txt
test1
test2
test3
test11
test22
test33
```

작업 내용을 **commit** 한다.

```
$ git add .
$ git commit --m "commit2-from-branch-1"
```

### 충돌 발생 시키기

**branch-2**에 **branch-1**의 **commit**을 **rebase**한다.

```
$ git checkout branch-2
$ git rebase branch-1
```

## Resolve conflict

위 순서대로 잘 진행하였다면 아래 그림과 같은 충돌이 발생하였을 것이다.
![conflict](/assets/images/posts/git-mergetool-vimdiff-tutorial/conflict-situation.png)

이제 이러한 충돌상황을 **vimdiff**를 이용하여 해결해보자.

### mergetool 실행

먼저 **mergetool**을 **vimdiff**로 실행해보자.

```
$ git mergetool --tool vimdiff
```

--tool 옵션을 주지 않으려면 `git config merge.tool vimdiff` 통해 설정이 가능하다.

## resolve conflict

실행하면 아래와 같은 화면을 볼 수 있다.

![vimdiff](/assets/images/posts/git-mergetool-vimdiff-tutorial/mergetool-vimdiff.png)

vimdiff에서 사용가능한 명령어들은 [링크](http://vimdoc.sourceforge.net/htmldoc/diff.html){: target="\_blank" }를 참고하기 바란다.

### 파일 수정

원하는 최종 형태는 아래와 같다.

```
--test.txt
test1
test2
test3
test111
test222
test333
test4
test5
```

그럼 아래와 같이 파일을 수정한다.

![resolve-conflict](/assets/images/posts/git-mergetool-vimdiff-tutorial/resolve-conflict-in-vimdiff.png)

그런다음 중앙화면은 저장 후 종료 (`:wq`)하고 나머지는 그냥 종료(`:q`)한다.

vim과 관련된 명령어는 [https://vimhelp.org/](https://vimhelp.org/){: target="\_blank" }에서 참고 바란다.

### 충돌 해결 완료

정상적으로 vimdiff를 종료한 후 화면은 아래와 같다.

![resolve-step-1](/assets/images/posts/git-mergetool-vimdiff-tutorial/resolve-step-1.png)

나머지 커밋을 적용하기 위해 아래 명령을 실행하자.

```
$ git rebase --continue
```

실행 후 정상적으로 성공했다면 `REBASE 1/1`이 사라진 브런치 명이 표시 될 것이다.

![resolve-step-2](/assets/images/posts/git-mergetool-vimdiff-tutorial/resolve-step-2.png)

## Backup file

끝내려고 하는데 `test.txt.orig`파일이 생성이 된것을 볼 수 있다.

```
$ git status
```

![backup file](/assets/images/posts/git-mergetool-vimdiff-tutorial/conflict-backup.png)

이 파일은 충돌파일을 해결하기 전 백업파일인데 제거하거나 해결되었다는 표시를 남기는 등의 조치를 취할 수 있다.

여기에서는 파일이 표시되는게 싫으니 지워버리겠다.

```
$ git clean -f *.orig
```

git clean의 옵션은 [이곳](https://git-scm.com/docs/git-clean){: target="\_blank" }을 참고하기 바란다.

만약 해당 파일이 생성되는 것이 필요 없고 백업 파일을 지우는 행위가 싫다면 `git config --global mergetool.keepBackup false` 설정을 통해 생성되지 않게 할 수 있다.

## Conclusion

좋은 IDEA나 보기 좋은 Tool을 이용하여 좀더 쉽게 충돌을 해결 할 수 있다. 하지만 **vimdiff**는 별도의 설치 없이 충돌을 쉽게 해결할 수 있다는 점에서 장점이 있다고 생각한다.

> [https://git-scm.com/](https://git-scm.com/){: target="\_blank" }
>
> [https://www.jetbrains.com/ko-kr/idea/](https://www.jetbrains.com/ko-kr/idea/){: target="\_blank" }
>
> [https://code.visualstudio.com/](https://code.visualstudio.com/){: target="\_blank" }
>
> [http://vimdoc.sourceforge.net/htmldoc/diff.html](http://vimdoc.sourceforge.net/htmldoc/diff.html){: target="\_blank" }
> [https://www.rosipov.com/blog/use-vimdiff-as-git-mergetool/](https://www.rosipov.com/blog/use-vimdiff-as-git-mergetool/){: target="\_blank" }
> [https://git-scm.com/docs/git-clean](https://git-scm.com/docs/git-clean){: target="\_blank" }
