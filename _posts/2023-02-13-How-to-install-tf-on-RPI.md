---
layout: post
title: "Raspberry PI에 tensorflow 설치"
date: 2023-02-13
---


Raspberry PI(RPI)에 tensorflow를 설치하려면 생각보다 복잡하다. <br/>
그래서 오늘은 RPI 설치부터 tensorflow를 설치하는 과정 전체를 다루고자 한다. <br/><br/>

# 01. RPI 설치
<hr>

- [RPI 공식홈페이지](https://www.raspberrypi.com/software/)에서 RPI Imager를 설치한다.
<p align="center"><img src="https://user-images.githubusercontent.com/33558083/218372952-c0ca82ad-4a01-4776-8054-3e9face90c13.JPG" height="200px" width="300px"></p>

- 원하는 OS 버전, 저장소를 선택한다.
  - 본인은 64-bit 선택
  - 현 게시물은 64-bit에 초점을 맞춰 진행함.

- 버전과 OS를 선택하면 그림과 같이 톱니 모양의 아이콘이 생성된다.
<p align="center"><img src="https://user-images.githubusercontent.com/33558083/218372957-82769388-3ac7-40ae-b9e9-bd687dd71341.JPG" height="200px" width="300px"></p>

- 톱니 아이콘을 눌러 **SSH 사용**과 **사용자 이름 및 비밀번호 설정**을 본인이 원하는대로 지정한다.
  - 이걸 해야 키보드 연결하는 귀찮음을 피할 수 있음.
  - 사전에 RPI에 어떤 내부 아이피가 할당되는지 알고 있다면, 매우 편리함.
  - 만약, 본인 집에서 WIFI를 잡아 사용한다면, 아이피를 몰라도 공유기 페이지에서 어떤 아이피로 잡혔는지 알 수 있음.

- 설정까지 다 했다면 쓰기를 눌러 설치해주도록 한다.


<br/> <br/>
# 02. tensorflow 설치

해당 내용은 저의 오리지널이 아님을 밝힙니다.<br/>
여러 개발 블로그, 스택 오버플로우, 유튜브 등을 참고하여 정리하였습니다.
<hr> 

- 이거 모든 설치 처음에 반드시 해주는 게 좋음 !
  - 패키지 최신버전화
```powershell
  $ sudo apt update
  $ sudo apt full-upgrade
```

- 현재 설치된 아키텍처 확인
  - 32bit 운영체제는 armv7l이 뜰꺼고, 64bit 운영체제는 aarch64라고 뜸.
```powershell
  $ uname -m
```

- 본인의 프로젝트 폴더 생성
  - 폴더 위치는 별로 중요하지 않음.
```powershell
  $ cd Desktop
  $ mkdir project
  $ cd project
```

<br/>

- 가상환경 설치
  - 가상환경에 설치를 해야 삭제가 용이하다.
    - 그냥 통으로 설치하면 잘못되었을 때 포맷을 해야하는 경우가 생김...
  - 명령어를 보면 아나콘다와 유사한 것을 알 수 있음.
```powershell
  $ python3 -m pip install virtualenv
  $ python3 -m virtualenv env
  $ source env/bin/activate
```
- 참고로 가상환경에 들어갔다 나오는 명령어들은 다음과 같다.
```powershell
  $ source env/bin/activate    # 접속
  $ exec $SHELL                # 해제
```

<br/>

- 원하는 tensorflow 버전을 찾아보자
  - [링크](https://github.com/PINTO0309/Tensorflow-bin/tree/main/previous_versions)에 접속하면 다양한 tensorflow 버전을 제공하고 있다.
  - 근데, 현재 파이썬 버전과 아키텍처를 맞춰 설치해야 함.
  ```bash
  $ source env/bin/activate
  $ python -V                  # 파이썬 버전 확인
  $ uname -m                   # 아키텍처 확인
  ```
  - 파일명이 다음과 같다면 tensorflow-2.9.0을 설치하는데 python 3.8.x(cp38) 버전이 설치되어 있고, 64bit(aarch64) 운영체제여야 한다.
  ```
  download_tensorflow-2.9.0-cp38-none-linux_aarch64.sh
  ```
- 설치 파일
  - 많은 기술블로그에서 wget 커맨드를 이용해 설치를 했는데, 내용을 복사해서 붙여넣어도 무관하다.
  - 아래 내용의 3, 4번 라인을 복사해 터미널에 그대로 붙여넣는다.
    - 1번은 주석이라서 복사 안해도 됨.
  <p align="center"><img src="https://user-images.githubusercontent.com/33558083/218372960-4e4dbcd4-beb2-4959-aac0-b3be1816ef47.JPG" height="200px" width="700px"></p>
  - 나와 같이 python3.9에 tensorflow2.9.0을 64bit 운영체제에 설치하고자 한다면 복붙하세용.
  
  ```bash
  $ curl -L https://github.com/PINTO0309/Tensorflow-bin/releases/download/v2.9.0 -tensorflow-2.9.0-cp39-none-linux_aarch64.whl -o tensorflow-2.9.0-cp39-none-linux_aarch64.whl
  
  $ echo Download finished.
  ```
  <p align="center"><img src="https://user-images.githubusercontent.com/33558083/218372964-71e56b7f-31c1-49bb-b919-dc25d4d21a87.JPG" height="100px" width="500px"></p>

<br/>

- 설치
  - pip 명령어를 통해 설치
  - 꽤 오래걸림.
  ```bash
  $ ls                            # whl 파일명을 복사해서
  $ pip install [your-whl-file]   # 붙여넣으세용
  ```
  
<br/>

- 확인
    ```python
    $ source env/bin/activate
    $ python

    >>> import tensorflow as tf
    >>> tf.__version__
    ```

<br/><br/><br/>
****이 게시글은 2023-02-13에 최초 작성되었습니다.**