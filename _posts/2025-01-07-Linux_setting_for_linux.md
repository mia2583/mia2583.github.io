---
title: "[Linux] 우분투 유용 기능 설정"
date: 2025-01-07 11:30:00 +09:00
categories: 리눅스
description: 한글 입력, 듀얼 부팅내의 디스크 공유 방법에 대해 작성한다.
pin: true
use_math: true
---

## 1. 한글 입력 세팅하기

나와 동일한 경우의 사람들을 위해서 처음 시도해서 실패한 방법도 함께 작성한다. 1-2부터 바로 시도하는 것을 추천한다.

## 1-1. 실패한 방법

우분투 언어를 영어로 설치하게되면 한글 입력이 되지 않는다. 한글을 입력할 수 있도록 세팅하고자 한다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/inputSource_add.png" alt="언어 추가하기" style="width: 50%">

[setting]-[Region & Language]-[Input Sources]-[+] 에서 Korean을 추가해준다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/add_korean.png" alt="한국어 추가" style="width: 80%">

나의 경우에는 이미 한국어가 설치되어 있는 상태였는데, 만약 설치가 안된 상황이라면 [Manage Installed Languages]-[Install/Remove Languages]를 통해 한국어를 설치할 수 있다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/inputSource_option.png" alt="언어 추가하기" style="width: 60%">

`window + space`을 입력으로 한글/영어로 바꿀 수 있다.

한국어로의 변환 키 입력 방법은 [setting]-[Keyboard Shortcut]-[Typing]에서 바꿀 수 있다.

## 1-2. 해결 방안

한국어로 변환은 되지만 입력은 여전히 영어로 되기에 찾아봤더니 Korean이 아니라 Hangul로 설치해야한다고 한다.

나의 경우에는 재부팅 후에 아래 작업을 제대로 수행할 수 있었다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/ibus_add_hangul.png" alt="한글 추가하기" style="width: 80%">

터미널에 `ibus-setup` 입력 후 뜨는 [IBus Preferences] 창에서 [Input Method]-[Add]-[Hangul]를 한다. [English]에 대해서도 동일하게 추가해준다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/inputSource_add_hangul.png" alt="한글 설정" style="width: 80%">

이후 1-1과 동일한 과정을 거치나 `Korean`이 아니라 `Korean (Hangul)`로 설치한다. 

이제 한글이 입력된다!!!!!

## 2. 듀얼부팅 간의 디스크 공유하기(미해결)

현재 C 드라이브에 윈도우와 리눅스가 같이 설치가 되어 있다. D 드라이브의 파일을 윈도우와 리눅스에서 모두 접근 가능하도록 하려고 한다. 

윈도우에서는 드라이브를 C, D와 같이 표현하지만, 리눅스에서는 /dev/sda, /dev/sdb와 같이 이름을 사용한다. 

리눅스에서 디스크 앱을 실행한다. 
현재 나의 디스크 이름과 상황은 아래와 같다.
- C 드라이브 <-> /dev/nvmeOn1 (512GB)
- D 드라이브 <-> /dev/sda (256GB)  : Not mounted

<img src="{{ site.baseurl }}/assets/img/post/Linux/edit_mount_options.png" alt="디스크 마운트 옵션 수정" style="width: 80%">

공유를 원하는 드라이브의 파티션을 선택한 후, 톱니바퀴 모양의 버튼을 클릭하여 Edit Mount Options를 클릭한다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/disk_mount_setting.png" alt="디스크 마운트 세팅" style="width: 80%">

아래와 같이 설정 후 노트북을 재부팅해준다.
- User Session Default 해제
- Mount at system startup에 체크
- Mount Point 바로위에 옵션에 rw를 추가 입력해준다.
- Mount Point 에서 디스크를 마운트 할 경로를 지정해준다.

재부팅 시에 D 드라이브에서 데이터를 삭제하거나 저장하려고 하면 권한의 이유로 실패한다. 나의 경우에는 디스크가 NTFS 파일 시스템이라 리눅스 환경에서 `chmod` 또는 `chown` 명령어로 권한 수정이 불가능하다..([참고](https://ubuntuforums.org/showthread.php?t=2480444))

따라서 윈도우에서 폴더 생성 시 권한을 미리 열어놔야 한다. 디스크 자체의 권한을 수정할 수도 있지만 나는 그냥 특정 몇 폴더에 대해서만 권한을 열려고 한다.

- [폴더에서 오른쪽 마우스]-[일반]-[읽기 전용 체크 해제]
- [폴더에서 오른쪽 마우스]-[보안]-[편집]-[Users 선택]-[모든 권한 허용]

와하하 아직 해결중....

## 참고

세종정조님의 [Ubuntu 24.04 한글 입력기 설정 (한영 변환 설정)](https://andrewpage.tistory.com/390)

케라비님의 [ubuntu 20.04와 윈도우 파일 공유하기](https://keravi.tistory.com/18)



