---
title: "[FPGA] Alchitry Au - RAM 예제"
date: 2025-01-19 13:20:00 +09:00
categories: FPGA
description: 시리얼 통신으로 데이터를 송/수신하고 RAM 사용법에 대해 배운다.
pin: true
use_math: true
---

이전 예제에서는 "h"가 수신될 때, 보드는 "Hello World"를 출력하였다. 이번엔 이 예제를 조금 수정해, 이름을 포함시켜 인사하도록 하자.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/clone.png" alt="프로젝트 클론" style="width: 70%">

이전 프로젝트를 열고 [Menu]- [Save Project As...]하면 동일한 새로운 프로젝트를 생성할 수 있다. 

## 1. RAM(Random Access Memory)

RAM은 데이터를 임시로 저장하고 빠르고 읽고 쓸 수 있는 메모리 장치이다.

<img src="{{ site.baseurl }}/assets/img/post/FPGA/simple_ram.png" alt="심플 램" style="width: 70%">

먼저 이름을 보관할 RAM 컴포넌트를 생성해주자. 해당 컴포넌트는 lucid 언어가 아닌 verilog로 작성되어 있다. 
FPGA에는 실제 RAM(BRAM)이 존재하기에 만약 작성된 RAM이 충분히 크다면, FPGA는 실제 RAM을 사용하여 이를 구현한다.
RAM은 이전 예제의 ROM과 비슷하게 작동하나 읽기 뿐만 아니라 주소에 값을 쓸 수도 있다. `write_enable`이 1이면, `write_data`에 저장된 데이터를 해당 주소에 적는다.

`WIDTH`는 각 항목의 크기를 지정하고, `ENTRIES`는 항목의 개수를 설정한다. 두 값의 default는 1로 설정되어 있다.

## 2. Greeter 모듈 수정하기

### 2-1. ROM 제거하기

greeter 모듈에서 ROM 모듈을 제거하고 대신에 constant 변수에 값을 저장한다. 

```
const HELLO_TEXT = $reverse("\r\nHello @!\r\n") // reverse so index 0 is the left most letter
const PROMPT_TEXT = $reverse("Please type your name: ")
```

이전에 예제에선 TEXT를 거꾸로 입력했던 것을 기억할 것이다. 이번에는 `$reverse()` 함수를 사용하여 더 쉽게 이를 처리하도록 한다. 
문자열의 각 캐릭터는 HELLO_TEXT[i]와 같이 인덱스로 접근이 가능하다. `@`는 이름을 삽입할 위치를 알려주는 기호이다. 


```
enum States {IDLE, PROMPT, LISTEN, HELLO}

.clk(clk) {
    .rst(rst) {
        dff state[$width(States)]   // our state machine
    }

    dff hello_count[$clog2($width(HELLO_TEXT, 0))]
    dff prompt_count[$clog2($width(PROMPT_TEXT, 0))]
    dff name_count[5]    // 5 allows for 2^5 = 32 letters

    simple_ram ram (#WIDTH(8), #ENTRIES($pow(2,$width(name_count.q))))
}
```

이전 프로젝트에서 FSM의 상태를 저장했던 것처럼 시작 상태를 IDLE, 이름을 물어보는 상태를 PROMPT, 이름을 듣는 상태를 LISTEN 그리고 마지막으로 인사와 이름을 출력하는 HELLO 상태로 분류하자. 
PROMPT와 HELLO의 경우에는 우리가 몇 개의 캐릭터를 출력했는지 추적하기 위해 카운터가 각각 필요하다. 그리고 입력할 이름의 저장하기 위해서 LISTEN에 대한 카운터(`name_count`)도 필요하다. 

TEXT의 `$width(TEXT, 0)`은 전체 문자의 수, `$width(TEXT, 1)`은 각 문자 크기인 8비트를 반환한다. 그리고 name_counter의 크기를 5로 설정했으므로 ram의 `ENTRIES`도 2^5인 32로 설정한다.

### 2-3. 기본값 설정

```
always {
    ram.address = name_count.q // use name_count as the address
    ram.write_data = 8hxx      // don't care
    ram.write_enable = 0       // read by default

    new_tx = 0                 // default to no new data
    tx_data = 8hxx             // don't care\
    ...
}
```

`name_count`의 q가 증가할 때마다 이름의 다음 캐릭터를 출력하므로 RAM의 주소가 곧 name_count.q와 동일하다.
아직 RAM에 기록되면 안되므로 `write_enable`을 0으로, 출력 `new_tx` 또한 0으로 비활성화한다.


### 2-2. FSM 상태에 따라 행동 정의하기

`LISTEN` 상태일 때는 사용자가 입력하는 이름을 듣고 RAM에 기록해야 한다. 

```
States.LISTEN:
    if (new_rx) { // wait for a new byte
        ram.write_data = rx_data       // write the received letter to RAM
        ram.write_enable = 1               // signal we want to write
        name_count.d = name_count.q + 1  // increment the address
 
        new_tx = rx_data != "\n" && rx_data != "\r" // only echo non-new line characters
        tx_data = rx_data // echo text back so you can see what you type
 
        // if we run out of space or they pressed enter, change state
        if (&name_count.q || rx_data == "\n" || rx_data == "\r") {
            state.d = States.HELLO
            name_count.d = 0  // reset name_count
        }
    }
```

new_rx가 활성화되어 새로운 데이터가 들어오면, 해당 데이터를 RAM에 저장한다. 또한 입력한 이름을 송신측에서도 볼 수 있도록 다시 전송해준다. 
즉, new_tx를 1로 활성화하고, tx_data에 rx_data를 전달해준다.

그리고 모든 이름이 입력되어 사용자가 enter(\n)가 입력하거나, 이름을 저장할 공간이 부족한 경우에 상태를 HELLO로 이동한다.

`HELLO` 상태일 때는 HELLO_TEXT를 출력하고 RAM에 저장되어 있는 이름을 함께 출력하면 된다. 

```
States.HELLO:
    if (!tx_busy) { // wait for tx to not be busy
        if (HELLO_TEXT[hello_count.q] != "@") { // if we are not at the sentry
            hello_count.d = hello_count.q + 1    
            new_tx = 1                           // new data to send
            tx_data = HELLO_TEXT[hello_count.q]  
        } else {                                // we are at the sentry
            name_count.d = name_count.q + 1      // increment the name_count letter
 
            if (ram.read_data != "\n" && ram.read_data != "\r") // if we are not at the end
                new_tx = 1
 
            tx_data = ram.read_data             // send the letter from the RAM
 
            // if we are at the end of the name or out of letters to send
            if (ram.read_data == "\n" || ram.read_data == "\r" || &name_count.q) {
                hello_count.d = hello_count.q + 1  // increment hello_count to pass the sentry
            }
        }
 
        // if we have sent all of HELLO_TEXT
        if (hello_count.q == $width(HELLO_TEXT, 0) - 1)
            state.d = States.IDLE // return to IDLE
    }
````

`tx_busy`가 0인 즉, 어떠한 데이터도 전송하고 있지 않을 때 전송을 시작한다. 
HELLO_TEXT를 전송하기 위해서 hello_count 카운터를 사용하고, RAM에 저장된 이름을 전송하기 위해선 name_count 카운터를 사용한다.
HELLO_TEXT를 전송 중에 '@'를 만나게 되면 else문에서 RAM의 내용을 출력한다. RAM의 내용을 모두 출력한 후에야 hello_count의 값을 증가시켜 모든 출력을 종료한다.
그리고 IDLE 상태로 전환한다.

`IDLE`과 `PROMPT`는 각각 초기 상태, 이름 질문을 출력을 하는 상태이다 따라서 이전 프로젝트의 IDLE과 GREET와 거의 동일하나, 다음 상태를 각각 `PROMT`, `LISTEN`으로 설정하면 된다.


## 3. 프로젝트 실행하기

`alchitry_top.luc`는 이전 프로젝트와 동일하게 사용하면 된다. 

### 3-1. 결과

입력한 이름을 인사 후, 그대로 출력해준다. 이름을 32자 이상 입력하면 이를 끊고 인사를 한다. 

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_ex5.png" alt="ex5 결과" style="width: 70%">

# 3-2. 최종 코드 파일

ex5 greeting with name 예제의 [코드 링크](https://github.com/mia2583/alchitry/tree/main/ex5_greeting_with_name)

## 참고

Alchitry의 [Hello YOUR_NAME_HERE](https://alchitry.com/tutorials/lucid_v1/hello-your-name-here/)
