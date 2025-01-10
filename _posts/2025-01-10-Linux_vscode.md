---
title: "[Linux] VScode 설치와 세팅(python, C++)"
date: 2025-01-10 22:20:00 +09:00
categories: Linux
description: VScode 설치와 파이썬, c/c++ 사용을 설정한다.
pin: true
use_math: true
---

우분투에서 VScode를 설치한 후에 파이썬과 c/c++을 사용할 수 있도록 설정하고자 한다. 

## 1. Visual Studio Code 설치

[VScode 공식 사이트](https://code.visualstudio.com/download)에서 OS에 맞게 원하는 버전을 다운로드 한다. 
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
sudo apt-get install pip3
```

// 이미지

- apt-get update
- apt-get upgrade
- apt-get install
- pip3

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

## 3-1. GCC 설정

<img src="{{ site.baseurl }}/assets/img/post/Linux/cpp_extension.png" alt="C++ extension" style="width: 80%">

일단은 파이썬과 마찬가지로 C/C++ extension을 설치해준다. 

<img src="{{ site.baseurl }}/assets/img/post/Linux/cpp_build.png" alt="c++ 빌드" style="width: 80%">

이후 C++ 코드를 작성한 후에 [ctrl+shift+b]-[C/C++ g++ build active file]을 선택하면 기본 tasks.json 파일을 바탕으로 빌드가 되어 실행 파일을 생성한다. 

<img src="{{ site.baseurl }}/assets/img/post/Linux/edit_tasksjson.png" alt="tasks.json 수정" style="width: 80%">

tasks.json 파일에 내용을 변경하여 빌드에 대한 설정을 수정할 수 있다. tasks.json 파일은 [ctrl+shift+p]-[create tasks.json from template]-[others]를 통해서 생성할 수 있다. 

## 3-2. CMAKE 설정



## 참고



