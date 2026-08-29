---
layout: single
title: "Dragon Hatchling 리뷰 - Transformer를 Neuron Graph로 다시 표현하기"
date: 2026-09-10 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/09/10/bdh-gpu-architecture.webp
---

Dragon Hatchling(BDH)은 attention과 working memory를 고차원 tensor 연산으로만 보지 않고, neuron-pair의 국소 상호작용과 synapse 변화로 다시 표현하려는 language model architecture다. 논문의 핵심은 실제로 학습한 tensor model인 BDH-GPU와, 이를 국소 graph dynamics로 환원한 BDH를 함께 제시했다는 데 있다.

## Transformer 대체재보다 뇌형 계산의 실험

Transformer를 곧 대체할 구조가 나왔다는 논문으로 읽지는 않았다. 더 흥미로운 지점은 언어 모델의 attention과 working memory를 neuron과 synapse가 있는 graph의 계산으로 다시 쓸 수 있고, 그 제약된 GPU 구현도 수천만에서 약 10억 parameter 규모까지 학습된다는 데 있다. 현재 Transformer 생태계의 직접적인 대안이라기보다, 뇌과학에서 가져온 국소성·희소 활성·synaptic plasticity를 실제 language model architecture와 연결하는 연구 방향에 가깝다.

여기서 “뇌에 가깝다”는 말은 범위를 좁혀야 한다. BDH는 다음 성질을 계산 구조에 넣는다.

- 모든 neuron이 같은 국소 규칙을 실행하는 graph model
- excitatory·inhibitory circuit과 양수 activation
- 함께 활성화된 neuron 쌍의 연결을 강화하는 Hebbian 형태의 빠른 state update
- 소수의 neuron만 활성화되는 sparse representation

반면 대규모 실험은 실제 sparse graph 위의 BDH가 아니라, 통신을 mean-field tensor 연산으로 바꾼 BDH-GPU에서 수행했다. 뇌의 학습을 재현했거나 뇌가 실제로 이 방정식을 쓴다는 증거도 아니다. 제목의 “missing link”는 Transformer의 macro operation을 neuron·synapse의 micro dynamics로 번역할 수 있다는 저자들의 이론적 framing이다.

## BDH와 BDH-GPU

BDH에서는 고정 graph가 장기 parameter를, 시간에 따라 바뀌는 edge weight $\sigma(i,j)$가 working memory를 맡는다. 입력이 graph 안에서 전파될 때 excitatory·inhibitory signal이 ReLU와 threshold를 거치고, 연속해서 활성화된 neuron 쌍은 $y_i x_j$만큼 synapse state를 갱신한다. 저자들은 이를 approximate implication과 Hebbian learning의 결합으로 해석한다.

이 국소 graph model은 그대로는 GPU에서 학습하기 어렵다. BDH-GPU는 neuron끼리 wire로 연결하는 대신 모든 neuron이 낮은 차원 message를 주고받는 mean-field 형태로 제한한다. 세 parameter matrix $E$, $D_x$, $D_y$가 neuron dimension $n$과 작은 hidden dimension $d$ 사이를 오가며, 전체 scalable parameter 수는 약 $3nd$다. 같은 parameter를 모든 layer가 공유하므로 model size는 주로 neuron 수 $n$ 하나로 늘어난다.

$$
\rho_{t,l}=\rho_{t-1,l}+\operatorname{LN}(Ey_{t,l-1})x_{t,l}^{\mathsf T}U
$$

$$
x_{t,l}=x_{t,l-1}+\left(D_x\operatorname{LN}(Ey_{t,l-1})\right)^+
$$

$$
y_{t,l}=\left(D_y\operatorname{LN}(\rho_{t-1,l}x_{t,l})\right)^+\odot x_{t,l}
$$

- $x_{t,l},y_{t,l}\in\mathbb{R}_{\ge 0}^{n}$: $t$번째 token, $l$번째 layer의 neuron activation이다. ReLU로 양수이며 $y$는 학습 뒤 대체로 희소해진다.
- $E\in\mathbb{R}^{d\times n}$: $n$차원 activation을 작은 value dimension $d$로 압축한다.
- $D_x,D_y\in\mathbb{R}^{n\times d}$: 낮은 차원 message를 neuron dimension으로 되돌린다.
- $\rho_{t,l}\in\mathbb{R}^{d\times n}$: layer별 recurrent state다. key와 value의 outer product를 누적하는 linear attention memory이며, BDH의 synapse matrix $\sigma$를 압축한 표현이다.
- $U$: RoPE나 ALiBi에 해당하는 위치 회전·감쇠다.
- $(\cdot)^+$와 $\odot$: 각각 ReLU와 원소별 곱이다. 마지막 곱은 현재 활성화된 neuron만 통과시키는 gate로 작동한다.

![ReLU-lowrank block과 linear attention으로 구성된 BDH-GPU architecture](/assets/images/2026/09/10/bdh-gpu-architecture.webp)

왼쪽의 ReLU-lowrank block은 $n$차원 neuron signal을 $d$차원으로 압축했다가 다시 펼친다. 오른쪽 linear attention은 $v^*x^{\mathsf T}$의 rank-1 update를 recurrent state $\rho_l$에 더한다. Transformer의 token별 KV cache 대신 크기가 고정된 $d\times n$ state를 layer마다 유지하는 구조다.

BDH-GPU를 local BDH로 옮길 수 있다는 주장은 이론적 환원이다. Sparse circuit graph가 $E$, $D_x$, $D_y$의 low-rank 연산과 attention을 같은 $O(nd)$ parameter와 state로 표현할 수 있음을 보인다. 따라서 “graph model을 10억 parameter로 직접 훈련했다”가 아니라 “훈련 가능한 tensor form과 동등한 표현력을 갖는 graph form을 구성했다”가 정확하다.

## 실험에서 확인된 범위

### GPTXL과의 scaling 비교

주요 scaling 실험은 Europarl의 English-Polish·English-Czech data 380MB를 raw UTF-8 byte로 처리한다. 모든 model은 1.2B byte token, 약 3 epoch를 학습했고 minibatch 길이는 2,048 byte token이었다. BDH-GPU는 recurrent state를, GPT-2형 TransformerXL baseline인 GPTXL은 최근 4,096개 KV entry를 minibatch 사이에 이어받았다.

비교 범위는 25M·50M·100M·200M·400M·800M parameter다. BDH 계열은 $d=256$, 8개 shared layer, 4개 head를 고정하고 $n$만 32,768에서 1,048,576까지 늘렸다. 그림에서 vanilla BDH-GPU는 GPTXL보다 계속 높은 validation loss를 보인다. xLSTM 계열 conditional gate와 모든 layer의 logit 결합을 추가한 BDH-GPU′가 전 규모에서 GPTXL과 비슷하고 800M에서는 조금 낮은 loss를 기록한다.

![BDH-GPU, BDH-GPU prime과 GPTXL의 parameter 규모별 validation loss](/assets/images/2026/09/10/bdh-gpu-scaling.webp)

따라서 “BDH가 GPT-2와 동급”이라는 요약에는 조건이 붙는다. 같은 data와 training regime의 raw-byte Europarl 번역·언어 모델링에서, 보강된 BDH-GPU′가 조정된 GPTXL baseline과 비슷한 scaling curve를 보였다. 현대 LLM benchmark 전반이나 같은 wall-clock·memory cost에서의 우위를 입증한 결과는 아니다.

### Sparse activation과 synapse state

저자들은 BDH-GPU의 양수 activation 가운데 $y$가 보통 약 5%만 활성화된다고 보고한다. 반복되는 8글자 pattern을 외우는 synthetic task에서는 layer 2의 활성 비율이 처음 정보를 저장할 때 4.0–7.5%, 이미 아는 pattern을 반복할 때 약 2.5%로 줄었다. 예측 가능한 입력일수록 state에 새로 쓸 것이 적다는 해석이다.

Europarl로 학습한 8-layer, $n=49{,}152$, $d=256$ model에서는 “currency”와 “country” 문장을 구분하는 특정 $\sigma$ entry를 찾았다. 같은 synapse가 영어의 “British Pound”와 프랑스어의 “livre sterling” 같은 표현에 함께 반응했다. Currency 관련 문장 50개와 currency를 언급하지 않은 정치 문장 50개를 비교한 검정에서 $p<10^{-14}$, rank-biserial correlation 0.86을 보고했다.

![Currency와 country 관련 문장에 반응하는 BDH synapse state](/assets/images/2026/09/10/bdh-concept-synapses.webp)

이 결과는 state 자체를 neuron-pair 단위로 읽을 수 있다는 가능성을 보여준다. 다만 “model 전체가 monosemantic하다”는 증거는 아니다. 연구진이 개념 유무를 잘 구분하는 entry를 사후 탐색했고, 소수 개념과 작은 평가 세트에서 확인한 사례다.

Trained parameter의 $D_xE$, $D_yE$를 neuron adjacency로 읽고 큰 양수 edge만 남기면 random graph나 random low-rank baseline보다 높은 modularity와 heavy-tailed degree distribution이 나타났다. 이것도 BDH-GPU가 명시적인 sparse graph를 학습했다는 뜻은 아니다. Dense low-rank matrix product를 threshold한 뒤 드러나는 유효 graph를 분석한 결과다.

## Fast weights와 linear attention을 neuron graph로 읽기

- McCulloch–Pitts neuron과 spiking neural network 계열은 계산을 개별 neuron의 firing과 국소 회로로 설명한다.
- Hebbian learning은 함께 활성화되는 neuron 사이의 synapse를 강화한다. BDH는 이 규칙을 inference 중 빠르게 바뀌는 state에 사용한다.
- Fast weights는 activation보다 오래가고 학습 parameter보다 빠르게 바뀌는 associative memory를 둔다. BDH의 $\sigma$와 BDH-GPU의 $\rho$가 이 계보에 놓인다.
- Linear attention은 key-value outer product의 누적을 고정 크기 recurrent state로 바꿔 Transformer를 RNN/SSM처럼 실행할 수 있게 한다. BDH-GPU는 이 state를 큰 양수 neuron dimension에 맞추고 ReLU-lowrank block과 결합한다.
- BDH가 더한 것은 이 tensor 계산을 다시 동일한 local rule을 실행하는 neuron graph로 환원하고, parameter·activation·state를 같은 neuron 좌표계에서 읽으려 한 점이다.

기술적 의의는 새로운 benchmark 점수보다 표현의 연결에 있다. Transformer 계열의 attention, fast-weight memory, sparse activation을 graph의 local dynamics와 같은 언어로 기술해 architecture 분석과 brain model 사이에 비교 가능한 중간층을 만들었다.

## 아직 graph model로 입증하지 못한 것

- 모든 대규모 학습과 실험은 BDH-GPU에서 수행됐다. Local graph BDH의 대규모 학습 효율과 실제 neuromorphic hardware 성능은 검증하지 않았다.
- “뇌 모델” 주장은 기능적·계산적 대응이다. Neural recording이나 생물학 실험으로 local kernel을 검증하지 않았고, 저자도 brain science에 더 깊게 grounding하는 것을 다음 단계로 둔다.
- Inference state는 Hebbian 형태로 갱신되지만 model parameter는 standard backpropagation과 truncated backpropagation through time으로 학습했다. 뇌의 장기 학습이나 short-term state가 long-term weight로 옮겨가는 과정은 설명하지 못한다.
- Hard context window가 없다는 말은 무한한 기억을 보장한다는 뜻이 아니다. $n\times d$ state의 구분 능력은 유한하고 오래된 signal의 noise를 막기 위한 감쇠가 필요하다.
- Scaling 근거는 raw-byte Europarl 중심이며, 핵심 비교에서 vanilla BDH-GPU가 아니라 보강된 BDH-GPU′가 GPTXL과 맞먹는다. 일반 language understanding, reasoning, safety, 실제 serving cost로 결론을 넓힐 수 없다.
- Monosemantic synapse와 scale-free graph는 선택·threshold 방법에 의존하는 초기 사례다. 여러 domain과 독립 model에서 같은 구조가 안정적으로 재현되는지 남아 있다.

## 참고 자료

- [The Dragon Hatchling: The Missing Link between the Transformer and Models of the Brain](https://arxiv.org/abs/2509.26507)
- [pathwaycom/bdh](https://github.com/pathwaycom/bdh)
