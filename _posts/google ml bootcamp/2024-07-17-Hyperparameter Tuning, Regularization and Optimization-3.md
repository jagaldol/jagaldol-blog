---
layout: single
title: "[DLS]Hyperparameter Tuning, Regularization and Optimization(3)"
date: 2024-07-17 02:12:00 +0900
last_modified_at: 2024-07-17 11:39:00 +0900
categories: google-ml-bootcamp
---

이번 게시글은 주로 **Hyperparameter Tuning**을 다룹니다.

## Hyperparameter 우선순위

$$\alpha >> \beta = \#hidden\,units = batch\,size >> \# layers=learning\,rate\,decay$$

- $\alpha$ : 학습률
- $\beta$: 모멘텀 계수(0.9 권장)
  - Adam의 hyperparameter는 모멘텀 $\beta$에 비해 우선순위 낮음
  - $\beta_1$: 0.9
  - $\beta_2$: 0.999
  - $\epsilon$: $10^{-8}$ 튜닝 필요 X

## Hyperparameter Tuning

다양한 값들을 직접 시도해보고 해당 모델에 가장 적합한 hyperparameter들을 골라야한다.

### Grid 대신 Random

각 hyperparameter들에 대해 정해진 값 리스트들을 조합하는 식의 grid 방식을 쓰지 말자. 대신 매 시도에 모든 hyperparameter를 랜덤하게 값 선택한다. 이를 통해 같은 수의 실험에서 각 hyperparameter에 대해 더 많은 값을 시도해볼 수 있다.

### 범위를 점점 좁게

괜찮게 나온 값들을 기준으로 랜덤 범위를 좁혀서 더 촘촘하게 검사한다.

### log 스케일 Random

hidden unit 개수 같은 것은 uniform한 확률로 해도 괜찮지만, 학습률의 경우 0.0001 ~ 1의 범위는 $10^{-4}$ ~ $10^0$으로 -4 ~ 0에서 균등하게 선택하는 걸 원한다.

이런 경우는 log scale 랜덤을 구한다.

```python
r = -4 * np.random.rand() # r = [-4, 0] 균등 분포
a = 10 ** r
```

### Panda vs Caviar Strategy

연산 자원에 따라 전략이 나뉜다. 각 동물의 babysitting의 방식을 생각해보자. 판다는 소수의 자식을 낳아 잘 크도록 열심히 보살피는 반면, 캐비어 같은 물고기 알은 한번에 수많은 알을 낳고 방치해서 성공적으로 성장하는 것만 남는다.

#### Panda Strategy - Babysitting one model

한번에 오직 한 모델을 hyperparameter를 조금씩 바꿔보며 개선시킨다. 변경에 따른 성능 변화를 직접 확인하며 그에 따라 다음 시도할 hyperparameter 값을 선택하게 된다.

#### Caviar Strategy - Training models in parallel

연산 자원이 충분하다면 동시에 여러 모델을 다양한 방식으로 학습하고 가장 나은 걸 선택한다.

## Batch Normalization(BN; Batch Norm)

입력을 normalization 시켰듯이 hidden layer의 출력도 normalization 시키는 것이 batch normalization이다.

$$
\begin{array}{ll}
\mu = \frac{1}{m}\sum_i z^{(i)} \\
\sigma^2 = \frac{1}{m} \sum_i \left(z^{(i)} - \mu \right)^2 \\
z_{norm}^{(i)} = \frac{z^{(i)} - \mu}{\sqrt{\sigma^2 + \epsilon}} \\
\tilde{z}^{(i)} = \gamma \cdot z_{norm}^{(i)} + \beta
\end{array}
$$

- 미니배치의 평균과 분산을 이용해 정규화한다.
- $\epsilon$은 divide by zero를 막기 위해 존재한다.
- $\gamma, \beta$ 는 W, b 와 같이 학습 가능한 parameter이다.
- 각 레이어에서 출력 뉴런 수 만큼의 벡터로 각 뉴런에 개별적으로 존재하는 파라미터이다.

input에 대한 normalization과 다르게 단순히 $N(0, 1)$로 표준화 하는 것이 아닌, 각 layer마다 다르게 normalization 가능하도록 $\gamma$와 $\beta$가 존재한다.

$\gamma$는 분산을 조절하고 $\beta$는 평균 값을 조정한다.

> 활성화 함수를 지난 a 값으로 할지, 선형 결합만 수행한 z 값에 할지는 논쟁이 있지만 보통 z에 Normalization을 많이 한다.

Batch Normalization을 사용하면 파라미터 b; bias가 필요없어진다. 출력들의 평균을 계산해서 빼기 때문에 bias 상수를 더하는 것이 의미가 없어진다. 대신 bias의 역할은 $~{z}$ 계산에 사용되는 $\beta$가 맡게된다.

즉, parameter는 $W^{[l]}$, $b^{[l]}$ -> $W^{[l]}$, $\beta^{[l]}$, $\gamma^{[l]}$ 로 3가지가 된다.

> BN은 mini-batch와 함께 사용된다

### test에서의 계산

훈련 과정에서는 각 mini-batch로 평균과 표준변차를 계산하여 BN을 적용한다. 하지만 실제 테스트에서는 어떻게 계산을 할까?

`exponentially weight average`로 mini-batch들의 평균과 분산의 평균을 계산하여 저장하고 이를 테스트(predict)에 사용한다.

### 추가

batch normalization과 하이퍼 파라미터가 어떠한 관계가 있는지 명확히 이해하지 못했다.

일단 장점은 다음과 같다.

- input normalization과 같이 학습 속도를 가속화할 수 있다.
- 많은 조합의 hyperparameter가 잘 동작하도록 해준다.
- Hyperparameter search problem을 쉽게 만들어준다.
- Neural network가 hyperparameter의 선택에서 좀 더 robust하도록 한다.

또 잘 정리된 블로그 링크를 남긴다.

- [Batch Normalization에 대해서 알아보자](https://velog.io/@choiking10/Batch-Normalization%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)
- 장점 아래 3개의 출처: [Batch Normalization](https://gofo-coding.tistory.com/entry/Batch-Normalization)

## Softmax Regression

2개로 분류하는 logistic 회귀가 아닌 여러 개로 분류 가능한 모델. 출력 뉴런을 각각 $P(something\|X)$ , 각 분류일 확률로 만들며, 따라서 출력 뉴런들의 합은 1이다.

$$a_i = \frac{e^{z_i}}{\sum_{j=1}^{n}e^{z_j}}$$

수식을 보면 알 수 있듯이 다른 활성화 함수와는 다른게 해당 층의 다른 뉴런들의 값 또한 계산에 포함된다는 점을 명심하자.

> logistic의 sigmoid 함수는 사실 softmax의 축소 버전이다.
>
> 가장 큰 값을 1 나머지는 0으로 바꾸는 것은 hardmax이다. 반대로 softmax는 각 확률로 변환한다.

### Loss function

마찬가지로 logistic 회귀의 일반화 버전이기에 Cross Entropy를 사용한다. 이진 교차 엔트로피에서는 (1에 관한 식) + (0에 관한 식)인 걸 생각했을 때, 일반화된 Cross Entropy에서는 모든 분류에 대해 다 더하면 된다는 걸 알 수 있다.

$$L(\hat{y}, y) = -\sum_{j=1}^{n}y_j\log\hat{y}_j$$

`back propagation`도 동일하게 $dz^{[L]}=\hat{y} - y$의 식을 사용해 n-벡터를 만들어 이루어진다.
