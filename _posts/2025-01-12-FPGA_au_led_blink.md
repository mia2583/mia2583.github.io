---
title: "[FPGA] led 예제2"
date: 2025-01-12 16:00:00 +09:00
categories: FPGA
description: D 플리플롭을 사용해서 blink led를 만든다.
pin: true
use_math: true
---

이전의 예제에서는 단순히 LED를 켜는 동작을 수행하도록 하였다. 이번에는 D 플립플롭을 사용하여 LED를 깜빡이게 해보자.

## 1. D-플립플롭 (D-flip-flop)

<img src="{{ site.baseurl }}/assets/img/post/FPGA/d-flip-flop.png alt="D 플립플롭" style="width: 80%">

위의 이미지는 D-플립플롭의 기호이다. 기호에서 D는 입력(data)을, en은 , clk는 클록을, rst는 그리고 마지막으로 Q는 출력을 의미한다. 

> 클록?  
> <img src="{{ site.baseurl }}/assets/img/post/FPGA/clock.png" alt="클록" style="width: 80%">  
> 클록은 단순히 0과 1 사이를 반복하여 토글하는 시그널이다. Au 보드의 클록은 100MHz 즉, 초당 1억번 토글한다. 
클록은 상승 엣지(rising edge), 하강 엣지(falling edge)로 구성된다. 일반적으로 상승 엣지가 더 중요한데, 위의 이미지에서 상승 엣지는 화살표로 표현된다.

D-플립플롭은 클록에서 상승 엣지가 발생할 때, D의 신호를 Q로 복사해 전달한다. 
클록이 진행되는 동안 한번 들어온 입력 신호를 저장(기억)해 상승 엣지에서 출력하므로, D-플립플롭은 데이터를 저장하고 상태를 유지하는 기억 소자로 불린다. 

## 2. 루프

<img src="{{ site.baseurl }}/assets/img/post/FPGA/loop.png" alt="루프" style="width: 80%">

위의 회로에 대해 생각해보자. 만약 입력이 1이라면, 출력은 0이 되고 이는 다시 입력으로 들어가 1을 출력한다. 
마치 0과 1을 빠르게 토글하는 회로로 생각되지만, 실제로는 전압을 사용하기 때문에, 진동하거나 중간값으로 고정 될 수도 있다. 
회로가 우리가 예상한 대로 동작하지 않을 수 있기 때문에 우리는 아래와 같은 회로로 대신하여 사용한다. 

<img src="{{ site.baseurl }}/assets/img/post/FPGA/loop_flip_flop.png" alt="루프와 플립플롭" style="width: 80%">

위의 회로는 입력이 0일 때, 다음 상승 엣지에서는 1을, 또 그 다음 상승 엣지에서는 0을 반복해서 출력하는 회로가 된다.
하지만 초기 상태의 D 혹은 Q의 값을 어떻게 알 수 있을까? 우리는 이 값을 알 수 대신에 rst를 통해 초기화 할 수 있다.  

루시드 언어에서 D-플립플롭을 의미하는 키워드 dff는 기본적으로 high reset으로 작동한다.

> High reset?  
> rst 신호가 1(high)일 때, 출력 Q는 0으로 reset된다. 즉, rst=1인 동안은 출력은 항상 0이다.  
> rst 신호가 0일 때는, D-플립플롭이 정상적으로 작동된다.

가끔 Q의 값을 보존하기 위해서 클록의 상승 엣지를 무시해야 할 때가 있다. 이때, en의 신호를 0으로 주면, 상승 엣지에도 출력 Q의 값은 변하지 않는다. 만약 en이 없는 플립플롭이라면, 플립플롭이 항상 정상적으로 작동한다고 생각하면 된다. 

## 3. 예제

D-플립플롭에 대해 배웠으니, 이제 이를 사용해 LED가 깜빡이도록 작동시켜보자. LED가 일정 시간 지나면 자동으로 켜지고 꺼져야 하므로 D 플립플롭이 필요하다. 

### 3-1. blinker 모듈

<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_add_module.png" alt="alchitry 모듈 추가" style="width: 80%">

Alchitry Lab에서 새로운 프로젝트를 열고, 새 module 파일을 "blinker"라는 이름으로 추가해준다.

blinker 모듈은 clk과 rst를 입력 받아 blink 신호를 출력으로 내보내며, D-플립플롭으로 구성된다. 

```
module blinker (
    input clk_,    // clock
    input rst_,    // reset
    output blink  // output to LED
  ) {
 
  dff counter[25](.clk_(clk), .rst_(rst));    // D-플립플롭
 
  always {
    blink = counter.q[24];   // counter의 q 신호를 blink에 대입
    counter.d = counter.q + 1;  // clk이 1(high)일 때마다 d의 값이 1씩 증가
  }
}

```

위의 코드에서 counter라는 이름의 D-플립플롭은 25비트의 크기의 입력과 출력을 가진다. 그리고 1씩 증가된 d의 값이 상승 엣지마다 q에 전달되므로, q는 1씩 계속해서 증가하는 카운터가 된다.

D-플립플랍인 dff를 정의할 때, clk은 항상 정의되어야 하지만, rst 정의는 선택할 수 있다. 위의 코드의 경우에는 rst 신호를 연결하였고, 이는 rst의 값이 high(1)일 때마다 카운터의 q 값은 0으로 초기화되는 것을 의미한다. FPGA가 처음 로딩될때에도 counter의 q 값은 0으로 초기화된다. 만약 0이 아닌 다른 숫자로 초기화하고 싶다면 아래와 같이 작성하면 된다. 

```
dff counter[25](#INIT(100), .clk_(clk), .rst_(rst));  // rst가 high(1)일 때, q는 100으로 초기화된다. 
```

앞서 우리가 25비트로 입출력을 정의했으므로, 카운터 회로는 실제로는 25개의 플립플롭을 사용한다. 하지만 보통 회로를 그릴 때에는 하나의 역할을 하는 여러 비트의 플립플롭은 하나로 생략해 그린다.

25비트의 카운터의 MSB(Most Significant Bit)는 25비트의 값을 정확히 반으로 나눈다. $(0~2^{24}-1, 2^{24}~2^{25}-1)$ 따라서 MSB를 blink 신호로 전달하면 절반의 시간만큼 0과 1을 각각 표현할 수 있다. 

클록의 주파수가 100MHz이고, 카운터는 0~$2^{25}-1$ 값을 반복하므로 blink에 걸리는 시간은 $frac{2^{25}}{100,000,000} = 0.34$ 초이다. 즉, 0.34초마다 LED는 켜지고 꺼짐을 반복한다. 이 사간은 카운터의 비트를 수를 수정하여 증가/감소 시킬 수 있다. 

#### D-플립플롭을 정의하는 다른 방법

D-플립플랍을 여러 개 정의할 때, {}를 통해 입력을 한번에 제어할 수도 있다.

```
// 3개의 카운터 모두 clk_, rst_를 입력으로 가진다.
.clk_(clk), .rst_(rst) {
    dff counter1[12];
    dff counter2[7];
    dff counter3[8]
}
```

또한 첫번째 방식과 위의 방식을 변형하여 clk는 모두 동일한 입력으로 가지고 일부만 rst를 입력으로 가질 수 있도록 정의할 수도 있다.

```
.clk_(clk) {
    .rst_(rst) {

    }
}

// 혹은

.clk_(clk) {
    dff counter1[12](.rst_(rst));
    ...
}
```

### 3-2. 상위 코드

이제 blinker 모듈을 사용해서 alchitry_top.luc의 최종 코드를 작성해보자.
clk와 rst를 사용해 blinker 인스턴스를 생성한다. 그리고 always 블록에서 led의 모든 비트를 blink 신호로 대체한다. 

```
s.clk(clk) {
    reset_conditioner reset_cond;
     
    .rst(rst) {
        blinker myBlinker;
    }
}
   
always {
    reset_cond.in = ~rst_n;    // input raw inverted reset signal
    rst = reset_cond.out;      // conditioned reset
     
    led = 8x{myBlinker.blink}; // blink LEDs
     
    usb_tx = usb_rx;           // echo the serial data
}
```

## 4. 결과 

rst가 low(0)일 때, 카운터가 $0~2^{25}-1$의 값을 반복하며 LED가 깜빡인다. 반면에 rst가 high(1)일 때, 카운터의 출력값이 항상 0으로 초기화되므로 LED는 꺼짐(0)을 유지한다.

<video>
    <source src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_ex2.mp4" type="video/mp4">Your browser does not support the video tag
</video>

## 5. 최종 코드 파일

ex2 led blink 예제의 [alchitry_top.luc](https://github.com/mia2583/alchitry/blob/main/ex2_led_blink/alchitry_top.luc), [blinker.luc](https://github.com/mia2583/alchitry/blob/main/ex2_led_blink/blinker.luc)


## 참고

Alchitry의 [Synchronous Logic](https://alchitry.com/tutorials/lucid_v1/synchronous-logic/) 
