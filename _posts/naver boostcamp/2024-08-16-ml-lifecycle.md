---
layout: single
title: "[Week 2]머신러닝 기초 이론"
date: 2024-08-16 19:00:00 +0900
categories: naver-boostcamp
---

어떠한 작업 T에 대하여 경험 E 와 함께 성능 P 를 향상시키는 것을 머신러닝이라고 한다. `<P, T, E>`

- 작업(T): 주어진 이미지가 고양이인지 개인지 분류하는 작업
- 경험(E): 고양이와 개 이미지들로 이루어진 학습 데이터셋
- 성능(P): 이미지 분류 정확도

> "Learning is any process by which a system imporves performance from experience" - Herbert Simon

## 최소 제곱법(OLS; Ordinary Least Squares)

선형 회귀 문제에서 파라미터(w, b)를 구하는 방법이다.

$$Cost(w, b) = \sum{(y_i- (wx_i + b))^2}$$

이를 최소화하기 위해 `gradient descent` 과정을 거쳐도 되지만 수식적으로 계산도 가능하다.

먼저 수식을 간략화 하기 위해 w와 b를 하나로 합쳐 $\beta$라고 표현한다.

$$
\begin{aligned}
\beta &= \left[
    \begin{array}{c} w \\ b \end{array}
    \right] \\
\mathbf{x} &= \left[
    \begin{array}{c} x & 1 \end{array}
    \right] \\
wx + b &= \mathbf{x}\beta
\end{aligned}
$$

위 식을 활용하면 아래와 같이 표현이 가능하다.

$$
\begin{aligned}
Cost(w, b) &= \sum{(y_i- (wx_i + b))^2} \\
           &= \sum{(y_i- \mathbf{x}_i\beta)^2} \\
\end{aligned}
$$

이제 미분 값이 0되는 지점의 $\beta$를 찾으면 된다.

$$
\begin{aligned}
\frac{\partial Cost(w, b)}{\partial \beta} &= \sum{-2\mathbf{x}_i \cdot (y_i- \mathbf{x}_i\beta)} \\
                                           &= -2 X^T(Y-X\beta) \\
\end{aligned}
$$

$$
\begin{aligned}
-2X^T(Y-X\beta) &= 0 \\
X^TX\beta &= X^TY
\end{aligned}
$$

$$\beta = (X^TX)^{-1}X^TY$$

이렇게 함으로써 원하는 파라미터를 계산 해낼 수 있다.

## 모델 평가 지표

### MAE(평균 절대 오차)

각각의 차이를 더한다.

$$MAE = \frac{1}{n}\sum_{i=1}^n{|y_i - \hat{y}_i|}$$

### MSE(평균 제곱 오차)

차이의 제곱을 더하기 때문에 데이터에 골고루 적용된다.

$$MSE = \frac{1}{n}\sum_{i=1}^n{\left(y_i - \hat{y}_i\right)^2}$$

### RMSE(제곱근 평균 제곱 오차)

제곱근을 통해 오차를 원래의 단위로 변환하였다. 해석이 용이하기 때문에 모델 평가에서 많이 사용된다.

$$RMSE = \sqrt{\frac{1}{n}\sum_{i=1}^n{\left(y_i - \hat{y}_i\right)^2}}$$

### $R^2$(결정 계수)

모델이 종속변수 y의 변동성을 얼마나 잘 설명하는지를 나타내는 지표

0 ~ 1 범위를 가지며 1이면 완벽히 설명하는 것이다.

$$
\begin{aligned}
R^2 &= \frac{ESS}{TSS} \\
    &= \frac{\sum_{i=1}^n{\left(\hat{y}_i - \bar{y}\right)^2}}{\sum_{i=1}^n{\left(y_i - \bar{y}_i\right)^2}} \\
    &= 1 - \frac{RSS}{TSS} \\
    &= 1 - \frac{\sum_{i=1}^n{\left(y_i - \hat{y}_i\right)^2}}{\sum_{i=1}^n{\left(y_i - \bar{y}\right)^2}}
\end{aligned}
$$

- **TSS: Total Sum of Squares**
  - 원래 데이터의 변동량
- **ESS: Explain Sum of Squares**
  - 모델이 표현하는 데이터(예측값)의 변동량
- **RSS: Residual Sum of Squares**
  - 예측값과 데이터와의 차이

## KL Divergence(Kullback - Leibler)

실제 분포 P와 예측 분포 Q 간의 차이를 측정하여 모델을 평가하는 지표이다.

$$
\begin{aligned}
D_{KL}(P \parallel Q) &= \sum_{i} P(i) \log \frac{P(i)}{Q(i)} \\
D_{KL}(P \parallel Q) &= \int P(x) \log \frac{P(x)}{Q(x)} dx
\end{aligned}
$$

`KL Divergence`는 정보 이론의 엔트로피를 바탕으로 설계되었다.

- **Entropy**
  - $H(P) = - \sum_{i} P(i) \log P(i)$
  - 확률 분포 P에서 발생하는 정보량
- **Cross Entropy**
  - $H(P, Q) = - \sum_{i} P(i) \log Q(i)$
  - 실제 분포 P와 예측 분포 Q에 대해 발생하는 정보량

$$
\begin{aligned}
D_{KL}(P \parallel Q) &= \sum_{i} P(i) \log \frac{P(i)}{Q(i)} \\
                      &= H(P, Q) - H(P)
\end{aligned}
$$

즉, `KL Divergence`은 크로스 엔트로피의 값에서 엔트로피를 뺀 값이다. 크로스 엔트로피가 최소가 되는 지점은 P와 Q가 같은 것으로 이때 H(P)와 동일한 값을 가지게 된다. 따라서 `KL Divergence`의 최솟값은 0이 되어 평가 지표로 사용된다.

> $D_{KL}(P \parallel Q)$는 $D_{KL}(Q \parallel P)$ 와 같지 않다!
>
> Divergence는 발산이 아니라 "차이"를 의미하게 사용되었다.

## numerical instability of Softmax

`np.exp(2000) = np.inf`이며 `np.inf / np.inf = nan`의 결과가 나온다.

따라사 softmax 함수의 오버 플로우 가능성을 막기 위해 최댓값을 빼주어 최댓값을 0으로 만들며 다른 값들을 전부 음수로 만든다.

$$\text{softmax}(x)_i = \frac{e^{x_i - \max(x)}}{\sum_j e^{x_j - \max(x)}}$$

> 이 경우 언더 플로우로 작은 값들이 0으로 간주될 수도 있지만, 이는 큰 문제가 되지 않는다.

## 데이터 정규화 기법

- PCA: 데이터를 정규화하여 zero-center을 만들고 축을 정렬
- whitening: `covariance` 행렬을 만듬
