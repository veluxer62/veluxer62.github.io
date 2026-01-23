---
layout: single
title: "Swap Memory"
categories:
  - REFERENCE
tags:
  - swap
  - paging

toc: true
toc_sticky: true
excerpt: Swap Memory에 대해 알아보자.
---

## About Swap Memory

Swap Memory에 대한 내용을 구글링해서 찾아보아도 정확한 정의를 Wiki에 정의해 놓은 글을 찾을 수 없었다. 그나마 찾은 것은 Wiki에 적힌 한글로 된 [Swap Memory](http://wiki.pchero21.com/wiki/Swap_memory){: target="\_blank" } 아래 같은 정의 뿐이었다.

> Swap메모리는 주 메모리가 부족할 때 하드디스크와 같은 공간을 메모리로 사용하기 위한 가상 메모리이다.(윈도우에선 가상 메모리(페이징 메모리)라고 지칭함)

위 내용에서도 언급 되지만 Swap Memory는 **Paging**을 말한다는 것을 알 수 있었다. 그래서 Swap Memory 아니, (https://en.wikipedia.org/wiki/Paging){: target="\_blank" }에 대해서 정의하면 아래와 같다.

> 컴퓨터 운영 시스템에서, 페이징은 컴퓨터가 주 기억장치에 사용하기 위해 보조기억장치에서 데이터를 저장하고 조회하는 메모리 관리 방식을 말한다. 페이징은 프로그램이 사용 가능한 물리적 메모리의 크기를 조과하도록 하기 위해서 보조 기억장치를 사용하는 최신 운영 체제의 가상 메모리 구현의 중요한 부분이다.

## Check Swap Memory

대표적인 OS인 Windows와 Linux에서 Swap Memory를 확인하는 방법을 알아보자. (Windows는 Page File이라고 한다.)

### Windows (Windows 10 기준)

1. `Win` + `E` 버튼을 눌러 File Explorer를 실행한다.
2. `This PC`를 오른쪽클릭한 후 `Properties`를 선택하여 System 창을 연다.
   ![how-to-open-system-window](/assets/images/posts/swap-memory/how-to-open-system-window.png)
3. `Advanced system settings` 버튼을 선택하여 Sytem Properties 창을 연다.
   ![system](/assets/images/posts/swap-memory/system-window.png)
4. `Advanced` 탭을 선택한 후 `Performance` 항목의 `Settings...` 버튼을 클릭하여 Performance Options 창을 연다.
   ![system-properties](/assets/images/posts/swap-memory/system-properties-window.png)
5. `Advanced` 탭을 선택한 후 `Virtual memory` 항목에서 용량을 확인한다.
   ![check-virtual-memory](/assets/images/posts/swap-memory/check-virtual-memory-window.png)

### Linux (Linux 4.14 기준)

```
$ free -m
```

![check-virtual-memory](/assets/images/posts/swap-memory/check-virtual-memory-linux.png)

## Change Swap Memory

Swap Memory를 변경하는 방법에 대해 알아보자.

### Windows (Windows 10 기준)

1. Virtual memory 설정 화면으로 이동한다. (위에서 작성한 가이드를 참고하자 [이동](/reference/swap-memory/#windows-windows-10-기준))
2. `Change...` 버튼을 클릭하여 Virtual memory 변경 창으로 이동한다.
   ![swap-change-button-window](/assets/images/posts/swap-memory/swap-change-button-window.png)
3. `Automatically manage paging file size for all drivers`를 비활성화 한 후 `Custom size:`를 선택한다. 그런다음 원하는 사이즈를 입력한다.
   ![change-swap-window](/assets/images/posts/swap-memory/change-swap-window.png)
4. `OK` 버튼을 클릭하여 변경을 완료한다.

### Linux (Linux 4.14 기준)

1. Swap memory로 사용할 파일 생성

   ```
   $ fallocate --length 2G /swapfile
   ```

2. 파일 사용 권한 설정

   ```
   $ chmod 600 /swapfile
   ```

3. Swap 파일 설정

   ```
   $ mkswap /swapfile
   ```

4. Swap 파일 활성화

   ```
   $ swapon /swapfile
   ```

5. 재부팅 시 스왑 영역 활성화 처리

   ```
   $ blkid /swapfile
   /swapfile: UUID="484df5da-4dcb-418b-89ec-659ba821f825" TYPE="swap"

   $ vi /etc/fstab

   # 아래 문구 입력 후 저장
   UUID=484df5da-4dcb-418b-89ec-659ba821f825 swap swap defaults 0 0
   ```

## How big

사실 Swap Memory를 얼마만큼 설정하느냐는 사용자에 따라 상황에 따라 다르다. Windows의 경우 자동으로 Page File의 크기를 조정해준다. Linux의 경우에는 자동으로 메모리 양을 관리해 주지는 않는다.

Redhat에서 권고하는 RAM/Swap Memory 권장 사항은 아래와 같다.

- RAM이 2GB 미만인 경우 RAM의 2배로 Swap Memory 설정
- RAM이 2GB ~ 8GB 인 경우 RAM과 동일하게 Swap Memory 설정
- RAM이 8GB ~ 64GB 인 경우 4GB ~ 8GB로 Swap Memory 설정
- RAM이 64GB를 초과하는 경우 4GB ~ 8GB로 Swap Memory 설정

## Wrap up

지금까지 Swap Memory에 대해 알아보고 간단하게 나마 Swap Memory가 어떤 역할을 하고 어떻게 설정되어 있는지 어떻게 설정할 수 있는지 배워 보았다. 깊이 알아보려면 한없이 파고들 수 있는 게 메모리인 것 같다. 좀더 전문적으로 깊은 내용을 확인하고 싶으면 아래 참고 링크들을 보면 좋을 것 같다.

> [http://wiki.pchero21.com/wiki/Swap_memory](http://wiki.pchero21.com/wiki/Swap_memory){: target="\_blank" }
>
> [https://en.wikipedia.org/wiki/Paging](https://en.wikipedia.org/wiki/Paging){: target="\_blank" }
>
> [https://www.geeksinphoenix.com/blog/post/2016/05/10/how-to-manage-windows-10-virtual-memory.aspx](https://www.geeksinphoenix.com/blog/post/2016/05/10/how-to-manage-windows-10-virtual-memory.aspx){: target="\_blank" }
>
> [https://www.howtogeek.com/196238/how-big-should-your-page-file-or-swap-partition-be/](https://www.howtogeek.com/196238/how-big-should-your-page-file-or-swap-partition-be/){: target="\_blank" }
>
> [https://devanix.tistory.com/311](https://devanix.tistory.com/311){: target="\_blank" }
