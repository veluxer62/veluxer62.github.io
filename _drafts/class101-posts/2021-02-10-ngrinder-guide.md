---
layout: single
title: "NGrinder 사용법"
categories:
  - POST
---

# 템플릿

```yaml
version: "3.1"

services:
  controller:
    image: ngrinder/controller:3.4
    ports:
      - 9999:80
      - 16001:16001
      - 12000-12009:12000-12009
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    networks:
      - ngrinder-network

  agent:
    image: ngrinder/agent:3.4
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    command:
      - controller:80
    networks:
      - ngrinder-network

networks:
  ngrinder-network:
```

`service.controller.prots` 에서 `9999` 부분이 로컬에서 접속할 포트임. 원하는 포트로 변경해도됨

`agent` 를 하나 이상 두고 싶으면

`services.agent`를 여러개 복사하시고 아래와 같이 이름만 바꾸시면 됨

```yaml
version: "3.1"

services:
  controller:
    image: ngrinder/controller:3.4
    ports:
      - 9999:80
      - 16001:16001
      - 12000-12009:12000-12009
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    networks:
      - ngrinder-network

  agent-1:
    image: ngrinder/agent:3.4
    volumes:
      - ~/.ngrinder:/opt/ngrinder-controller
    command:
      - controller:80
    networks:
      - ngrinder-network

	agent-2:
	    image: ngrinder/agent:3.4
	    volumes:
	      - ~/.ngrinder:/opt/ngrinder-controller
	    command:
	      - controller:80
	    networks:
	      - ngrinder-network

networks:
  ngrinder-network:
```

# 명령어

파일명이 `docker-compose` 인 경우

```bash
docker-compose up
```

파일명을 다르게 한 경우

```bash
docker-compose -f {파일명} up
```

캐시를 사용하지 않고 재 빌드 하고 싶은 경우 `build` 옵션을 주세여

백그라운드에서 실행하려명 `d` 옵션을 주세여

```bash
docker-compose up [--build] [-d]
```

Controller 뜨는데 겁나 오래 걸림!

# Controller 접속

`[http://localhost:9999/login](http://localhost:9999)` 접속

![ngrinder-login](/assets/images/draft/class101-posts/ngrinder-login.png)

초기 ID/PW 는 `admin`/ `admin` 

## Agent 확인

상단 프로필에서 `Agent Management` 선택

![ngrinder-agent-mgmt](/assets/images/draft/class101-posts/ngrinder-agent-mgmt.png)

아래 그림처럼 Agent 확인되면 잘 만든거!

![ngrinder-agent-list](/assets/images/draft/class101-posts/ngrinder-agent-list.png)

# Quick Start

메인 화면에 보면 Quick Start 배너가 보임

거기에서 테스트할 URL 넣고 `Start Test` 누르세여

![quick-start](/assets/images/draft/class101-posts/ngrinder-quick-start.png)

원하는 테스트 스펙을 입력하고 `Save and Start` 버튼을 누름 → 팝업이 뜨는데 `Run Now` 를 누름

![ngrinder-start-setting](/assets/images/draft/class101-posts/ngrinder-start-setting.png)

실행 결과를 볼 수 있음

![ngrinder-result](/assets/images/draft/class101-posts/ngrinder-result.png)

# 스크립트 작성

상단의 `script` 메뉴 선택 후 `Create a script` 버튼 선택

![ngrinder-create-script](/assets/images/draft/class101-posts/ngrinder-create-script.png)

파일명 입력 후 `Create`

![ngrinder-script-name](/assets/images/draft/class101-posts/ngrinder-script-name.png)

스크립트 작성 후 `save` 눌러서 저장

작성방법은 [여기](https://brownbears.tistory.com/27?category=168280) 참고하자!

![ngrinder-script-save](/assets/images/draft/class101-posts/ngrinder-script-save.png)

## 스크립트를 이용한 테스트

`Performance Test` 메뉴로 이동해서 `Create Test` 버튼을 클릭

![ngrinder-create-script-test](/assets/images/draft/class101-posts/ngrinder-create-script-test.png)

테스트 내용 입력하고 `Script` 선택 후 `Save` 또는 `Save and Start` 클릭!

![ngrinder-select-script](/assets/images/draft/class101-posts/ngrinder-select-script.png)

# Reference

공식 홈페이지

[nGrinder](http://naver.github.io/ngrinder/)