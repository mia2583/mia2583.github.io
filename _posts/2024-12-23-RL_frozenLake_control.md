---
title: "[RL] 강화학습과 FrozenLake 기본 예제"
date: 2024-12-23 19:50:00 +09:00
categories: 강화학습
description: 강화학습에 대해 소개하고, FrozenLake를 간단히 조작해본다.
pin: true
---
이번 포스트에서는 강화학습 개념에 대해 간단히 소개한다. 또한, FrozenLake 환경을 이해하기 위해 강화학습을 실습하기 전에 사람이 직접 조작할 수 있는 형태로 게임을 실행해본다. 

## 1. 강화학습이란?
강화학습은 시행착오를 통해 학습하는 방법이다. 에이전트는 실수와 보상을 기반으로 가중치와 편향을 학습하며, 이를 통해 최적의 정책을 학습하는 것을 목표로 한다.

강화학습에서는 에이전트와 환경이 주어진다. 에이전트가 주어진 환경(environment)에서 어떤 행동(action)을 취하느냐에 따라 현재 상태(state 혹은 observation)가 달라지며, 이 과정에서 에이전트는 더 많은 보상을 얻기 위한 행동을 학습한다.

강화학습은 오래전부터 존재하던 개념이지만 Atari Breakout Game에 적용되면서 주목받기 시작했다. 유명한 알파고도 강화학습 알고리즘을 메인으로 사용하여 학습되었다.

## 2. OpenAI Gym
강화학습을 진행하려면 에이전트와 환경이 필요하다. OpenAI Gym은 강화학습 실험에 필요한 다양한 환경을 제공한다. FrozenLake도 OpenAI Gym에서 제공하는 환경 중 하나이다.

 Gym은 다음 명령어로 설치할 수 있다.
 ```shell
 pip install gym
```

## 3. FrozenLake 소개
<img src="{{ site.baseurl }}/assets/img/post/RL/frozenLake_info.JPG" alt="FrozenLake 게임화면" style="width: 70%">

FrozenLake는 에이전트를 시작점(S)에서 목적지(G)까지 이동시키는 게임이다. 에이전트는 상하좌우로 움직이며, 얼음판(F)을 통해 목적지로 가는 안전한 경로를 찾아야 한다. 단, 구멍(H)에 빠지면 게임이 종료된다.

목적지(G)에 도달하면 에이전트는 보상(reward)을 얻게 된다.


환경에 대한 더 자세한 정보는 [Gym Document-FrozenLake](https://www.gymlibrary.dev/environments/toy_text/frozen_lake/)에서 확인할 수 있다.

## 4. 실습
우선은 사용할 환경을 세팅해준다. 
```python
import gym
from gym.envs.registration import  register

register(
	id = 'FrozenLake-v3',
	entry_point='gym.envs.toy_text:FrozenLakeEnv',
	kwargs= {'map_name':'4x4', 'is_slippery':False}
)
# render_mode : (['human', 'ansi', 'rgb_array'])
env = gym.make('FrozenLake-v3', render_mode='ansi')

env.reset()
print(env.render())
```
그리고 사람이 키보드 입력으로 원하는 action을 선택할 수 있도록 한다. `keyboard`, `sys`, `pyauogui` 등을 이용하여 키보드 입력을 구현할 수 있지만, 해당 포스트에서는 파이썬의 기본 입력 `input()`을 사용하여 구현한다.
```python
# mecro
LEFT = 0
DOWN = 1
RIGHT = 2
UP = 3

arrow_keys = {
    'w' : UP,
    'a' : LEFT,
    's' : DOWN,
    'd' : RIGHT
}

key = input("Enter wasd\n") # w, a, s, d를 통해서 방향 조작

if key not in arrow_keys:
	print("Game aborted!")
	break
action = arrow_keys[key]
```

이제 선택한 action에 따라 변화된 state를 받아오고, 목적지에 도착하거나 구멍에 빠지게 되면 게임을 종료한다.
```python
state, reward, done, _, info = env.step(action)
print("State: ", state, ", Action: ", action, ", Reward: ", reward, ", Info: ", info)
print(env.render()) # 환경 출력

if done:
	print("Finished with reward ", reward)
	break
```
키보드 입력과 상태 변화는 게임이 종료될 때까지 반복해야 하므로 반복문으로 감싸준다.
```python
while True:
```

최종적으로 전체 코드는 아래와 같다.
```python
import gym
from gym.envs.registration import  register

# mecro
LEFT = 0
DOWN = 1
RIGHT = 2
UP = 3

arrow_keys = {
    'w' : UP,
    'a' : LEFT,
    's' : DOWN,
    'd' : RIGHT
}

register(
	id = 'FrozenLake-v3',
	entry_point='gym.envs.toy_text:FrozenLakeEnv',
	kwargs= {'map_name':'4x4', 'is_slippery':False}
)
# render_mode : (['human', 'ansi', 'rgb_array'])
env = gym.make('FrozenLake-v3', render_mode='ansi')

env.reset()
print(env.render())

while True:
	key = input("Enter wasd\n") # w, a, s, d를 통해서 방향 조작

	if key not in arrow_keys:
		print("Game aborted!")
		break
	action = arrow_keys[key]
	
	state, reward, done, _, info = env.step(action)
	print("State: ", state, ", Action: ", action, ", Reward: ", reward, ", Info: ", info)
	print(env.render()) # 환경 출력

	if done:
		print("Finished with reward ", reward)
		break
```

#### 4-1. 실행 결과

키보드 입력에 따라 에이전트가 이동하고, 목적지에 도달하게 되면 보상을 얻고 게임이 종료된다.

<img src="{{ site.baseurl }}/assets/img/post/RL/frozenLake_playscreen.JPG" alt="FrozenLake 실행화면" style="width: 70%">