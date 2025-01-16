---
title: "[FPGA] Alchitry Au를 위한 세팅"
date: 2025-01-06 18:40:00 +09:00
categories: FPGA
description: Alchitry Lab과 Vivado 설치하기
pin: true
use_math: true
---

Alchitry에서 제공하는 [셋업 튜토리얼](https://alchitry.com/tutorials/setup/) 과정을 따라했다.
- 환경 OS: 리눅스

## 1. Alchitry Lab 설치하기

Alchitry Lab은 FPGA 개발을 위한 IDE로, 주로 Lucid 언어로 작성된다. 

Alchitry 사이트에서 [다운](https://alchitry.com/alchitry-labs/)을 받아 설치한다. 나는 터미널 입력으로 대신 설치했다.

```bash
CPU=$(dpkg --print-architecture)
wget https://github.com/alchitry/Alchitry-Labs-V2/releases/latest/download/alchitry-alchitry-labs_2.0.21_${CPU}.deb
sudo apt install ./alchitry-alchitry-labs_2.0.21_${CPU}.deb
```

## 2. Vivado 설치하기

Vivado는 Xilinx에서 개발한 FPGA 설계 IDE로 HDL로 작성이 가능하다.

Vivado 사이트에서 [다운](https://xilinx.com/support/download.html)을 받아서 설치한다. Alchitry 사이트의 설명에 따르면 2020.3 버전은 피해서 설치할 것을 권장한다. 저작권과 관련된 문제로 다운로드 시 반드시 계정 로그인이 필요하다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/vivado_install_version.png" alt="비바도 설치 버전" style="width: 80%">

나는 최신 버전의 2024.2의 linux용 bin 파일을 설치했다. 
아래 명령어를 통해 터미널에서 .bin파일에 대한 권한을 설정하고, 실행하여 설치한다.

```bash
chmod +x [파일_이름].bin  // 권한 수정
./[파일_이름].bin  // 실행
```
설치 시 이메일과 비밀번호는 다운로드 시에 입력한 계정과 동일하게 입력한다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/vivado_install1.png" alt="비바도 설치1" style="width: 80%">

설치는 [Vivado]-[Vivado ML standard]로 설정한다. Alchitry의 세팅 튜도리얼에 말하는 Webpack은 2020.2버전 이후로 더 이상 유효하지 않다고 한다.

따라서 설치 시 옵션은 이 [사이트](https://forum.alchitry.com/t/instructions-to-install-vivado-2024-1-to-used-with-au-board/1696)를 참고하여 세팅하였다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/vivado_install2.png" alt="비바도 설치2" style="width: 80%">

- (Optional) Vitis Model Composer 선택을 해제한다.
- (Optional) Vitis Embedded Development를 체크 해제한다.
- Device에서 7 Series 내부의 Artix-7만 체크(나머지는 체크 해제)

나의 경우에는 Vitis Embedded Development는 체크한 채로 설치를 진행했다. 

<img src="{{ site.baseurl }}/assets/img/post/FPGA/vivado_install3.png" alt="비바도 설치3" style="width: 80%">

설치 디렉토리의 경우에는 `/opt/Xilinx`로 설정했는데, 아래 과정을 통해 권한 설정을 수정했다.

```bash
sudo mkdir -p /opt/Xilinx   // 디렉토리 생성
sudo chown $USER /opt/Xilinx  // 권한 설정 변경
```

최종적인 설치 세팅은 아래와 같다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/vivado_install4.png" alt="비바도 설치4" style="width: 80%">

설치 후 실행이 안되어서 터미널로 실행했더니 에러 확인 가능했다.

- 터미널로 실행
```bash
source /opt/Xilinx/Vivado/2024.2/settings64.sh
vivado&
```

- 발생 에러
```
application-specific initialization failed: couldn't load file "libxv_commontasks.so" : libtinfo.so.5: cannot open shared object file: No such file or directory
```

libtinfo.so.5가 없어서 발생한 에러이니 이를 설치하고 링크를 걸어준다. 

```bash
sudo apt update
sudo apt install libtinfo-dev
sudo ln -s /lib/x86_64-linux-gnu/libtinfo.so.6 /lib/x86_64-linux-gnu/libtinfo.so.5
```

이제 프로그램을 다시 실행하면 열린다.

## 참고

Alchitry의 [튜토리얼](https://alchitry.com/tutorials/)
