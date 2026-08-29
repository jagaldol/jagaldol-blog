---
layout: single
title: "Weight-sparse Transformer 리뷰 - 해석 가능한 Circuit을 학습하기"
date: 2026-09-12 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/09/12/weight-sparse-quote-circuit.webp
---

대부분의 weight를 정확히 0으로 둔 Transformer를 처음부터 학습하면, 특정 동작을 담당하는 circuit이 dense model보다 훨씬 작고 분리된 형태로 나타난다. 이 논문은 weight sparsity를 압축 기법이 아니라 **해석 가능한 계산 구조를 유도하는 inductive bias**로 사용한다.

## Sparse model을 dense model의 리모컨으로 쓰기

Sparse model 자체를 들여다보는 데서 끝나지 않는다. Dense model과 sparse model의 각 층을 bridge로 연결하면 sparse model을 dense model의 해석 가능한 proxy처럼 쓸 수 있다. Sparse circuit에서 찾은 node 신호를 bridge를 통해 dense model로 옮겨, dense model의 동작을 의도한 방향으로 바꾸는 경로가 생긴다.

내가 이 구조에서 떠올린 것은 일종의 “리모컨”이다. Sparse model에서 따옴표 종류처럼 사람이 이해할 수 있는 신호를 찾고 그 값을 바꾸면, bridge가 대응하는 dense activation의 변화로 번역한다. 논문의 두 실험에서는 이 개입이 실제 dense model의 다음 token 확률을 바꿨다. 다만 이는 독립된 완성형 “Bridge 모델”이 아니라, 함께 학습한 sparse proxy와 층별 변환을 이용한 초기 방법이다. 두 단순한 Python 과제에서 보인 결과이므로 dense model 전체를 충실히 제어한다고 일반화할 수는 없다.

Circuit 설명도 사후 이야기에 머물지 않았다. 중첩 list를 세는 회로가 context 전체의 평균에 깊이를 저장한다는 사실을 이해한 뒤, list가 길어지면 신호가 희석되어 실패한다는 조건을 미리 예측했다. 내부 계산을 안다는 것은 출력에 이름을 붙이는 일이 아니라, 아직 보지 않은 실패와 개입 결과를 예측하는 일에 가깝다.

## 무엇을 sparse하게 만들었나

여기서 sparsity는 sparse attention이나 Mixture-of-Experts가 아니다. GPT-2 계열 decoder-only Transformer의 embedding을 포함한 모든 weight와 bias에 비정형 sparsity를 적용한다. 가장 sparse한 model은 약 1,000개 weight 중 1개만 nonzero이고, 각 계산 지점의 activation도 대략 4개 중 1개만 남긴다.

학습의 각 step에서 AdamW update를 수행한 뒤 행렬마다 절댓값이 큰 weight만 남기며, 학습 전반부에 dense 상태에서 목표 nonzero 수 $L_0$까지 점차 줄인다. Gradient와 Adam moment는 dense하게 유지한다. 목적은 실제 연산 효율이 아니라 각 neuron이 읽고 쓸 수 있는 residual channel 수를 제한해, 하나의 개념과 계산이 여러 위치에 겹쳐 퍼지는 현상을 억제하는 것이다.

실험 model은 Python code로 사전학습하고, 따옴표 닫기·중첩 list·변수 type 추적처럼 답이 두 token 중 하나인 20개 next-token 과제로 평가했다. 범용 언어 능력 전체가 아니라 작은 알고리즘을 얼마나 국소화해서 읽을 수 있는지 보는 설정이다.

## Circuit을 찾고 검증하는 방법

Circuit의 node는 MLP neuron 하나, attention channel 하나, 또는 residual channel의 read/write 하나다. Edge는 이 node들을 잇는 nonzero weight 하나다. 각 과제에서 task loss와 남길 node 수를 함께 최소화하는 mask를 학습하고, 제거한 node는 사전학습 분포에서의 평균 activation으로 고정한다.

작은 circuit을 찾았다는 사실만으로 원래 계산을 설명했다고 볼 수는 없다. 저자들은 두 방향으로 인과성을 확인한다.

- **충분성:** circuit 밖의 node를 모두 mean ablation해도 과제를 계속 풀어야 한다.
- **필요성:** 전체 model에서 circuit node만 제거하면 해당 과제의 성능이 크게 무너져야 한다.

같은 사전학습 loss를 가진 dense model과 비교했을 때, weight-sparse model은 같은 task loss를 내는 circuit이 평균적으로 약 16배 작았다. Nonzero weight 수 $L_0$를 줄이면 circuit은 단순해지지만 capability가 떨어진다. 반대로 전체 parameter 공간을 넓히되 $L_0$를 고정하면 각 neuron의 연결을 더 흩어 놓을 여지가 생겨 capability와 circuit 크기가 함께 개선됐다. 논문의 scaling은 “sparse할수록 무조건 좋다”가 아니라, 전체 공간과 실제 연결 수를 별도 축으로 조절해 capability–interpretability frontier를 옮기는 결과다.

## 따옴표를 닫는 회로

![Double quote로 시작한 문자열을 같은 quote로 닫는 weight-sparse circuit](/assets/images/2026/09/12/weight-sparse-quote-circuit.webp)

따옴표 과제의 pruned circuit은 12개 node와 9개 edge만으로 거의 완벽하게 동작했다. 첫 MLP는 현재 문자열이 single quote인지 double quote인지 감지하고 종류를 residual channel에 기록한다. 뒤쪽 attention head는 quote 위치를 찾는 key와 종류를 운반하는 value를 사용해 마지막 token 위치로 이 정보를 복사하고, unembedding이 알맞은 닫는 따옴표를 출력한다.

여기서는 neuron activation의 의미, node 사이 weight, attention의 정보 이동이 하나의 짧은 알고리즘으로 이어진다. 그러나 저자들이 수작업으로 해석한 것은 단순해 보이는 세 과제를 골라 과제당 약 하루를 들인 결과다. 작은 circuit이 자동으로 사람이 이해할 수 있는 circuit을 뜻하지는 않는다.

## 회로 이해로 실패를 예측하다

중첩 list 회로에서는 `[` embedding에서 나온 open-bracket 신호를 attention head가 context 전체에 걸쳐 균등 평균한다. 이 평균값의 크기가 nesting depth 역할을 하고, 뒤의 attention head가 이를 threshold처럼 읽어 닫는 괄호를 하나 더 내야 하는지 결정한다.

![List 길이가 늘면서 nesting-depth 신호와 정답 확률이 함께 희석되는 현상](/assets/images/2026/09/12/weight-sparse-context-dilution.webp)

평균을 쓰므로 list 원소가 늘수록 bracket 신호가 $1/n_{\text{context}}$에 비례해 희석된다. 이 구조에서 긴 list 실패를 예측할 수 있었고, 앞선 주석에 관계없는 `[`를 넣어 flat list를 중첩 list로 오인시키는 공격도 만들 수 있었다. 비슷한 capability의 dense model에도 이 공격이 통했다. 이것이 같은 내부 circuit을 쓴다는 증명은 아니지만, sparse model에서 읽은 계산 motif가 dense model의 실패 조건을 찾는 단서가 될 수 있음을 보여준다.

단순화한 도식은 7개 node와 4개 edge지만, 실제 관련 component에는 도식에서 생략된 수백 개 edge와 수십 개 neuron이 더 있다. 해석 가능한 핵심 경로를 찾았다는 것과 model의 모든 연결을 설명했다는 것은 구분해야 한다.

## Bridges: sparse 해석을 dense model에 연결하기

![Dense와 sparse residual activation을 양방향으로 연결하는 bridge architecture](/assets/images/2026/09/12/weight-sparse-bridge-architecture.webp)

각 bridge는 dense activation을 sparse 쪽으로 보내는 linear encoder와 AbsTopK activation, sparse activation을 dense 쪽으로 되돌리는 linear decoder로 구성된다. Sparse model의 residual activation 전체가 latent가 된 sparse autoencoder와 비슷하지만, latent가 독립 feature 목록이 아니라 다음 층의 계산을 계속 수행하는 runnable model이라는 차이가 있다.

학습에는 세 종류의 정합성이 들어간다. 같은 층의 dense·sparse activation을 양방향으로 맞추고, 중간 한 지점에서 dense 계산을 sparse로 넘긴 경로와 sparse 계산을 dense로 넘긴 경로의 최종 분포가 원래 dense model과 가까워지도록 한다. 따라서 bridge는 단순한 activation 상관관계가 아니라, 변환 뒤에도 남은 층이 계산을 이어 갈 수 있도록 훈련된다.

![Sparse circuit의 신호를 바꿔 dense model의 다음 token 확률을 조정한 결과](/assets/images/2026/09/12/weight-sparse-bridge-steering.webp)

저자들은 과제에 필요한 sparse node를 고르고, 실제 activation과 반사실적 activation의 차이를 bridge로 dense residual stream에 사상했다. 따옴표 실험에서는 double-quoted prompt를 single-quoted 문자열처럼 닫게 만들었고, 제어문 실험에서는 `return True` 뒤에 newline 대신 colon을 낼 확률을 높였다. 그래서 “sparse model을 해석하고 dense model을 조종한다”는 리모컨 직관은 이 두 결과를 잘 요약한다. 다만 bridge가 보존한 표현의 범위와 faithfulness는 아직 과제별로 검증해야 한다.

## 해석용 model-organism이라는 방향

기존 mechanistic interpretability는 이미 학습된 dense model에서 attention head, neuron, feature와 circuit을 역공학한다. Superposition 가설은 한정된 차원에 더 많은 feature가 겹쳐 표현되면서 개별 neuron과 weight가 읽기 어려워진다고 본다. Sparse autoencoder와 transcoder는 dense activation을 학습 후 sparse feature basis로 바꾸지만, 그 basis에서 그린 circuit이 원래 model의 실제 계산을 얼마나 그대로 보존하는지 별도 검증이 필요하다.

이 논문은 반대편에서 출발한다. 모델을 처음부터 weight-sparse하게 학습해 low-level weight 자체가 짧은 circuit을 이루게 한다. 전통적인 pruning이 같은 model을 싸게 실행하기 위한 압축이라면, 여기서는 계산 효율을 희생하고도 해석 가능한 구조를 유도하는 것이 목적이다. Bridge는 이 model-organism 접근과 기존 dense model 분석 사이를 잇는다. Sparse model에서 공통 circuit motif를 배우거나, 좁은 행동 분포에 한정해 dense model의 해석·개입용 proxy를 만드는 방향이다.

## Frontier model과 거리가 남는 이유

- 같은 capability의 dense model보다 학습과 추론에 약 100–1,000배 많은 compute가 든다. 비정형 sparse 연산은 dense matrix multiplication용 hardware를 제대로 활용하기 어려워 frontier model의 대체재가 되기 힘들다.
- 결과는 작은 Python model과 단순한 binary next-token 과제에 집중되어 있다. 수천만 nonzero parameter를 넘어가도 작은 circuit과 사람이 읽을 수 있는 feature가 유지되는지는 확인되지 않았다.
- Compact circuit은 interpretability의 proxy일 뿐이다. Polysemantic node와 연속적인 feature magnitude가 남으며, 복잡한 행동에서는 circuit 자체가 크게 늘어날 수 있다.
- Mean ablation은 완전한 faithfulness 검증이 아니다. Distribution shift를 더 엄격히 통제하는 causal scrubbing 같은 검증이 필요하다.
- Pruning 결과에는 수작업 정리와 task-specific data가 들어간다. 공개된 몇 개의 깔끔한 회로를 model 전체의 투명성으로 확대해서 보면 안 된다.
- Bridge 실험은 sparse와 dense 계산이 완전히 동형이라는 증거가 아니다. 두 model 사이의 부분적 대응을 두 과제에서 행동 개입으로 확인한 단계다.

## 참고 자료

- [Weight-sparse transformers have interpretable circuits](https://arxiv.org/abs/2511.13653)
- [OpenAI 연구 PDF](https://cdn.openai.com/pdf/41df8f28-d4ef-43e9-aed2-823f9393e470/circuit-sparsity-paper.pdf)
