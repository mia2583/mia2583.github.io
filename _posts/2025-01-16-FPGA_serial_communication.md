---
title: "[FPGA] Alchitry Au - 시리얼 통신 예제"
date: 2025-01-16 15:20:00 +09:00
categories: FGPA
description: 시리얼 통신으로 데이터를 송, 수신한다.
pin: true
use_math: true
---

USB 포트를 사용하여 Au 보드와 컴퓨터 사이에서 데이터를 주고 받아보고자 한다. Au 보드의 USB 포트는 FTDI USB를 통해 시리얼 통신이 가능하다. 

> 시리얼 통신?  
> 데이터를 한 번에 한 비트씩 전송하는 데이터 전송 방식을 의미하며, 종류로는 USART, UART, $I^2C$ 등이 있다. 

> UART(Universal Asynchronous Receiver Transmit)?  
> 시리얼 통신을 지원하는 하드웨어 모듈로, USART와 달리 비동기식 통신만을 지원한다. 

## 1. UART 예제

### 1-1. 컴포넌트 추가하기

컴포넌트(Component)는 미리 정의된 모듈로, 이번 튜토리얼에서는 두가지 컴포넌트(데이터 수신, 데이터 송신)를 사용할 예정이다.

Alchitry lab에서 템플릿 'Base Projec-Minimun boilerplate project for ALchitry boards'으로 새로운 프로젝트를 생성한다. 그리고 컴포넌트를 추가해준다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_menu.png" alt="alchitry 메뉴" style="width: 70%">

<img src="{{ site.baseurl }}/assets/img/post/FPGA/component_ibrary.png" alt="컴포넌트 추가" style="width: 70%">

Alchitry lab의 툴바에서 블록 세개짜리 아이콘(add component)를 클릭한 후 [Component Library]-[Interface]에서 UART Rx와 UART Tx를 추가해준다.

컴포넌트를 사용하기 위해서는 우선 `alchitry_top.luc`에서 정의를 해주어야 한다. 

```alchitry_top.luc
.clk(clk) {
  
  ...
 
  .rst(rst) {
    uart_rx rx (#BAUD(1000000), #CLK_FREQ(100000000)); // serial receiver
    uart_tx tx (#BAUD(1000000), #CLK_FREQ(100000000)); // serial transmitter
  }
}
```

`BAUD` 파라미터는 초당 전송할 비트 수를 의미하여, 연결된 컴퓨터와 이 수를 맞추어야 통신할 수 있다. 
`_CLK_FREQ_`는 클록의 주파수로, 비트당 필요한 클록 사이클 수를 계산한다. 

### 1-2. 데이터 송/수신하기
이제 정의된 rx, tx 모듈을 FPGAq보드의 입력, 출력 신호에 연결한다.

```alchitry_top.luc
always {
  ...

  led = 8h00;
 
  rx.rx = usb_rx;         // connect rx input
  usb_tx = tx.tx;         // connect tx output
 
  tx.new_data = rx.new_data;
  tx.data = rx.data;         
  tx.block = 0;           // no flow control, do not block
}
```

`tx.block`은 UART의 송신 동작의 `tx.busy`를 확인하는 설정이다. tx.block이 1이면, 데이터가 완전히 송신되기까지 다음 작업을 멈추고, 0이면 송신 완료 여부를 확인하지 않고 계속해서 다음 작업을 이어간다. 

송신할 데이터는 tx.data에 저장되며, 송신 준비가 완료되면 tx.new_data가 1로 활성화된다. 수신 역시 마찬가지로 수신할 데이터가 전달되면 rx.new_data가 1로 활성화되고, rx.data에 수신한 데이터를 저장한다. 

따라서 위의 코드는 컴퓨터에서 송신한 데이터를 FPGA 보드에서 데이터를 받고(rx), 이를 다시 컴퓨터쪽으로 송신(tx) 해준다. 

## 1-3. 송신된 데이터를 보드에서 사용하기

그러면 마지막으로 수신된 데이터에 따라 보드의 LED도 함께 켜지도록 해보자. clk 블록 내부면서 rst 블록 외부인 곳에 D 플립플롭 하나를 정의해준다.
그리고 이 D 플립플롭에 수신된 데이터의 저장한다.

```alchitry_top.luc
{
    ...
    .clk(clk){
        ...
        dff data[8];

        .rst(rst) { ... }
    }
}

always {
    ...

    if(rx.new_data)
        data.d = rx.data;

    led = data.q;
}
```

`char`의 크기가 1바이트(8비트)이므로 수신된 캐릭터가 D 플립플롭에 저장이 된다. 그리고 이는 8비트의 LED의 값에 부여된다.

### 1-4. 결과

Alchitry Lab에서 [Tools]-[Serial Terminal]에서 보드와 BAUD rate를 설정해준다. 그리고 원하는 입력을 적어 송신한다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/serial_terminal.png" alt="시리얼 터미널" style="width: 70%">
<img src="{{ site.baseurl }}/assets/img/post/FPGA/char_led.JPG" alt="!의 LED" style="width: 70%">

위의 이미지는 !에 해당하는 LED이다.(0,5 번 인덱스에 불빛이 들어왔다.)

## 2. 최종 코드 파일

ex3 serial_communication 예제의 [alchitry_top.luc](hhttps://github.com/mia2583/alchitry/blob/main/ex3_serial_communication/alchitry_top.luc)


## 참고

Alchitry의 [Components](https://alchitry.com/tutorials/lucid_v1/components/) 
