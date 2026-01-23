---
layout: single
title: "Introduce GIT GONE"
categories:
  - TUTORIALS
tags:
  - git
toc: true
toc_sticky: true
excerpt: git gone 명령어에 대해 알아보자.
---

## Introduction

일반적으로(적어도 나의 경우에는) 개발 협업 시 default branch(이후 여기에서는 `master` branch를 default branch로 말하겠다.)에 변경 코드를 직접 `PUSH`하지 않고 별도의 branch를 생성한 후 `pull request`를 통해 `master`브런치로 병합을 하는 전략을 채택한다. 이때 개발자는 작업을 위해 수많은 브런치를 생성하고 병합 후에 지우게 된다.

[Github](https://github.com/){: target="\_blank" }를 예로 들면 `pull request`가 승인되어 `master`에 병합될 때 자동으로 remote branch를 삭제하도록 설정을 할 수 있다 ([자동 브런치 삭제 설정 방법](https://help.github.com/en/github/administering-a-repository/managing-the-automatic-deletion-of-branches){: target="\_blank" }). GIT은 구조상 Remote Branch가 지워지더라도 Local Branch는 삭제 되지 않고 남아있다. ([Remote Branch와 Local Branch의 차이](https://stackoverflow.com/questions/16408300/what-are-the-differences-between-local-branch-local-tracking-branch-remote-bra){: target="\_blank" }) 그래서 더이상 사용하지 않는 branch를 삭제하고 싶으면 직접 그 브런치들을 삭제하는 작업이 필요하다.

```
$ git branch -d <branchname>
```

삭제할 브런지가 한두개면 괜찮겠지만, 만약 수십개가 된다면?(~~나의 게으름을 혼내키자~~) 지루하고 재미없는 반복적인 작업을 해야 될 것이다. 그리고 브런치명을 잘못 입력하는 실수가 생길 가능성도 많이 있다.

서론이 길었는데, 이럴 때 사용하면 좋은 명령어가 [`git gone`](http://eed3si9n.com/git-gone-cleaning-stale-local-branches){: target="\_blank" }이다. 이제 이 `git gone`에 대해 알아보자.

## How to install

`git gone`은 GIT에서 제공하는 기본 명령어는 아니다. 그래서 명령여를 사용하기 위해서는 명령어 파일을 설치해야 한다.

설치 가이드는 [**eed3si9n님의 블로그**](http://eed3si9n.com/git-gone-cleaning-stale-local-branches){: target="\_blank" }에 적혀있는 방식대로 진행하면 된다.

먼저 [https://github.com/eed3si9n/git-gone](https://github.com/eed3si9n/git-gone){: target="\_blank" }로 이동한다.
[![git-gone-repository](/assets/images/posts/introduce-git-gone/git-gone-repository.png)](https://github.com/eed3si9n/git-gone){: target="\_blank" }

`Clone or download`버튼을 클릭한 후 `Download ZIP`버튼을 클릭하여 소스를 다운로드 받는다.
![git-gone-repository](/assets/images/posts/introduce-git-gone/git-gone-download.png)

### Windows

Windows에서 GIT을 사용하기 위해서는 [git bash](https://git-scm.com/){: target="\_blank" }를 설치해야 한다. 설치 방법은 [Gabii님의 블로그](https://gabii.tistory.com/entry/Git-Git-Bash-219-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0){: target="\_blank" }를 참고하자.

다운로드한 파일을 압축 해제 한 후 `git-gone`파일을 `<GIT-HOME>/bin` 디렉토리에 복사한다. (Windows의 경우 `C:\Program Files\Git\usr\bin`이다.)
![git-gone-copy-directory](/assets/images/posts/introduce-git-gone/git-gone-copy-directory.png)

### Mac

Mac은 기본적으로 GIT이 설치되어 있는데 기본 설치 경로에는 git-gone 스크립트 파일을 넣을 경로를 찾기가 쉽지 않다. (~~혹시 아시는분은 댓글로 공유해 주시면 감사합니다.~~) GIT의 버전도 업데이트 할겸 brew로 새로 설치해 주자. brew 설치는 [공식 홈페이지](https://brew.sh/index_ko){: target="\_blank" }를 참고하자.

먼저 현재 GIT 버전을 확인한다. 버전을 확인하는 이유는 설치 후에 GIT 명령어 실행 스크립트가 정상적으로 교체되었는지 확인하기 위함이다.

```bash
git --version
```

새로운 GIT을 설치한다.

```bash
brew install git
```

버전을 다시 확인해보고 만약 처음과 동일하다면 bash_profile에서 PATH 설정을 해주자.

```bash
echo "export PATH=/usr/local/bin:$PATH" >> ~/.bash_profile
```

다운로드한 파일을 압축 해제 한 후 `git-gone`파일을 script가 모여있는 디렉토리로 복사한다.

```bash
cp git-gone /usr/local/Cellar/git/{version}/libexec/git-core
```

<br/>

`git gone`명령어를 실행해본다. 아래와 같이 표시되면 정상적으로 설치를 완료한 것이다. (만약 명령어를 찾지 못했다는 경고가 표시된다면 커맨드창을 재실행해보자.)
![git-gone-copy-directory](/assets/images/posts/introduce-git-gone/git-gone-command-check.png)



## Syntax

- Synatx
  기본 문법은 아래와 같다. (**_`git gone`명령어를 입력하면 `git gone [-pldD] [<branch>=origin]`이라고 표시되는데 `[-pldD]`에서 `l`은 오타인듯 하다. 현재 버전에서는 `n`이 맞다._**)

  ```
  $ git gone [-pndD] [<branch>=origin]
  ```

- Prune remote branch
  `prune`의 의미는 `가지를 잘라내다`라는 의미로 사용된다. 즉 `원격의 branch를 잘라내다`라는 의미로 사용되는데, Git에서 원격 branch가 삭제된 경우 삭제여부에 대한 정보를 가져오는 명령어이다. <br/>
  동작방식은 `git fetch --prune`과 유사하다.

  ```
  $ git gone -p
  ```

- Dry run: list the gone branches
  `dry run`의 의미는 `시운전`이라는 의미인데, 해당 옵션은 remote branch가 제거되고 local branch가 제거되지 않고 남아 있는 branch 목록을 제공하는 명령어이다.

  ```
  $ git gone -n
  ```

- Delete the tone branches
  remote branch가 제거된 local branch들을 제거한다. <br/>
  동작방식은 `git branch -d`와 유사하다.

  ```
  $ git gone -d
  ```

- Delete the gone branches forcefully
  remote branch가 제거된 local branch들을 강제로 제거한다. <br/>
  동작방식은 `git branch -D`와 유사하다.

  ```
  $ git gone -D
  ```

## Use cases

`git gone`을 사용하는 다양한 사례들이 있겠지만, 내가 어떻게 `git gone`을 사용하는지 적어보겠다.

1. **Pull Request** 병합 시 자동으로 해당 branch가 삭제되도록 설정
   Github의 [Managing the automatic deletion of branches](https://help.github.com/en/github/administering-a-repository/managing-the-automatic-deletion-of-branches){: target="\_blank" }문서를 보면 `master`브런치로 코드가 병합이 되면 자동으로 개발용 branch가 삭제되도록 설정할 수 있다. 이는 불필요한 remote branch가 존재하지 않도록 하기 위해 권장되는 방법 중 하나이다.

2. 삭제된 remote branch 정보를 local repository에 갱신
   local repository에 불필요한 개발용 branch는 가지고 있을 이유가 없기 때문에 지우고자 한다. 이 때, remote repository에서 삭제된 branch들의 정보를 local repository에서 알기 위해서 정보에 대한 갱신 요청이 필요하다. 아래와 같이 명령어를 입력해서 정보를 갱신하자.

   ```
   $ git gone -p
   ```

   ![git-gone-copy-directory](/assets/images/posts/introduce-git-gone/git-gone-pruned-branches.png)

   `fetch` 명령어를 사용해도 동일한 결과를 얻을 수 있다.

   ```
   $ git fetch --prune
   ```

   ![git-gone-copy-directory](/assets/images/posts/introduce-git-gone/git-fetch-prune.png)

3. remote branch가 삭제된 local branch 목록 확인
   local repository에서 remote repository에서 삭제된 branch들이 무엇인지 확인할 수 있다. 정보 갱신 시 삭제된 branch 들을 확인할 수 있지만 만약 local branch들을 삭제하지 않고 다른 작업을 먼저 수행하였을 경우 지우지 않은 local branch들을 확인하는 용도로 사용된다.

   ```
   $ git gone -n
   ```

   표시되는 항목은 `<local branch name> <head-hash> [<remote branch name>: <status>] <head-commit message>`형태로 표시된다. **pruned 상태인 branch만 표시된다.**
   ![git-gone-copy-directory](/assets/images/posts/introduce-git-gone/git-gone-n.png)

   `branch` 명령어를 사용해서도 확인이 가능하다.

   ```
   $ git branch -vv
   ```

   표시되는 항목은 `<local branch name> <head-hash> [<remote branch name>: <status>] <head-commit message>`형태로 표시된다. **모든 branch가 표시된다.**
   ![git-gone-copy-directory](/assets/images/posts/introduce-git-gone/git-branch-vv.png)

4. remote branch가 삭제된 local branch 삭제
   remote repository에서 삭제된 local branch들은 더이상 local branch에 가지고 있을 이유가 없다. 그래서 일괄적으로 삭제한다. `git gone`을 사용하는 가장 큰 이유는 이 branch 들을 하나하나 찾아서 삭제할 필요 없이 하나의 명령어로 원하는 기능을 수행할 수 있기 때문이다.
   단, 주의할 점은 현재 checkout된 branch가 pruned branch라면 삭제가 되지 않으니 삭제되지 않을 branch로 checkout 한 후 삭제 명령어를 실행해주어야 한다. 그리고 `master` branch에 병합 시 `rebase merge`인 경우에는 soft delete가 되지 않을 수 있으니 hard delete 명령어인 `-D`로 실행하면 된다.

   ```
   $ git gone -d
   ```

   ![git-gone-copy-directory](/assets/images/posts/introduce-git-gone/git-gone-d.png)

## Wrap up

`git gone`은 대체가능한 명령어가 존재하기 때문에 굳이 필요하지 않다라고 생각할 수 있다. 하지만 나는 `git gone`을 설치하고 사용해본다면 왜 진작에 사용하지 않았는가 라고 생각할 만큼 사소하지만 큰 편리함을 제공한다고 느꼈다.

**지금 한번의 불편함 때문에 앞으로 생길 여러번의 불편함을 감내하지 말자!**

> [https://help.github.com/en/github/administering-a-repository/managing-the-automatic-deletion-of-branches](https://help.github.com/en/github/administering-a-repository/managing-the-automatic-deletion-of-branches){: target="\_blank" }
>
> [https://backlog.com/git-tutorial/kr/stepup/stepup2_5.html](https://backlog.com/git-tutorial/kr/stepup/stepup2_5.html){: target="\_blank" }
>
> [https://stackoverflow.com/questions/16408300/what-are-the-differences-between-local-branch-local-tracking-branch-remote-bra](https://stackoverflow.com/questions/16408300/what-are-the-differences-between-local-branch-local-tracking-branch-remote-bra){: target="\_blank" }
>
> [http://eed3si9n.com/git-gone-cleaning-stale-local-branches](http://eed3si9n.com/git-gone-cleaning-stale-local-branches){: target="\_blank" }
