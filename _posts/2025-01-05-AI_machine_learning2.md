---
title: "[AI] Machine Learning2"
date: 2025-01-05 21:30:00 +09:00
categories: 인공지능
description: 기계 학습 성능에 영향을 주는 요소와 초거대 언어모델에 대해 소개한다.
pin: true
use_math: true
---

## 4. 학습 시 고려사항
기계 학습의 목표인 일반화는 오차를 구성하는 편향(bias), 분산(variance)과 밀접하게 연관되어 있다. 최적화된 모델을 선택하기 위해서는 이 둘의 trade-off 관계를 이해할 필요가 있다. 그리고 학습 정도에 따른 underfitting과 overfitting에 대해서도 설명한다.

기계 학습을 진행하기 위해서는 사용할 모델 클래스(hyper class, ex. 선형 모델)와 파라미터(parameter)를 먼저 정의해야 한다. 모델은 학습 데이터 s에 대해 잘 작동하는 파라미터 W, b를 찾게 되는데, 이를 parameter estimation이라 한다.

해당 모델이 잘 동작되는지 평가하기 위해서 손실 함수(Loss function)를 사용하는데, 손실 함수는 실제와 예측값이 차이가 날 수록 큰 값을 반환하도록 정의된다.

결국 모델의 학습은 손실 함수를 최소화하는 문제로 변형될 수 있다.

> 지도 학습에서의 손실함수  
> - 분류(Classification) : 0/1 loss function  
> - 회귀(regression) : squared loss function

### 4-1. Generalization Error
모든 데이터를 가지고 있는 이상적인 데이터 셋을 universal dataset이라고 하자. 이 universal 데이터셋으로 학습을 하면 우리는 true distribution을 생성할 수 있다. 하지만 이 universal 데이터셋은 구할 수 없기에 우리는 일부를 샘플링한 train set과 test set으로 대신 학습과 평가를 한다.

일반화 능력에 대한 에러를 generalization error, train 데이터셋에 대한 에러를 training error 그리고 테스트 데이터셋에 대한 에러를 test error라고 하자. 그러면 underfitting과 overfitting에 대한 정의를 아래와 같이 할 수 있다.

- Underfitting : Generalization error < Training error ( =training error가 충분히 낮지 않은 상황)
- Overfitting : Generalization error > Training error ( =training error와 test error간의 차이가 매우 큰 상황)

Underfitting의 경우, 모델이 학습이 아예 안된 상황이기에 Overfitting보다 더 좋지 않은 상황이다.

### 4-2. Model Capacity
현재 7개의 데이터가 있고, 해당 데이터에 대해 모델을 생성한다고 하자.

<img src="{{ site.baseurl }}/assets/img/post/AI/model_capacity.JPG" alt="모델 차수에 따른 표현" style="width: 100%">

왼쪽 이미지는 linear 함수로 해당 데이터를 학습하는 상황이다. 아무리 여러번 학습을 수행해도 하나의 선으로는 데이터에 대한 일반화가 어렵다. 따라서 이 경우는 Underfitting이라고 할 수 있다.

반대로 데이터를 9차 함수로 표현한다고 하자. 이 경우에는 오른쪽 그림과 같이 모든 데이터를 정확히 지나가는 모델을 그릴 수 있을 것이다. 하지만 이 경우에 여러번의 학습이 수행되면 데이터에 경향에 과하게 맞추기 위해, 데이터가 관측되지 않은 곳에 대해서는 급격한 기울기 변화가 일어나는 overfitting이 일어날 수 있다.

그렇기에 적절한 일반화가 이루어지기 위해서는 중간 이미지와 같이 모델의 차수를 너무 작지도, 너무 크지도 않은 수로 설정해야 한다.

<img src="{{ site.baseurl }}/assets/img/post/AI/error_capacity.JPG" alt="에러와 모델 차수와의 관계" style="width: 80%">

일반적으로 모델의 차수가 클수록, training error는 줄어들지만 generalization  error과의 차이는 커진다. 따라서 우리는 training error가 충분히 낮으면서 generallization error와 차이가 작은 빨간선 지점의 모델 차수를 찾아야 한다.

> Occam의 면도날 법칙  
> 현상을 설명할 수 있는 모델이 여러개일 때, 경험적으로 가장 간단한 모델이 정답인 경향이 있다.

### 4-3. Regularization(규제)

앞서 모델의 학습은 손실 함수를 최소화하는 문제로 변형될 수 있다고 말했었다. 하지만 overfitting에 빠지지 않기 위해서는 모델의 capacity 또한 동시에 최소화해야 한다. 이럴 때 사용할 수 있는 것이 regularization이다.


// 추가 작성



### 4-3. 편향과 분산

<img src="{{ site.baseurl }}/assets/img/post/AI/bias_variance.JPG" alt="편향과 분산" style="width: 80%">

편향과 분산은 양궁 과녁으로 쉽게 이해할 수 있다. 모델의 목표는 중앙 과녁을 맞추는 것이 목표이다. 하지만 편향이 높으면 원래 목표 지점으로부터 예측이 많이 멀어지게 되고, 분산이 높으면 어떤 때는 잘 예측하고 어떤 때에는 잘못된 결과를 내듯이 예측 결과의 정답률이 불안정하다. 따라서 좋은 모델은 안정적이게 중앙과녁 중심으로 예측해야 한다. 

편향이 높다는 것은 정답을 근처로 예측하지 못하는 상황이므로 underfitting을 포함한다고 해석할 수 있다. 따라서 편향은 모델의 복잡성을 높여 낮출 수 있다.

반대로 분산이 높다는 것은 모델이 불안정하는 것을 의미하므로 overfitting으로 해석할 수 있다. 따라서 분산은 학습 데이터 수를 늘려 낮출 수 있다. 

하지만 편향과 분산은 서로 trade off 관계이며 아래 수식을 만족한다.

test error = Bias + Variance
- bias는 예측의 평균 값과 실제 값 사이의 평균
- variance는 예측 평균값과 예측 값 사이의 평균

적절히 낮은 값의 편향과 분산를 가지도록 모델을 학습해야 한다. 


모델 학습 방법
underfitting
overfitting

## 5. 초거대 언어 모델




## 참고
본 포스팅은 LG Aimers 강좌 중 서울대학교 김건희 교수님의 'Machine Learning 개론'에서 학습한 내용을 정리한다.


