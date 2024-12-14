---
layout: single
title: "[Review] Attention Is All You Need"
date: 2024-12-14 15:51:00 +0900
categories: papers
---

기존 sequence 변환 모델은 RNN 또는 CNN 기반의 인코더-디코더 구조를 바탕으로 한다. 최고 성능의 모델들은 여기에 `attention mechanism` 을 사용하고 있는데, 우리는 RNN과 CNN을 배제한 채 **`attention mechanism` 만을 사용하는 새로운 구조 Transformer**를 제시한다!

- Recurrent가 사라짐으로 인해 완전 병렬화 가능(훈련 시간 단축)
- 번역 task에서 기존 모델들(앙상블 포함)보다 높은 성능으로 SOTA 달성
- 영어 구문 분석(English constituency parsing)에서도 우수한 성능 → 일반화 성능 또한 높다

## Background

기존 `LSTM`, `GRU` 등의 **RNN 사용** 시 다음과 같은 한계가 존재한다:

- 입력 시퀀스에 대해 **순차적인 계산이 필요**
- 이전 상태에 대한 기억으로 인해 메모리 제약이 높고 시퀀스 길이가 길어질 수록 batch 힘듬

순차적인 계산을 줄이기 위해 **CNN 을 사용**한 모델들(`Extended Neural GPU`, `ByteNet`, `ConvS2S`)이 존재했다. `CNN` 을 사용하면 병렬로 계산이 가능하지만, **시퀀스 내에 멀리 떨어져 있는 토큰과의 의존성을 학습하기가 어려운** 단점이 존재한다.

## Model Architecture

![figure1](/assets/images/2024/12/14/figure1.png)

입력 문장의 토큰들은 왼쪽 `Encoder`로 들어가고, 출력 문장의 토큰들은 오른쪽 `Decoder`로 들어간다.

**입력 토큰들의 각 벡터**는 `Attention layer`를 거치며 다른 토큰들과 관계 속에서 풍부한 표현으로 변한다. 이는 **최종적으로 `decoder`에게 `key`, `value`로 전달**된다.

출력 문장(번역 문장)의 토큰들은 먼저 `masked`를 하며 `attention`을 적용한다. 이는 훈련과정에서 미래의 토큰을 확인할 수 없게 막는 것이다.($-\infty$으로 세팅)

이후 **`enocder`로부터 전달받은 `key`, `value`를 확인하고 원본 문장을 참고한 적절한 표현으로 치환**된다. 이러한 디코더 구조를 거쳐 **다음에 위치할 토큰의 확률을 추정하고 이를 학습**하게 된다.

> 추론 시는 단순히 `decoder`가 재귀적으로 동작하며 다음 토큰을 하나씩 예측한다.

- Encoder 블럭 및 Decoder 블럭 개수: N=6
- Residual 및 Norm: $LayerNorm(x + sublayer(x))$

### Multi-Head Scaled Dot-Product Attention

![figure2](/assets/images/2024/12/14/figure2.png)

각 Attention 블럭의 세부 구조는 위와 같다. Q, K, V로 들어온 각 값을 선형 변환하여 각 head의 attention 계산에 사용된다.

시퀀스의 각 토큰들에 대해 단계별로 Query, Key, Value가 각각 존재한다. **Attention 계산이란, 현재 위치의 query를 사용하여 key의 일치도에 따라 전체 시퀀스에 대한 attention 정도를 계산 후, 비율대로 각 value를 weighted sum 하는 것**이다.

$$\operatorname{Attention}(Q, K, V)=\operatorname{softmax}\left(\frac{Q K^T}{\sqrt{d_k}}\right) V$$

여기서 scale이 존재하는 데 이는 dot product 이후 차원 크기에 따라 분산이 커지는데 이를 정규화하기 위함이다.

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$

$$\text{where } \text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$$

이러한 Attention을 병렬적으로 사용하여 MultiHead 구조를 구성한다. 이는 각 Head가 서로 다른 표현 공간에서 attention을 적용할 수 있게 해주며 풍부한 표현을 얻을 수 있게 해준다.

### 3가지 형태의 Attention

- **Encoder-Decoder Attention**
  - Decoder의 이전 출력에서 Query, Encoder의 출력에서 Key, Value
  - 디코더의 모든 위치가 입력 시퀀스 전 범위에 attention 가능
- **Encoder Self Attention**
  - 이전 레이어의 출력을 Query, Key, Value로
  - 인코더의 각 위치가 인코더 내 모든 위치에 attention
- **Decoder Self Attention**
  - 이전 레이어의 출력을 Query, Key, Value로
  - masking을 통해 decoder의 각 위치가 현재 및 이전 위치에만 attention

### Position-wise Feed-Forward Networks

Attention 외에도 sub-layer가 존재한다.

$$\text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2$$

> 내부 계층의 차원은 $d_{ff} = 2048$ 이다.

### Positional Encoding

LSTM 기반 모델과 다르게 각 토큰들은 거리나 순서 없이 동등하게 attention 계산이 이루어진다. 따라서 우리는 **위치 정보를 모델에 전달할 필요**가 존재한다. 이를 위해 입력 임베딩에 임베딩 차원과 동일한 위치 인코딩을 더해준다.

$$\text{PE}(\text{pos}, 2i) = \sin\left(\frac{\text{pos}}{10000^{\frac{2i}{d_{\text{model}}}}}\right)$$

$$\text{PE}(\text{pos}, 2i + 1) = \cos\left(\frac{\text{pos}}{10000^{\frac{2i}{d_{\text{model}}}}}\right)$$

삼각함수를 활용함으로써, 학습되지 않은 길이(+300)에 대한 시퀀스도 잘 처리할 수 있음을 확인하였다.

> 고정된 offset k에 대해 어떤 위치에서 k만큼 이동하였을 때, 삼각함수의 덧셈정리에 의해 인코딩의 변화가 선형적으로 표현이 가능하다.

## Self-Attention, Recurrent, Convolution 비교

![table1](/assets/images/2024/12/14/table1.png)

- **Complexity per Layer**
  - n < d 인 경우, self attention이 더 빠르다
  - 모델의 dimension을 크게 설정하기 때문에 대부분의 경우 조건을 만족한다.
- **Sequential Operations**
  - 병렬 계산에 대한 복잡도로 recurrent의 경우 sequence 길이 만큼 반복해야한다.
- **Maximum Path Length**
  - 네트워크 내 장거리 의존성 사이의 path length
  - self-attention은 항상 모든 토큰과 직접 연결되므로 path lengh가 1이다.
  - O(1)을 달성함으로써 장기 메모리에 대한 문제도 해결하였다.

## Training

아래는 논문에서 모델 학습에 사용한 환경 및 `hyper parmamter`들이다:

- 450만 개의 문장 쌍 WMT 2014 English-German 데이터셋
  - 37,000개 vocab, BPE
- 3,600만 개의 문장 쌍 WMT 2014 English-French 데이터셋
  - 32,000개 vocab, Word-piece
- NVIDIA P100 GPU \* 8개
- 학습 시간
  - base model - 0.4s/step \* 100,000 steps = 12hours
  - big model - 1.0s/step \* 300,000 steps = 3.5days
- Adam
  $$\beta_1 = 0.9, \, \beta_2 = 0.98, \, \epsilon = 10^{-9}$$
- lr ( warmup\*steps = 4000)
  $$\text{lrate} = d_{\text{model}}^{-0.5} \cdot \min(\text{step\_num}^{-0.5}, \, \text{step\_num} \cdot \text{warmup\_steps}^{-1.5})$$
- Residual Dropout = 0.1
- Label Smoothing = 0.1

## Result

![table2](/assets/images/2024/12/14/table2.png)

최종적으로 영어-독일어 번역에서 big Transformer 모델이 이전의 최고 성능 모델들을 2.0 BLEU 이상 초과하여 28.4를 달성하였다. 영어-프랑스어 번역에서도 41.0으로 단일 모델에서 가장 높은 성능을 기록하였고, 이는 1/4 미만의 훈련 비용으로 달성한 것이다.

### 각 구성요소에 대한 평가

아래는 Transformer의 각 구성요소들을 평가하기 위해 다양한 방식으로 변형하여 성능 변화를 측정한 결과이다:

![table3](/assets/images/2024/12/14/table3.png)

1. **Attention Head 개수**(계산량 유지)
   - 단일 head는 낮은 성능을 보이며, 너무 많은 head를 사용한 경우에도 품질 저하가 발생가능
2. **key 차원**
   - 각 위치에 대한 compatibility 를 계산하는 것이 어려운 task라는 걸 의미
3. 모**델 크기**
   - 모델 규모가 커질수록 성능이 향상
4. d**ropout**
   - 적용 시 과적합을 막는데 도움
5. **학습된 위치 임베딩**
   - 성능이 거의 동일
   - 절대 위치 인코딩이 학습안된 길이 처리에 강점이 있어 sinusoid 사용

### English Constituency Parsing

RNN 기반의 seq2seq 모델은 소규모 데이터 환경에서 SOTA를 달성하지 못했었다. 하지만, Transformer는 매우 우수한 성능을 보이면서 일반화 성능 또한 입증했다.

![table4](/assets/images/2024/12/14/table4.png)

## Conculusion

Transformer는 전적으로 attention에 기반한 최초의 sequence transduction 모델이다. 번역 task에서는 더 적은 학습비용으로 앙상블 모델을 포함한 기존의 모델들을 이기고 SOTA를 달성하였다.

attention 기반 모델의 미래에 대해 기대를 가지고 있으며, 텍스트 뿐만 아니라 이미지, 오디오, 비디오 같은 다양한 형태의 데이터에 대해 적용하고 실험할 예정이다!

## 후기

NLP, LLM을 공부하며 Transformer는 나에게 매우 친숙한 모델이였다. 여러 강의를 통해 세부 구조도 이미 알고 있었지만 이번 기회에 제대로 논문을 읽고 정리해보았다.

나는 LLM을 먼저 접하고 이후에 Transformer를 배우게 된사람으로써, 논문을 읽고 해당 시기에 transformer가 생성형 task가 아닌 번역 task를 위해 고안되었다는 것을 새삼 깨닫게 되었다. 단순히 transformer를 공부하는 것이 아닌 해당 논문이 출판된 시기의 배경까지 어렴풋이 알 수 있었고 머리 속이 맑게 개이는 기분이였다.

생각보다 자세히 아는 내용이라 그런지 재밌게 읽혔고 다음에 읽을 BERT도 재밌게 읽어봐야겠다!
