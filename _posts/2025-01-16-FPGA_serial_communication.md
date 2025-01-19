---
title: "[FPGA] Alchitry Au - 시리얼 통신과 FSM 예제"
date: 2025-01-16 15:20:00 +09:00
categories: FPGA
description: 시리얼 통신으로 데이터를 송/수신하고 ROM과 FSM 사용법에 대해 배운다.
pin: true
use_math: true
---

USB 포트를 사용하여 Au 보드와 컴퓨터 사이에서 데이터를 주고 받아보고자 한다. Au 보드의 USB 포트는 FTDI USB를 통해 시리얼 통신이 가능하다. 

> 시리얼 통신?  
> 데이터를 한 번에 한 비트씩 전송하는 데이터 전송 방식을 의미하며, 종류로는 USART, UART, $I^2C$ 등이 있다. 

> UART(Universal Asynchronous Receiver Transmit)?  
> 시리얼 통신을 지원하는 하드웨어 모듈로, USART와 달리 비동기식 통신만을 지원한다. 

## 1. 컴포넌트

컴포넌트(Component)는 미리 정의된 모듈로, 이번 튜토리얼에서는 두가지 컴포넌트(데이터 수신, 데이터 송신)를 사용할 예정이다.

Alchitry lab에서 템플릿 'Base Projec-Minimun boilerplate project for ALchitry boards'으로 새로운 프로젝트를 생성한다. 그리고 컴포넌트를 추가해준다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_menu.png" alt="alchitry 메뉴" style="width: 70%">

<img src="{{ site.baseurl }}/assets/img/post/FPGA/component_library.png" alt="컴포넌트 추가" style="width: 70%">

Alchitry lab의 툴바에서 블록 세개짜리 아이콘(add component)를 클릭한 후 [Component Library]-[Interface]에서 UART Rx와 UART Tx를 추가해준다.

### 1-2. UART Txs

```
module uart_tx #(
    CLK_FREQ = 50000000 : CLK_FREQ > 0,            // clock frequency
    BAUD = 500000 : BAUD > 0 && BAUD <= CLK_FREQ/2 // desired baud rate
)(
    input clk,          // clock
    input rst,          // reset active high
    output tx,          // TX output
    input block,        // block transmissions
    output busy,        // module is busy when 1
    input data[8],      // data to send
    input new_data      // flag for new data
) {
```

해당 모듈은 송신 준비가 완료되면 `new_data`가 1로 활성화되며 `data`에 저장되어 있는 송신할 데이터를 1비트씩 전송한다. 

모듈의 입력인 `block`은 UART의 송신 동작의 `busy`를 확인하는 설정이다. `block`이 1이면, `busy`를 통해 데이터가 완전히 송신되기까지 다음 작업을 멈추고(`busy`가 1이면 송신하기 위해 들어오는 새 데이터를 무시한다.), 0이면 송신 완료 여부를 확인하지 않고 계속해서 다음 작업을 이어간다. 

이 모듈이 정상적으로 작동하기 위해서는 두 개의 파라미터가 설정되어 있어야한다. `CLK_FREQ`은 클록의 주파수로. 비트당 필요한 클록 사이클 수를 계산한다. Au의 경우 기본 주파수는 100MHz이다. 두 번째는 `BAUD`는 초당 전송할 비트 수를 의미하며, alchitry lab의 시리얼 모니터는 1M로 기본 설정되어 있다. `CLK_FREQ`와 `BAUD`는 모두 음수값이 아니어야하며,`CLK_FREQ`는 적어도 `BAUD`의 두 배 이상이어야 한다. 

```
CLK_FREQ = 50000000 : CLK_FREQ > 0,            // clock frequency
BAUD = 500000 : BAUD > 0 && BAUD <= CLK_FREQ/2 // desired baud rate
```

위의 코드와 같이 `=` 을 사용해 각 값의 default를 설정할 수 있고, `:`을 통해 파라미터 값에 대해 제한을 설정할 수 있다. 

### 1-3. UART Rx

```
module uart_rx #(
    CLK_FREQ = 50000000 : CLK_FREQ > 0,            // clock frequency
    BAUD = 500000 : BAUD > 0 && BAUD <= CLK_FREQ/4 // desired baud rate
)(
    input clk,      // clock input
    input rst,      // reset active high
    input rx,       // UART rx input
    output data[8], // received data
    output new_data // new data flag (1 = new data)
) {

```

이 모듈은 데이터를 받는 역할을 한다. 만약 데이터가 수신이 되면 `new_data`가 1로 활성화되고, 수신한 데이터는 `data`에 저장된다. Tx와 달리 Rx는 `CLK_FREQ`가 `BAUD`의 4배 이상이어야 한다. 만약 4배이면 한 비트동안 4번의 클록이 발생하기에 (하나의 비트를 4번 샘플링하기 때문에) half cycle을 기다린 후 중간 클록의 데이터를 샘플링하여 수신된 데이터의 신뢰성을 높일 수 있다.

### 1-4. 컴포넌트 사용하기

컴포넌트를 사용하기 위해서는 우선 `alchitry_top.luc`에서 이를 정의를 해주어야 한다. 

```
.clk(clk) {
  
  ...
 
    .rst(rst) {
        // 방법1
        uart_rx rx (#BAUD(1000000), #CLK_FREQ(100000000)); // serial receiver
        uart_tx tx (#BAUD(1000000), #CLK_FREQ(100000000)); // serial transmitter

        // 혹은 방법2
        #BAUD(1000000), #CLK_FREQ(100000000) {
          uart_rx rx;
          uart_tx tx;
        }
    }
}
```

`BAUD`와 `CLK_FREQ`에 값을 할당하고, `BAUD`의 경우에는 나중에 컴퓨터와 연결 시에 컴퓨터에서 이 수를 동일하게 맞추어 주어야 통신이 가능하다.

## 2. 예제1 - 수신된 데이터에 따라 LED 불빛 켜기

보드와 컴퓨터를 시리얼 통신으로 연결하고, 컴퓨터에서 전송한 데이터를 보드에서 수신하여 그에 맞는 LED를 켠 후, 데이터를 다시 컴퓨터로 전송해주는 프로젝트를 만들어보자.

### 2-1. 데이터 송/수신하기
이제 정의된 rx, tx 모듈을 FPGAq보드의 입력, 출력 신호에 연결한다.

```
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

 수신 역시 마찬가지로 수신할 데이터가 전달되면 rx.new_data가 1로 활성화되고, rx.data에 수신한 데이터를 저장한다. 

따라서 위의 코드는 컴퓨터에서 송신한 데이터를 FPGA 보드에서 데이터를 받고(rx), 이를 다시 컴퓨터쪽으로 송신(tx) 해준다. 

### 2-2. 송신된 데이터를 보드에서 사용하기

그러면 마지막으로 수신된 데이터에 따라 보드의 LED도 함께 켜지도록 해보자. clk 블록 내부면서 rst 블록 외부인 곳에 D 플립플롭 하나를 정의해준다.
그리고 이 D 플립플롭에 수신된 데이터의 저장한다.

```
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

### 2-3. 결과

Alchitry Lab에서 [Tools]-[Serial Terminal]에서 보드와 BAUD rate를 설정해준다. 그리고 원하는 입력을 적어 송신한다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/serial_terminal.png" alt="시리얼 터미널" style="width: 70%">

<img src="{{ site.baseurl }}/assets/img/post/FPGA/char_led.JPG" alt="!의 LED" style="width: 70%">

위의 이미지는 !에 해당하는 LED이다.(0,5 번 인덱스에 불빛이 들어왔다.)

## 2-4 최종 코드 파일

ex3 serial_communication 예제의 [alchitry_top.luc](https://github.com/mia2583/alchitry/blob/main/ex3_serial_communication/alchitry_top.luc)


## 3. 예제2 - 'h'가 수신될 때, "Hello world!" 전송하기

이번에는 USB 시리얼을 통해 "h"가 수신되었을 때, ROM에 저장된"Hello World"를 보내는 프로젝트 생성하고자 한다. 

### 3-1. ROM(Read-Only Memory)

전송할 문장인 "Hello World!"를 ROM에 미리 저장해놓자. 이를 위해서 새로운 모듈 `hello_world_rom`을 생성한다.

```
module hello_world_rom (
    input address[4],  // ROM address
    output letter[8]   // ROM output
) {
 
    const TEXT = "\r\n!dlroW olleH"; // text is reversed to make 'H' address [0]
 
    always {
      letter = TEXT[address]; // address indexes 8 bit blocks of TEXT
    }
}
```

이때 TEXT는 2차원 배열의 형식이다. 1차원에는 각 캐릭터에 대한 인덱스를 저장하고, 2차원은 8비트로 실제 캐릭터 값을 저장한다. 이때, "H"가 주소 [0]에 저장되고, 이어서 "e"는 [1], "l"은 [2], ...순으로 저장된다. 실제 각 캐릭터는 TEXT의 2차원에 저장되므로 `letter = TEXT[address];`로 접근할 수 있다.

### 3-2. FSM(Finite State Machine)

이제 현재 상태에 따라 다른 동작을 하는 FSM 모듈(greeter)을 새로 생성하자.

유한 상태 기계은 D-플립플롭과 비슷하게 clk, rst와 입력 d, 출력 q를 가진다. 하지만 유한 상태 기계의 경우에는 값이 아닌 상태를 저장한다. 이번 예제에서는 FSM의 상태를 `enum`으로 IDLE 또는 GREET로 정의하고 상태에 따라 다르게 동작하도록 하자. `$width()`는 입력값의 크기를 반환한다. 

```
module greeter (
    input clk,         // clock
    input rst,         // reset
    input new_rx,      // new RX flag
    input rx_data[8],  // RX data
    output new_tx,     // new TX flag
    output tx_data[8], // TX data
    input tx_busy      // TX is busy flag
) {

    const NUM_LETTERS = 14;

    enum States {IDLE, GREET};      // define state

    .clk(clk) {
        .rst(rst) {
            dff state[$width(States)];     // finite state machine
        }
        dff count[$clog2(NUM_LETTERS)]; // min bits to store NUM_LETTERS - 1
    }

    hello_world_rom rom;

    always {
        rom.address = count.q;
        tx_data = rom.letter;       // which data to send (get from ROM)
 
        new_tx = 0; // default to 0
 
        case (state.q) {
            States.IDLE:            // if IDLE and receive "h", change state to GREET
            count.d = 0;
            if (new_rx && rx_data == "h")
                state.d = States.GREET;
 
            States.GREET:           // if GREET, send "Hello World". Once done, change state to IDLE
                if (!tx_busy) {
                    count.d = count.q + 1;
                    new_tx = 1;
                    if (count.q == NUM_LETTERS - 1)
                        state.d = States.IDLE;
                }
        }
    }
}
```

ROM에 저장된 데이터를 주소 0에서부터 NUM_LETTERS까지 순서대로 가져오기 위해서 D-플립플랍 카운터를 정의해서 사용하고 있다. `$clog2`는 입력된 값의 $log_2$를 취해 반환해주기 때문에 최소 비트 크기(특정 값 범위를 표현하기 위한 비트 수) 계산에 주로 사용된다.

### 3-3. greeter 모듈 연결하기

이제 마지막으로 앞서 생성한 greeter(FSM) 모듈을 `alchitry_top.luc`에 적용시켜준다.

```
.clk(clk) {
    .rst(rst) {
        ...
        greeter greeter; // instance of our greeter
    }
}

always {
    ...
    rx.rx = usb_rx;         // connect rx input
    usb_tx = tx.tx;         // connect tx output
 
    greeter.new_rx = rx.new_data;
    greeter.rx_data = rx.data;
    tx.new_data = greeter.new_tx;
    tx.data = greeter.tx_data;
    greeter.tx_busy = tx.busy;
    tx.block = 0;
 
    led = 8h00;             // turn LEDs off
}
```

### 3-4. 결과

Alchitry Lab에서 [Tools]-[Serial Terminal]에서 보드와 BAUD rate를 설정해준다. 그리고 입력 "h"를 적어 송신한다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/serial_terminal2.png" alt="시리얼 터미널2" style="width: 70%">

## 3-5. 최종 코드 파일

ex4 greeting 예제의 [코드 링크](https://github.com/mia2583/alchitry/tree/main/ex4_greeting)


## 참고

Alchitry의 [Components](https://alchitry.com/tutorials/lucid_v1/components/)와 [ROMs and FSMs](https://alchitry.com/tutorials/lucid_v1/roms-and-fsms/)

