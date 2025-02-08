---
title: "[SLAM] 로봇 위치 추"
date: 2025-02-08 13:20:00 +09:00
categories: SLAM
description: 
pin: true
use_math: true
---

어딘지 모르는 곳에 갑자기 떨어졌다고 하자. 그러면 우리는 현재 위치를 이해하기 위해, 주변 정보를 얻기 위해서 자유롭게 돌아다닐 것이다. 물론 원래 위치로 다시 돌아오는 방법을 기억하면서 말이다. 이처럼 로봇도 위치를 파악하고 위치 추적 기능을 인간과 비슷한 방식으로 수행한다. 

## 1. localization(위치 추정)

그렇다면 localized(위치를 파악함)했다는 것은 무엇을 정확히 무엇을 의미할까?

> "Robot localization denotes the robot's ability to establish its own position and orientation within the frame of reference"

위키피디아에 따르면 robot localization은 로봇이 공간내에서 참조 프레임 내의 자신의 위치와 방향을 찾는 능력을 말한다.

1. reference frame(참조 프레임)
2. 위치, 방향 = robot pose

이 두가지에 대해 더 자세히 이해해보자.

우리는 지구에서 경도와 위도로 지리적 좌표(위치)를 고유하게 나타내는 글로벌 reference frame을 사용하며, 적도가 그리니치와 만나는 지점을 원점(origin)으로 정의한다. 
예를 들어, 서울은 위도 37.5665° N, 경도 126.9780° E로 나타낼 수 있다. 
다른 나라의 사람이 이 위치에 대한 정보를 듣더라도, 우리와 동일한 reference frame을 사용하기 때문에 서울의 위치를 동일하게 알아낼 수 있다.

그리고 우리는 보통 어떠한 건물을 위치를 표현할 때, 자신을 기준으로 설명한다. 1m 직진하면 식당이 나온다와 같이 자신의 위치를 원점으로 잡게 된다.

이제 로봇으로 상황을 적용해보자.
경도와 위도를 사용했던 거처럼 모든 물체가 동일한 reference frame을 사용해야지 로봇을 우리가 원하는대로 동작하고 활용할 수 있다.

이를 위해서 로봇에서는 먼저 map이라고 불리는 고정된 global reference frame을 정의해야한다. 실제 이 frame의 위치는 중요하지 않지만 한번 고정되면 변경하면 안된다.

다음은 odom이라고 부르며, 로봇이 처음 움직임을 나타내는 위치를 정의해야 한다. 이 프레임 또한 자유롭게 정의할 수 있지만 한번 정의되면 고정된다. 해당 프레임은 map 프레임과 로봇의 프레임 간의 변환 관계를 추정하기 위해 사용된다.

마지막으로 로봇에 부착되어 로봇과 함께 이동하는 base link 프레임이 있다. 로봇의 이동에 따라 연속적으로 변화하며, 갑자기 다른 곳으로 이동하는 것을 불가능하다. 

map, odom, base link, 이 세 프레임을 사용하면 로봇의 위치를 추적할 수 있다. odom과 base_link를 통해 초기 위치에서 로봇이 어느 방향으로 얼만큼 이동했는지를 추적하고, map과 odom 프레임 간의 관계를 통해 로봇의 위치를 map 프레임의 기준에서 나타낼 수 있게 된다. 

즉, 로봇의 위치 추정은 map, odom 그리고 base link 간의 변환 행렬을 구하는 문제로 단순화할 수 있다. 

<img src="{{ site.baseurl }}/assets/img/post/SLAM/transform_matrix.png" alt="로봇 이동" style="width: 100%">

### 1-1. odom -> base_link

먼저 odom 프레임과 base link 프레임 사이의 변환을 구해본다. 이 변환을 odometry라고 하며, local localization이라고 한다. 
속도, 가속도, 이동 거리를 바탕으로 로봇의 위치를 추정할 수 있는데, 예를 들어, 로봇이 만약 선형적으로 이동한다면 바퀴의 회전으로 이동하는 위치를 계산할 수 있다.

$$
d = 2 \pi r \;\;(r은\;바퀴의\;반지름)
$$

실제로는 휠 인코더, IMU, 라이다, 카메라 등 다양한 도구를 사용해 로봇의 상대적 위치인 odometry를 계산한다. 하지만 시간이 지남에 따라 오차가 누적되기 때문에 이를 고려해 위치를 추정해야 한다.

### 1-2. map -> base_link

두번째는 map 프레임과 odom 프레임 사이의 변환을 구해본다. 이 변환을 global localization이라고 한다. 
이 변환은 센서를 통해 구해지며, 보통 SLAM이라 부르는 과정이 이 변환을 의미한다.

### 1-3. map -> odom
센서와 odometry의 값은 시간에 따라 오차 누적되므로 이를 보정하기 위해서 map에서 odom 프레임으로의 변환을 지속적으로 업데이트하며, 이 과정을 map correction이라고 한다.

$$
(map \rightarrow odom) = (map \rightarrow base_link) \times (base\_link \rightarrow odom)
$$

따라서 로봇의 최종 위치는 $(map \rightarrow odom) \times (odom \rightarrow base\_link)$로 추정한다.

## 2. odometry

로봇이 첫 번째 위치 A=(x, y)에서 두 번째 위치 B=(x', y')으로 이동했다고 하자. 그 사이의 로봇의 궤적이 어땠든 우리는 이 두 위치 사이의 변환을 계산할 수 있다. 

<img src="{{ site.baseurl }}/assets/img/post/SLAM/robot_move.png" alt="로봇 이동" style="width: 100%">

$$
\begin{align*}
&d = \sqrt{(x-x')^2 + (y-y')^2} \\
&r1 = atan2(y'-y, x'-x) - \theta \\
&r2 = \theta' - \theta - r1
\end{align*}
$$
 
위의 식은 로봇의 단순한 odometry 모션 모델을 나타낸다. 그리고 이 세 값은 모두 약간의 노이즈를 추가로 가지며, 각 노이즈는 평균이 0인 가우시안 분포를 가진다.

이번에는 회전과 이동을 바탕으로 변환 후의 위치를 식으로 나타내본다.

$$
x' = x + d \times cos(\theta + r1)
y' = y + d \times sin(\theat + r2)
\theta' = theta + r1 + r2
$$
