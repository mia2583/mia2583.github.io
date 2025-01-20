---
title: "[Linux] VScode 설치와 세팅(Python, C++, CMake)"
date: 2025-01-10 22:20:00 +09:00
categories: Linux
description: VScode 설치와 파이썬, C/C++, CMake 사용을 설정한다.
pin: true
use_math: true
---

우분투에서 VScode를 설치한 후에 파이썬과 c/c++을 사용할 수 있도록 설정하고자 한다. 

## 1. Visual Studio Code 설치

Visual Studio Code는 마이크로소프트에서 개발한 소스 코드 편집기이다. [VScode 공식 사이트](https://code.visualstudio.com/download)에서 OS에 맞게 원하는 버전을 다운로드 한다. 
나는 우분튜용  .deb를 다운로드 했다.

- code_1.96.2-1734607745_amd64.deb

이제 터미널에서 다음 명령어로 설치한다.

```bash
sudo dpkg -i [vscode_파일_다운_위치]
```

## 2. 파이썬 환경 설정

<img src="{{ site.baseurl }}/assets/img/post/Linux/python_version.png" alt="파이썬 버전" style="width: 80%">

우분투 20.04 LTS에는 파이썬이 자동으로 설치되어 있어 따로 설치할 필요는 없지만 나중에 파이썬을 최신 버전으로 업데이트하고 싶다면 ` sudo apt-get upgrade python3`을 터미널에 입력하여 업데이트할 수 있다.

그리고 pip는 설치되어 있지 않으므로 아래의 명령어로 설치 할 수 있다.

```bash
sudo apt install python3-pip
```

> Linux 명령어?  
> - `apt-get install`  
> 특정 패키지를 설치한다  
> 설치나 업데이트에 자주 사용하는 아래 명령어들을 역할을 살펴본다.  
> - `apt-get update`  
> 최신 패키지 목록을 업데이트 한다.
> - `apt-get upgrade`  
> 업데이트된 패키지 목록 중 설치된 패키지들을 최신버전으로 업데이트한다. 그 목록은 `apt list --upgradable`로 확인할 수 있다.   
> - `pip3`  
> 파이썬3 패키지 관리 툴로, 파이썬에서 사용하는 라이브러리나 패키지를 설치, 엡그레이트, 제거할 때 사용된다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/python_extension.png" alt="파이썬 extension" style="width: 80%">

마지막으로 vscode의 extension에서 python을 설치해주면 파이썬 사용 준비가 완료된다.
- 설치 시 Pylance, Python, Python Debugger이 설치된다.

## 3. C/C++ 환경 설정

C/C++ 파일을 빌드하기 위해서는 컴파일러가 필요하다. 여러 컴파일러중 GCC(GNU Compiler Collection)을 선택해 설치하고자 한다. GCC만으로 C/C++ 파일 빌드가 가능하지만 복잡한 프로젝트에서는 make, CMAKE 등을 함께 사용하면 파일들을 효율적으로 관리할 수 있다.  

```bash
sudo apt update
sudo apt install build-essential
```

- build-essential 패키지는 GCC를 비롯해 make, CMAKE 등과 같이 C/C++ 컴파일을 위한 도구들을 포함한다.

설치가 완료되면 다음 명령어를 통해 C/C++파일을 빌드할 수 있다.

```bash
gcc [소스파일_이름].c -o [출력파일_이름]  // C 컴파일
g++ [소스파일_이름].cpp -o [출력파일_이름]  // C++ 컴파일
```

이제 설치한 GCC를 VScode에도 적용해보자.

### 3-1. GCC 설정

GCC는 다양한 프로그래밍 언어를 지원하는 컴파일러 모음이다. 소스코드를 컴파일하여 실행 가능한 파일을 생성한다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/cpp_extension.png" alt="C++ extension" style="width: 80%">

일단은 파이썬과 마찬가지로 C/C++ extension을 설치해준다. 

<img src="{{ site.baseurl }}/assets/img/post/Linux/cpp_build.png" alt="c++ 빌드" style="width: 80%">

이후 C++ 코드를 작성한 후에 [ctrl+shift+b]-[C/C++ g++ build active file]을 선택하면 기본 tasks.json 파일을 바탕으로 빌드가 되어 실행 파일을 생성한다. 

<img src="{{ site.baseurl }}/assets/img/post/Linux/edit_tasksjson.png" alt="tasks.json 수정" style="width: 80%">

tasks.json 파일에 내용을 변경하여 빌드에 대한 설정을 수정할 수 있다. tasks.json 파일은 [ctrl+shift+p]-[create tasks.json from template]-[others]를 통해서 생성할 수 있다. 

### 3-2. CMAKE 설정

CMake는 소스 코드의 빌드 과정을 자동화하는 도구이다. Makefile을 통해 컴파일할 소스파일을 정의하고, 필요한 라이브러리들을 링크할 수 있다.

`sudo apt install cmake`로도 CMake를 설치할 수 있지만, 최신 버전으로 설치가 되지 않는다고 하여 소스코드로 직접 설치하고자 한다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/cmake_install.png" alt="tasks.json 수정" style="width: 80%">

[CMake 공식 사이트](https://cmake.org/download/)에서 OS에 맞춰 원하는 버전을 다운로드 한다.
- 나의 경우에는 2025.1 기준 Latest Release인 3.31.4를 다운로드하였다.

다운로드 된 tar.gz 파일을 압축 해제 한 후, 설치를 진행한다.

```bash
tar -xvzf cmake-3.31.4.tar.gz  // 압축 해제

cd cmake-3.31.4/
./bootstrap
make
sudo make install
```

설치 중 다음과 같은 에러가 뜬다면 `sudo apt-get install libssl-dev`를 설치해준다.
```
CMake Error at Utilities/cmcurl/CMakeLists.txt:772 (message):
  Could not find OpenSSL.  Install an OpenSSL development package or
  configure CMake with -DCMAKE_USE_OPENSSL=OFF to build without OpenSSL.
```

설치가 완료되면 버전 출력으로 정상적으로 설치가 되었는지를 확인한다.

```bash
cmake --version
```

VScode에서 CMake를 사용하기 위해선 [extension]에서 'CMake'와 'CMake Tools'를 설치해준다. 

## 4. Git 설치

git은 소프트웨어 개발에 사용되는 분산 버전 관리 시스템으로, 코드의 변경 내역을 기록하고 관리할 수 있다. 

먼저 git을 설치한 후에 버전 출력 통해 제대로 설치되었는지 확인한다.

```
sudo apt-get install git
git --version  // 나의 경우엔, 2.25.1
```

git에서 사용할 이메일과 이름을 등록한다. 이는 github의 이메일과 이름을 동일하게 사용하는 것을 추천한다.

```
git config --global user.email [이메일_주소]
git config --global user.name [유저_이름]
git config -l // 정보 확인
```

원하는 레포지토리를 클론하고, 기존의 브랜치에 새로운 커밋을 작성해본다.

```
git clone [레포지토리_주소]
git branch -r  // 원격 저장소에 존재하는 브랜치 리스트 확인
git checkout -t [브랜치명]  // 원하는 브랜치 가져오기

git add .  // 수정한 파일 전체 포함하기
git commit -m "[커밋_메세지_작성]"
git push origin [브랜치명]
```

이제 깃허브에서 확인하면 변경사항이 커밋된 것을 확인 할 수 있다. 처음 push를 시도하면 로그인을 해야한다. 하지만 2021년 8월 이후로 아이디, 비번이 아닌 토큰 형식의 로그인만을 지원한다.   

```
remote: Support for password authentication was removed on August 13, 2021.
remote: Please see https://docs.github.com/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication.
```

<img src="{{ site.baseurl }}/assets/img/post/Linux/github_setting.png" alt="깃허브 세팅" style="width: 80%">

토큰을 발급받기 위해서는 먼저 깃허브에 로그인한 후 [setting]-[Developer setting]-[Personal access tokens]-[Tokens (classic)]에 들어가, [Generate new token]을 선택한다.

인증 정보를 디스크에 영구적으로 저장하기 위해(git 자동 로그인 설정을 위해) 아래 명령어를 입력한 후, 로그인을 시도한다.
```
git config --global credential.helper store
```
로그인 정보는 ~/.git-credentials 파일에 저장된다.

이후 생성된 토큰을 비밀번호를 대신하여 입력한다. (생성 후 한번만 보여지므로 주의한다.)

## 참고

Visual Studio Code의 [C/C++ for Visual Studio Code](https://code.visualstudio.com/docs/languages/cpp)

인공능지연구소 님의 [Ubuntu에서 CMake 설치하는 방법](https://velog.io/@letnilab/Ubuntu%EC%97%90%EC%84%9C-CMake-%EC%84%A4%EC%B9%98%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)

서쿠 님의 [\[Git\] Credential Helper로 GitHub 인증하기](https://velog.io/@euisuk-chung/Git-Credential-Helper%EB%A1%9C-GitHub-%EC%9D%B8%EC%A6%9D%ED%95%98%EA%B8%B0)
