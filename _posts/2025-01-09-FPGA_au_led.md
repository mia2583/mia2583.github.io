---
title: "[FPGA] 첫 프로젝트 - led 예제"
date: 2025-01-09 12:40:00 +09:00
categories: FPGA
description: Alchitry Au 보드를 사용해 led를 켜본다.
pin: true
use_math: true
---

## 1. 프로젝트 생성

Architry Labs을 통해서 LED를 동작하는 프로젝트를 만들어본다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_project.png" alt="alchitry 프로젝트 생성" style="width: 80%">

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

다음으로 always 블록은 계산을 수행하거나 신호를 읽고 쓸 수 있다. FPGA 보드는 always에서 정의하는 동작을 복제하는 디지털 회로를 만든다.

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

## 3. 예제

이제 리셋 버튼을 첫 번째 LED에 연결하고, 버튼을 누르면 LED가 켜지도록 코드를 작성해보자. always 블록 내부에서 `led = 8h00`를 찾아서 아래와 같이 수정한다. 

#### 방법1

```
led = c{7h00, rst};     // connect rst to the first LED
```

8비트 배열의 LED를 단일 비트인 rst에 연결하기 위해서는 concatenation operator인 `c{x, y, ..., z}를 사용한다. 

> Concatenation operator?
> 여러 비트를 연결하여 하나의 값으로 합치는 연산자 이다. 예를 들어. c{3b101, 2b11}은 5b10111을 의미한다. 따라서 위의 예제에서 c{7h00, rst}는 7개의 0과 rst값으로 연결된 8비트를 의미한다.

#### 방법2

LED의 배열를 부분적으로 나눠서 할당 시키는 방법으로도 동일한 역할의 코드를 작성할 수 있다.

```
led[7:1] = 7h0;    // LED off
led[0] = rst;      // connect rst to led[0]
```

> 인덱싱?  
> 배열의 인덱스는 0부터 시작하며, 배열의 슬라이싱을 표현하는 방법으로는 총 3가지가 있다. 첫 번째는 `배열명[상위비트:하위비트]`, 두 번째는 `배열명[시작비트-:개수]`, 마지막으로 `배열명[시작비트+:개수]`으로 표현할 수 있다. 아래는 모두 1부터 7까지 슬라이싱하는 예이다.  
> 
```
led[7:1] = 7b0;  // select bits 7-1
led[7-:7] = 7b0; // select 7 bits starting from 7 and going down
led[1+:7] = 7b0; // select 7 bits starting from 1 and going up
```  

#### 방법3

뒤의 명령문이 우선순위가 더 높은 always 블록의 특성을 이용한 것이다.

```
led = 8b0;              // turn the LEDs off
led[0] = rst;           // connect rst to led[0]
```

led[0]=rst의 우선 순위가 더 높기 때문에 0번째 비트는 0이 아닌 rst 값으로 부여된다. 

### 3-1. 프로젝트 빌드

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_build_success.png" alt="alchitry 빌드 성공" style="width: 80%">

툴바에 망치 아이콘을 클릭하여 프로젝트를 빌드할 수 있다. Finished building project가 뜨면 빌드가 완료된 것이나 혹시 에러가 뜨게되면 오류의 원인을 찾아 수정해야 한다. 

> Vivado Location? 
> [Menu]-[Settings]-[Set Vivado Location]에서 비바도 폴더를 설정한다. 예를 들어 설치 버전이 2024.2이면 `Xilinx/Vivado/2024.2` 폴더로 지정한다.

## 3-2. 프로젝트 로드

프로젝트 빌드가 완료되면 이제 보드를 컴퓨터에 연결한 후, 보드에 프로젝트를 로드해야 한다.

- 빈 화살표 : RAM에 쓰기 (전원을 끄면 자동 소실)
- 실선 화살표 : 보드의 FLASH 메모리에 쓰기 (보드 재부팅 시 configuration 자동 로드)
- 지우개 : FLASH 메모리 비우기

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_load_success.png" alt="alchitry 로드 성공" style="width: 80%">

### 3-3 결과

로드 완료되면 DONE LED는 항상 켜져있고, Reset 버튼을 누르는 동안 첫번째 led의 불빛이 켜진다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_ex1.png" alt="alchitry ex1" style="width: 80%">

## 4. 하드웨어의 특징

위의 디자인의 경우, FPGA는 프로세서와 달리 버튼의 입력이 LED 출력에 직접 연결된다. 즉, 루프에 추가된 코드의 양과 상관없이 물리적 와이어에 의해서만 속도가 결정이 된다. 나머지 FPGA의 디자인 변경되더라도 LED와 버튼의 회로와는 독립적으로 동작하기 때문에 속도에 영향을 주지 않는다. 

## 5. 최종 코드 파일

ex1 led 예제의 [alchitry_top.luc](https://github.com/mia2583/alchitry/blob/main/ex1_led/alchitry_top.luc)

## 참고

Alchitry의 [Your First FPGA Project](https://alchitry.com/tutorials/lucid_v1/your-first-fpga-project/) 