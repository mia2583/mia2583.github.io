---
title: "[FPGA] 첫 프로젝트 - led 예제"
date: 2025-01-09 12:40:00 +09:00
categories: FPGA
description: 
pin: true
use_math: true
---

## 1. 프로젝트 생성

Architry Labs을 통해서 LED를 동작하는 프로젝트를 만들어본다.

// 이미지

[create Project]를 클릭해 아래 설정으로 프로젝트를 생성한다.
- Board : Alchitry Au v1
- template : 기본 (이 튜토리얼 완성하면 button to led와 동일한 템플릿이 완성된다)

## 2. 코드 분석

source 내부의 최상위 파일인 `board_top.luc`을 찾아서 연다. 이 파일에는 모든 외부와의 입력과 출력이 설정되어 있다.

기본적으로 제공되는 코드를 살펴보면 다음과 같은 코드를 찾을 수 있다.
```
module alchitry_top (
    input clk,              // 100MHz clock
    input rst_n,            // reset button (active low)
    output led [8],         // 8 user controllable LEDs
    input usb_rx,           // USB->Serial input
    output usb_tx           // USB->Serial output
  ) {
```
- 모듈 이름 : alchitry_top (모듈의 이름과 파일 이름은 동일해야 한다)
- 단일 비트 신호 : led는 8비트 (보드에 led가 총 8개이며 각 비트는 각 보드 led에 대응된다)
- rst_n : 리셋 버튼 (기본적으로 1이며 버튼 누르면 0이 되는 active low)

다음으로 always 블록은 계산을 수행하고 신호를 읽고 쓸 수 있다. FPGA 보드는 always에서 정의하는 동작을 복제하는 디지털 회로를 만든다.

```
always {
  reset_cond.in = ~rst_n; // input raw inverted reset signal
  rst = reset_cond.out;   // conditioned reset
  led = 8h00;             // turn LEDs off
  usb_tx = usb_rx;        // echo the serial data
}
```
블록 내부에서는 뒤에 적힌 명령문이 앞의 명령어보다 우선순위가 높다. 

```
always {
  led = 8h00;   //  LED off
  led = 8hFF;   //  LED on
}
```

예를 들어, 위와 같이 정의된 always 블록의 동작은 프로그래밍 코드처럼 두 동작이 순차적으로 일어나는 것이 아니라 led가 항상 켜진 동작을 의미한다. 


// 설명 추가

## 3. 예제

이제 리셋 버튼을 첫 번째 LED에 연결하고, 버튼을 누르면 LED가 켜지도록 코드를 작성해보자. always 블록 내부에서 `led = 8h00`를 찾아서 아래와 같이 수정한다. 

#### 방법1

```
led = c{7h00, rst};     // connect rst to the first LED
```

8비트 배열의 LED를 단일 비트인 rst에 연결하기 위해서는 concatenation operator인 `c{x, y, z}를 사용한다. 

> Concatenation operator?
> x, y, z가 하나의 배열을 형성
> 
// 추가 설명

#### 방법2

혹은 LED의 배열를 부분적으로 나눠서 할당 시키는 방법으로도 동일한 역할의 코드를 작성할 수 있다.

```
led[7:1] = 7h0;    // LED off
led[0] = rst;      // connect rst to led[0]
```

> 인덱싱?  
> 
```
led[7:1] = 7b0;  // select bits 7-1
led[7-:7] = 7b0; // select 7 bits starting from 7 and going down
led[1+:7] = 7b0; // select 7 bits starting from 1 and going up
```  
> 

#### 방법3

```
led = 8b0;              // turn the LEDs off
led[0] = rst;           // connect rst to led[0]
```

### 3-1. 프로젝트 빌드

툴바에 망치 아이콘을 클릭하여 프로젝트를 빌드할 수 있다. Finished building project가 뜨면 빌드가 완료된 것이나 혹시 에러가 뜨게되면 오류의 원인을 찾아 수정해야 한다. 

> Vivado Location? 
> [Menu]-[Settings]-[Set Vivado Location]에서 비바도 폴더를 설정한다. 예를 들어 설치 버전이 2024.2이면 `Xilinx/Vivado/2024.2` 폴더로 지정한다.

## 3-2. 프로젝트 로드

프로젝트 빌드가 완료되면 이제 보드를 컴퓨터에 연결한 후, 보드에 프로젝트를 로드해야 한다.

// 이미지

- 빈 화살표 : RAM에 쓰기 (전원을 끄면 자동 소실)
- 실선 화살표 : 보드의 FLASH 메모리에 쓰기 (보드 재부팅 시 configuration 자동 로드)
- 지우개 : FLASH 메모리 비우기

로드 완료되면 DONE LED는 항상 켜져있고, Reset 버튼을 누르는 동안 첫번째 led의 불빛이 켜진다.

### 3-3 결과


## 4. 하드웨어의 특징

위의 디자인의 경우, FPGA는 프로세서와 달리 버튼의 입력이 LED 출력에 직접 연결된다. 즉, 루프에 추가된 코드의 양과 상관없이 물리적 와이어에 의해서만 속도가 결정이 된다. 나머지 FPGA의 디자인 변경되더라도 LED와 버튼의 회로와는 독립적으로 동작하기 때문에 속도에 영향을 주지 않는다. 

## 5. 최종 코드 파일

## 참고