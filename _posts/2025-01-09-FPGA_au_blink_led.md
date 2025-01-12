---
title: "[FPGA] led 예제2"
date: 2025-01-09 12:40:00 +09:00
categories: FPGA
description: 
pin: true
use_math: true
---

이전의 예제에서는 단순히 LED를 켜는 동작을 수행하도록 하였다. 이번에는 D 플립플롭을 사용하여 LED를 깜빡이게 해보자.

## 1. D-플립플롭 (D-flip-flop)

<img src="{{ site.baseurl }}/assets/img/post/FPGA/d-flip-flop.JPG" alt="D 플립플롭" style="width: 80%">

위의 이미지는 D-플립플롭의 기호이다. 기호에서 D는 입력(data)을, en은 , clk는 클록을, rst는 그리고 마지막으로 Q는 출력을 의미한다. 


## 2. 

일정 시간이 자나면 자동으로 켜지고 꺼질 수 있는 회로를 만들고자 한다 .이를 위해서는 D 플립플롭이 필요하다. 

- 
<img src="{{ site.baseurl }}/assets/img/post/FPGA/alchitry_add_module.JPG" alt="alchitry 모듈 추가" style="width: 80%">
