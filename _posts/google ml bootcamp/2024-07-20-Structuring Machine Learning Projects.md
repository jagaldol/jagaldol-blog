---
layout: single
title: "[DLS]Structuring Machine Learning Projects"
date: 2024-07-20 03:04:00 +0900
categories: google-ml-bootcamp
---

Andrew Ng 교수님의 Coursera - Deep Learning Specialization 세번째 강의입니다.

## Orthogonalization

[앞 강의](https://blog.jagaldol.com/google-ml-bootcamp/Hyperparameter-Tuning,-Regularization-and-Optimization-1/#machine-learning-methodology)에서 말했듯이 모델 테스트 및 실험의 각 단계는 다른 단계에 영향을 미치지 않도록 하는 것이 좋다.

> 각 작업이 독립성을 띄면 orthogonality, 직교성을 가지게 되는 것이다.

## Evaluation Metrics

여러 모델 중 가장 좋은 한 모델을 선택하기 위해선 지표(Metric)이 필요하다. 지표들에는 1개의 Optimizing metric와 다수의 satisficing metric이 존재할 수 있다.

예를 들어, 최적화 지표로 정확성 / 만족 지표로 100ms 내 실행, siri 오감지(false Positive) 평균 24시간 내 1번 등이 있을 수 있다.

특정 조건만 만족하면 되는 만족 지표는 다수가 존재해도 되지만, 최대한 최적화해야하는 지표는 단 하나만 존재해야한다.

### F1 score

데이터가 불균형할 때도 모델을 평가할 수 있는 지표이다. Precision과 Recall라는 정확성에 대한 지표들이 있다. 이 지표들을 조화평균으로 합쳐 계산된다.

$$F1\,score = 2 \cdot \frac{Precision \cdot Recall}{Precision + Recall}$$

- Precision: True라고 예측한 것 중에서 실제로 True인 것의 비율
- Recall: 실제 True인 것 중에서 True라고 예측한 것의 비율

## Bayes error

이론적으로 도달 가능한 최소 오차를 뜻한다. 따라서 모델의 정확도는 **Bayes error**를 뛰어 넘을 수 없다.

bayes error와의 training error의 차이를 `Avoidable bias`라고 한다. `Avoidable bias`와 variance의 크기가 작은 걸 먼저 개선하는 식으로 모델을 개선한다.

> 인간의 능력으로 가능한 최소한의 에러는 Bayes error에 근접하므로 Human level error를 bayes error의 추정치로 사용한다.

> 모델의 성능은 bayes error를 뛰어넘을 수는 없지만 그에 근접한 Human level error는 뛰어넘을 가능성있다.

## Dev set error 분석

dev set에서 잘못 분류(예측)한 데이터를 분석한다. 분석한 결과 예를들어 개를 고양이로 잘못 분류한 것이 30%일 수 있다. 이 경우 개에 대한 학습을 진행하여 오류를 최대 30퍼 감소시킬 수 있다.

또한, 잘못 분류되어 있던 데이터의 라벨도 함께 분석할 수 있다. mislabeled 데이터의 비율도 분석하여 비율이 큰 경우 라벨링을 새롭게 할 필요가 있을 수도 있다. 하지만, 라벨링을 새롭게하는 것은 수고가 많이 들어가므로 신중히 결정해야한다.

> Dev set 분석은 보통 error가 적기 때문에 error의 데이터를 수동으로 하나씩 보며 분석하곤 한다.

## 훈련과 dev/test의 분포가 일치하지 않은 경우

실제 동작 환경의 데이터 세트(dev/test)가 훈련을 위해 모은 데이터의 분포와 다를 수 있다.

만약 실제 환경의 데이터의 양이 훈련 데이터에 비해 적다면, 둘을 하나로 합쳐 하나의 분포로 통일하는 것은 좋지 못하다. dev/test가 목표로 하는 실제 환경의 분포를 반영하지 않고 대부분을 차지하는 training 상황을 더욱 나타낼 것이기 때문이다.

따라서 분포가 다르더라도 dev/test에는 실제 타켓 환경의 분포로 만드는게 중요하다.

### Training-dev set

train과 dev가 다른 분포라면 당연하게도 variance가 발생할 수 밖에 없다. 이 variance의 크기가 어느 정도가 진짜로 문제가 되는 variance인지 알 수 없으므로 새로운 set을 만든다.

training-dev set은 훈련 세트와 동일한 분포를 가지지만 훈련에 사용되지 않는 분포이다. 이 경우 training-dev set과 training set의 error를 비교하여 수정해야할 variance를 파악할 수 있다.

trainig-dev error와 dev error 간의 차이는 data mismatch라고 표현한다.

- Human level error
  - ↕️ avoidable bias
- Trainig set error
  - ↕️ variance
- Traiinig-dev set error
  - ↕️ data mismatch
- Dev error
  - ↕️ degree of overfitting to dev set
- Test error

### Data mismatch 해결하기

error 분석을 통해 수작업으로 두 데이터 분포 간의 차이점을 파악한다. 에러를 일으키는 주요한 불일치들을 하나씩 해결한다.

> Artificial data synthesis 기법을 사용해서 훈련 데이터들에 실제 분포의 noise 등을 합성해 분포를 바꿀 수 도 있다.

## Transfer learning

같은 형태의 입력 데이터가 존재할 때, 다른 용도의 기존 모델을 사용하여 마지막 레이어만 바꿔 새로운 용도의 모델을 만들 수 있다. 즉, pre-trained model을 사용하여 fine-tunig 하는 것이다. 이는 초기 layer에서 데이터에 대한 특징 분석을 하기 때문이다. 뒤쪽의 레이어에서 결론 도출 부분만 바꾸고 초반 레이어의 특징 분석은 그대로 채용하는 방식으로 적은 데이터만으로도 좋은 모델을 만들 수 있다.

예시는 다음과 같다.

- 이미지 인식 모델 -> 영상 의학 모델
- 음성 인식 모델 -> wakeword/trigger 인식 모델

> 해당 모델을 학습하기에 데이터가 충분하지 않을 때 transfer learning을 사용한다.

## Multi task learning

이미지 속에 사람/신호등/자동차 등 이 있는 지 한번에 알고 싶을 수가 있다. 이 때 여러개의 별개 모델을 만드는 것이 아닌 출력 층에 뉴런이 여러개인 모델을 만들 수 있다.

## End to End deep Learning

데이터를 넣으면 신경망 모델이 처음부터 끝까지 알아서 분석하여 결과를 가져오는 것. 이렇게할 경우 모델이 충분히 학습할 수 있도록 매우 커다란 데이터 세트가 필요하다.

충분하게 많은 데이터를 확보하기 어렵기 때문에, end to end가 아닌 각 단계별로 나누어서 진행하는 경우가 많다. (e.g.) 사물 인식 -> crop 영역의 객체 분류

> 직접 사람이 손으로 감지할 특징을 설정하는 일은 수고가 많이 들며, end to end의 경우 인간이 특정하지 못한 새로운 특징을 모델이 알아서 감지할 가능성이 있으므로 정말 충분한 데이터와 복잡한 모델 설계가 있다면 end to end DNN도 좋은 선택이다.
