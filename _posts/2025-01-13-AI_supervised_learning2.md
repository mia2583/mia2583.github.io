---
title: "[AI] 지도 학습2"
date: 2025-01-13 :23:40 +09:00
categories: 인공지능
description: 지도학습 중 분류에 대해 자세히 알아본다.
pin: true
use_math: true
---

이제 회귀에 이어, 분류 모델의 학습 과정에 대해 자세히 살펴보자.

## 1. 분류(Classification)

<img src="{{ site.baseurl }}/assets/img/post/AI/binary_classification_ex.JPG" alt="이진 분류 예시" style="width: 70%">

먼저 분류 모델 중 가장 간단한 Binary Classification의 예로 설명해보고자 한다. 이 모델은 예측 결과를 두 개의 집단으로 분류하는데, 데이터들을 선형적으로 구분 가능하다고 가정한다. 즉, 데이터를 완벽히 나누는 선 혹은 hyperplane이 하나 이상 존재한다고 가정한다. 분류 모델 역시, 회귀와 마찬가지로 함수 클래스 G 중에 정답 함수 f*에 가장 근접한 함수 g를 찾는 것을 목표로 한다.

데이터들이 $x^{(i)} = (x_1^{(i)}, x_2^{(i)})$ 형태로 주어지고 예측 결과는 $y^{(i)} \in {-1, 1}$이 된다. 

$$
L(y, \hat{y}) = /begin{cases} 
0, & \text{if} y = \hat{y} \\
1, & \text{if} y \neq \hat{y}
\end{cases}
$$

분류의 경우에는 0/1 loss 방법으로 손실함수를 정의하는데, 각 데이터에 대해 예측이 틀린 경우에는 손실 값을 1, 맞은 경우에는 손실 값을 0으로 주는, 정답의 개수를 카운팅하는 함수이다. 

Binary classification은 퍼셉트론 알고리즘으로 풀 수 있다. 

> 퍼셉트론 알고리즘(Perceptron Algorithm)?  
> $g = a^{T}x + b$라고 하면, 각 데이터에 대한 예측이 틀릴 때마다 $(y \neq \hat{y})$ 아래 수식으로 가중치와 bias를 업데이트 한다. 아래 식에서 $\eta$ 는 학습률(learning rate)이다. 
> 
$$
a \leftarrow a + \eta (y_i-\hat{y_i})x_i
b \leftarrow b + \eta (y_i-\hat{y_i})
$$

하지만 퍼셉트론 알고리즘은 정답 함수가 있다고 가정할 때 사용될 수 있다. 만약 정답 함수가 없다면, 퍼셉트론 알고리즘은 멈추지 않고 무한히 동작할 것이다. 문제는 이것이 정답 함수가 없어서 알고리즘이 무한 반복되는 것인지, 정답 함수로 향해가는 중인지를 구분할 수 없다는 것이다. 

그러면 위의 binary classification 문제를 조금 다듬어보자. 기존의 binary classification 


### 3. 데이터

일반적으로 학습 데이터를 학습, 검증 데이터로 분리해 학습을 한다.
- 학습:검증을 8:2로 주로 설정한다.
학습 데이터와 검증 데이터 그리고 테스트 데이터의 관계를 연습 문제, 모의 고사, 수능으로 생각하면 이해하기 쉽다. 학습 데이터로 학습을 하다가 검증 데이터로 overfitting이 발생하지 않았는지 확인한다. 그리고 모델의 최종적인 성능은 테스트 데이터로 판단이 된다. 

### 3-1. 교차 검증(cross validation)

validation error가 train error보다 살짝 크다면 적절히 학습이 된 것이고, 둘 사이의 차이가 크다면 overfitting일 가능성이 크다. 

만약 데이터의 양이 적어 학습 데이터와 검증 데이터를 나누기 어렵다면 교차 검증 방법을 사용할 수 있다. 
- 학습 데이터 양이 작아도 오버피팅이 될 가능성이 높다.

<img src="{{ site.baseurl }}/assets/img/post/AI/cross_validation.JPG" alt="교차 검증" style="width: 90%">

예를 들어 학습 데이터를 총 5개의 그룹으로 나누고 한번은 첫 번째 그룹을 검증 데이터로 사용하고, 다음번엔 두번째 그룹을 검증 데이터로 사용한다.
이처럼 매 iteration마다 번갈아가면서 특정 그룹을 검증 데이터로 활용하는 방법을 교차 검증이라 한다. 

### 3-2. 증강(Augmentation)

<img src="{{ site.baseurl }}/assets/img/post/AI/augmentation.JPG" alt="데이터 증강" style="width: 90%">

강아지 사진을 이동, 회전, 뒤집어도 여전히 강아지 사진이다. 이처럼 데이터에 다양한 변환(회전, 이동, 확대, 축소, 뒤집기 등)을 적용해 새로운 데이터를 생성하는 방법이다. 하지만 숫자 2와 같이 변환으로 인해 데이터의 성질이 변화되는 경우가 있을 수도 있으니 주의해야 한다.






