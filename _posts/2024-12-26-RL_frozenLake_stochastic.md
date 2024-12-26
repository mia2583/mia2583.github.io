---
title: "[RL] Stochastic 환경에서의 FrozenLake"
date: 2024-12-26 14:40:00 +09:00
categories: 강화학습
description: Stochastic 환경에서의 FrozenLake를 학습해본다.
pin: true
use_math: true
---
이번 포스팅에서는 stochastic 환경에 대해 간단히 소개하고, stochastic 환경에서 FrozenLake를 학습해본다.

## 1. Deterministic VS Stochastic world
환경은 크게 Deterministic 과 Non Deterministic(혹은 Stochastic) 으로 분류할 수 있다. Deterministic model이란 파라미터와 조건에 따라 모델의 결과가 정해진 모델을 말하고, Stochastic model이란 모델의 결과가 다양하게 나타나는 모델을 말한다.

이를 FrozenLake 환경에서 이해해보자. 이때까지는 우리가 `action=RIGHT`를 선택하면, 에이전트는 정상적으로 `RIGHT`로 한 칸 이동했다. 하지만 실제로 우리가 얼음 위에서 이동할 때에는 미끄러져서 한 칸이 아니라 두 칸 이상 이동할 수도 있고, 오른쪽이 아닌 아래로 이동하게 될 수도 있다. 이처럼 의도한대로 이동되지 않고 확률적으로 이동하는 것을 stochastic하다고 한다.

## 2. Stochatic FrozenLake
FrozenLake 환경에서 stochastic한 환경을 설정하려면 어떻게 해야할까?

```python
register(
	id = 'FrozenLake-v3',
	entry_point='gym.envs.toy_text:FrozenLakeEnv',
	kwargs= {'map_name':'4x4', 'is_slippery':is_slippery}
)
```

FrozenLake 실행 코드에서 환경 세팅 부분을 자세히 보면 `is_slippery`라는 옵션이 있다. default가 True이지만 그동안은 False로 세팅하고 사용하였다. 이 옵션이 True이면 stochatic한 환경이, False이면 deterministic한 환경이 설정된다.

이제 `is_slippery`를 True로 설정하고 deterministic 환경에서 사용했던 학습 코드를 그대로 사용해보자. 코드는 [여기](https://github.com/mia2583/Auto_Drive_Game_Agent/blob/main/code/example1/frozenLake_EnE.py)에서 확인할 수 있다.
(#5를 `is_slippery=True`로 수정 후 사용)

실행 결과, Success rate: 0.023으로 매우 좋지 않은 성능을 보인다.

## 3. Stochastic 환경에서의 학습
이전 코드의 성능이 좋지 않은 이유는 stochastic한 환경 때문에 액션에 따른 결과가 매번 다르다보니 이를 그대로 학습하는 것은 도움이 되지 않는다. 그렇다면 학습의 성능을 높이기 위해서는 어떻게 해야할까?

기존에 Q 테이블을 업데이트하기 위해 사용했던 수식은 아래와 같다.

$$
\hat{Q}(s, a) \leftarrow r + \gamma \;max_{a'} \hat{Q}(s', a') \quad (\gamma<1)
$$

action으로 얻을 수 있는 보상과 다음 state에서 얻을 수 있는 최대 보상의 합으로 Q 테이블의 값을 업데이트 하였다. 하지만 stochastic 환경에서는 action에 대한 결과값이 확률적으로 얻어지기에 모든 데이터를 즉각해서 반영하기 보다는 반복 시행으로 얻은 값들의 평균으로 업데이트해보자.

예를 들어 n개의 데이터가 있다고 하면 평균은 아래와 같다.

$$
average_n = \frac{1}{n} \sum_{i=1}^{n} A_i = \frac{1}{n} (\sum_{i=1}^{n-1} A_i + A_n)
$$

위의 평균을 정리하면 아래와 같은 변형식을 얻을 수 있다.

$$
\begin{align*}
average_n &= \frac{1}{n} (\sum_{i=1}^{n-1} A_i + A_n) \\
&= \frac{1}{n} ( (n-1) \times \frac{1}{n-1} \sum_{i=1}^{n-1} A_i + A_n ) \\
&= \frac{1}{n} \left( (n-1) \times average_{n-1} + A_n \right) \\
&= \frac{n-1}{n} \times average_{n-1} + \frac{A_n}{n} \\
&= average_{n-1} + \frac{1}{n} (A_n-average_{n-1})\\
\end{align*}
$$

$\frac{1}{n}$ 를 $\alpha\;(\alpha<0)$로 표기하고 이를 학습률(learning rate)라고 한다. 그리고 위의 식에서 `average = Q function`이므로 수식을 아래와 같이  다시 정리할 수 있다.

$$
\begin{align*}
\hat{Q}(s, a) &\leftarrow \hat{Q}(s, a) + \alpha \left( A_n - \hat{Q}(s, a) \right ) \\
&\leftarrow \hat{Q}(s, a) + \alpha \left( r + \gamma \;max_{a'} \hat{Q}(s', a') - \hat{Q}(s, a) \right ) \\
&\leftarrow (1-\alpha) \hat{Q}(s, a) + \alpha \left( r + \gamma \;max_{a'} \hat{Q}(s', a') \right ) \quad (\alpha, \gamma <0)
\end{align*}
$$


Deterministic 환경에서의 수식 $\hat{Q}(s, a) \leftarrow r + \gamma \;max_{a'} \hat{Q}(s', a')$ 를 생각하면, stochastic 환경에서는 자신의 기존 ${Q}(s', a')$를 최대한 살리고 새로운 지식은 매우 작은 수( $\alpha$, learning rate ) 만큼 학습에 반영한다는 것을 알 수 있다. $\alpha$ 값을 조절하여 학습이 잘되는 방향으로 이끌 수 있다.

## 4. 실습
이전의 코드에서 Q 테이블 업데이트 코드를 아래와 같이 수정해주자.

```python
Q[state, action] = (1-learning_rate) * Q[state, action] \
            + learning_rate * (reward + dis * np.max(Q[new_state, :]))
```

최종적인 전체 코드는 아래와 같다.

```python
import gym
from gym.envs.registration import  register
import numpy as np
import matplotlib.pyplot as plt

register(
	id = 'FrozenLake-v3',
	entry_point='gym.envs.toy_text:FrozenLakeEnv',
	kwargs= {'map_name':'4x4', 'is_slippery':True}
)
    
env = gym.make('FrozenLake-v3', render_mode='ansi')

# Q 테이블 생성
Q = np.zeros([env.observation_space.n, env.action_space.n])
learning_rate = 0.85
dis = 0.99 # discount factor
num_episodes = 2000
# episode마다의 총 리워드
rList = []

for i in range(num_episodes):
    state = env.reset()[0]
    rAll = 0 # 리워드 총합
    done = False
    e = 0.1 / (i+1) # e-greedy factor

    while not done:
        # e-greedy
        if np.random.rand(1) < e:
            action = env.action_space.sample()
        else:
            # Q(a,s)에 random noise 추가(decaying)
            action = np.argmax(Q[state, :] + np.random.randn(1, env.action_space.n) / (i+1))

        new_state, reward, done, truncated, info = env.step(action)

        # discount future reward(decay rate)
        # non deterministic을 극복하기 위해 learning_rate 사용
        Q[state, action] = (1-learning_rate) * Q[state, action] \
            + learning_rate * (reward + dis * np.max(Q[new_state, :]))
        
        rAll += reward
        state = new_state

    rList.append(rAll)
    
#최종 Q 테이블 출력
print("Final Q-Table Values")
print("LEFT DOWN RIGHT UP")
print(Q)
    
#시각화
print("Success rate: " + str(sum(rList)/num_episodes))
plt.bar(range(len(rList)), rList, color="blue")
plt.show()

```
### 4-1. 실행 결과
Success rate가 0.7145로 높은 성공률을 보인다. 결과는 매 실행마다 조금씩 차이가 있을 수 있다.

## 참고
Sung Kim 강사님의 [모두를 위한 RL](https://www.youtube.com/watch?v=dZ4vw6v3LcA&list=PLlMkM4tgfjnKsCWav-Z2F-MMFRx-2gMGG)

greentec님의 [강화학습 알아보기(2) - DQN](https://greentec.github.io/reinforcement-learning-second/) (learning rate 유도식)
