---
title: "[Linux] LG 그램에 Ubuntu 듀얼 부팅 설정하기"
date: 2025-01-05 18:40:00 +09:00
categories: Linux
description: LG 그램에 Ubuntu를 설치하여 듀얼 부팅을 시도한다.
pin: true
use_math: true
---

대부분의 프로그램을 윈도우에서 개발하다가 중고 노트북을 구매하여 개인 리눅스 개발 환경을 세팅하고자 한다. VMware의 경우에는 속도가 느려 노트북에 듀얼 부팅으로 리눅스를 설치하기로 하였다.

- 사용한 노트북 : LG 그램 13Z990-A.AAS5UI (포맷된 중고 노트북)
- 상태 : C 드라이브에 윈도우 설치되어 있음
- 목표 : C 드라이브에 우분투 설치하여 듀얼 부팅하고자 함
- 설치 버전 : Ubuntu 20.04 LTS

## 1. USB 부팅 디스크 만들기

빈 USB 내에 원하는 버전의 Linux ubuntu iso를 다운로드 한다.

[https://releases.ubuntu.com/](https://releases.ubuntu.com/)
- ubuntu-20.04.6-desktop-amd64.iso 다운

USB 부팅 디스크를 생성하기 위해서 윈도우용 부팅 디스크 생성 도구인 Rufus를 설치한다. Rufus 대신에 balenaEtcher과 같은 프로그램 사용해도 가능하다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/rufus.JPG" alt="Rufus 설치" style="width: 60%">

[https://rufus.ie/ko/](https://rufus.ie/ko/)
- 최신 버전인 rufus-4.6.exe 다운

Rufus 다운로드가 완료되면 다운 받은 Ubuntu 이미지 파일을 선택해 디스크를 구워준다.

- 장치 : USB 선택하기
- 부트 선택 : 다운 받은 iso
- 선택
모든 설정 후 [시작] 누르기

원래라면 정상적으로 완료되어야 했으나...선택한 USB에서 다운된 이미지가 인식되지 않는 오류가 발생하여 balenaEtcher로 대신 사용하였다. 
balenaEtcher에서는 아래와 같이 이미지 URL을 통해 쉽게 USB를 구울 수 있다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/balenaEtcher.JPG" alt="balenaEtcher 설치" style="width: 60%">

## 2. 부팅 설정과 용량 설정하기

C 드라이브에 윈도우와 우분투를 함께 설치하기 위해서는 C 드라이브 내에 우분투 용량을 별도로 할당해야 한다. 별도의 드라이브에서 해당 드라이브 전체 우분투용으로 사용할 예정이라면 2-1 단계는 생략 가능하다.
그리고 우분투를 설치하기 전에 노트북을 부팅 시에 USB로 접근 가능하도록 설정해야 한다. 

### 2-1. 드라이브 파티션 준비

드라이브의 일부를 우분투 설치에 할당하고 나머지를 개인 파일 저장 용도로 사용하고자 하는 경우에 아래와 같이 설정한다.

window 검색창에 [디스크 관리]를 입력 후 [하드디스크 파티션 만들기 및 포맷]을 선택한다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/disk_partition.JPG" alt="디스크 파티션" style="width: 60%">

파티션 하고자하는 디스크 공간에서 [우클릭] - [볼륨 축소] 하여 원하는 용량 입력 후 축소하면 된다. 설정 시 원하는 용량만큼 `할당되지 않음`으로 표시된다.
나의 경우에는 220GB를 우분투에 사용할 예정이다.

### 2-2. USB로 부팅하기

메인보드마다 설정이 다르지만, LG 그램의 경우에는 전원 종료 후 재시작 시 LG 그램 로고가 보일 때 `F2`를 연타하면 BIOS로 진입이 가능하다.
BIOS 진입 후 아래와 같이 설정을 수정한다.

- [Security] - [Secure Boot Configuration] 에서 [Secure Boot Option]을 off로 설정
- [Boot] 에서 USB를 최우선으로 설정 (나의 경우에는 USB HDD를 최우선 순위로 두었다.)

설정이 모두 완료되면 저장 후에 다시 재부팅을 시작한다. 

## 3. 우분투 설치하기

[Try or Install Ubuntu]를 선택하여 우분투 설치를 시작한다. 나의 경우에는 [Ubuntu] 옵션으로 떴는데, 이 경우에도 동일하게 작동된다.

우분투 설치 언어는 영어를 권장하며 (폴더명 등이 영어로 설정되어야지 오류가 적어진다) 나중에 한글 입력 추가 설정이 가능하다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/install_thirdparty.JPG" alt="third party 설정" style="width: 60%">

[Updates and other software]에서는 third-party 설치를 체크하는 것을 추천한다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/install_type.JPG" alt="파티션 설정" style="width: 60%">

요구사항에 맞게 설정하다가 [Installation type]에서 [Something else]를 선택한다. 해당 옵션은 윈도우가 이미 설치되어 있을 때, 우분투 설치를 위한 별도의 디스크 파티션 설정을 의미한다.

2-1에서 설정한 크기의 `free space`를 선택 후 아래 [+]를 선택하여 아래와 같이 새로운 파티션을 추가해준다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/install_swap.JPG" alt="swap 크기 설정" style="width: 60%">

swap은 물리적 RAM이 부족할 때 디스크 일부를 가상 메모리로 활용하는 것으로, 옛날 RAM 크기가 작았을 때에는 기존 RAM 2배 크기로 할당을 권장했다. 하지만 나의 경우 RAM의 크기가 20GB이므로 4GB정도만 할당하기로 한다.

<img src="{{ site.baseurl }}/assets/img/post/Linux/install_swap.JPG" alt="swap 크기 설정" style="width: 60%">

다시 동일한 `free space`에서 [+]를 선택하여 주 파티션을 설정해준다. 이는 ubuntu 운영체제과 필수 시스템 파일이 저장되며, 어플리케이션 설치시에도 사용하는 공간이다. 나머지 전체 부분을 할당하고 파일 시스템을 Ext4로, mount point를 `/`로 설정한다.

마지막로 부트로더 설치 위치를 설정해야 한다. 우분투를 설치하고자하는 디스크와 동일한 디스크를 선택한다.

> window boot manager가 2개?  
> 나의 경우에는 중고 노트북을 사용했는데, 부트로더 설치 위치를 선택하고자 하니, window boot manager가 각각 드라이브에 하나씩 총 2개 였다.  
> 구글링 결과, 기존의 사용자가 윈도우를 새로 설치하면서 기존의 윈도우가 제대로 삭제되지 않은 경우 혹은 다른 드라이브에 백업용으로 설치된 경우에 그럴 수 있다고 한다.  
> 우분투 설치 디스크와 동일한 디스크를 선택해 부트로더를 설치하니 별도의 문제가 생기진 않았다. 

나머지는 사용자 스스로에 맞게 설정해주면 설치를 완료할 수 있다.

## 4. 설치 확인
우분투 설치가 완료되면 노트북을 다시 재부팅 시킨다. 그러면 grub으로 ubuntu로 부팅을 할 것인지, window로 부팅을 할 것인지 선택할 수 있다.

나의 경우, 모두 정상적으로 작성했지만 혹시 몰라서 boot-repair을 설치해서 재확인했다.

## 4. window에서 부팅 디스크 포맷
USB를 부팅 디스크로 설정하면서 파티션이 나눠진 상태이다. 이를 다시 원래의 USB로 되돌리기 위해서는 아래와 같이 포맷을 진행할 수 있다.
- cmd를 관리자 권한으로 실행 후 `diskpart`를 실행
- `list disk`
- USB 선택하기 `ex) select disk x`
- `attributes disk clear readonly`
- `list disk`
- USB 선택하기 `ex) select disk x`
- clean

<img src="{{ site.baseurl }}/assets/img/post/Linux/usb_format.JPG" alt="usb 포맷" style="width: 80%">

이후 디스크 관리 페이지에서 [새 단순 볼륨]을 선택해 USB를 초기화해준다.

## 참고

robotcelia 님의 [Ubuntu 18.04/20.04/22.04 듀얼부팅 설치](https://robotcelia.tistory.com/7)

AI는눈곱만큼도모름 님의 [리눅스 우분투 다운로드와 usb 파티션 삭제](https://ashton0410.tistory.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%9A%B0%EB%B6%84%ED%88%AC-%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C%EC%99%80-%EC%84%A4%EC%B9%98-%EA%B7%B8%EB%A6%AC%EA%B3%A0-usb-%ED%8C%8C%ED%8B%B0%EC%85%98-%EC%82%AD%EC%A0%9C)

