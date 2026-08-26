---
layout: single
title: "Semantic Tube Prediction 논문 리뷰 - LLM의 hidden trajectory를 곧게 만드는 보조 손실"
date: 2026-07-12 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/07/12/semantic-tube-prediction-hero.png
---

Semantic Tube Prediction(STP)은 한 token sequence 안에서 앞·뒤 hidden-state의 이동 방향을 맞추는 학습법이다. 별도의 target encoder나 두 view 없이 기존 forward의 hidden state를 재사용한다.

![휘어진 hidden-state 경로를 정렬해 곧은 semantic tube로 만드는 STP 개념](/assets/images/2026/07/12/semantic-tube-prediction-hero.png){: .align-center }

추가 Transformer forward와 추론 비용이 없다는 장점이 있다. 다만 논문의 기하학적 설명은 강한 가정 위에 있고, “16배 데이터 효율”은 1B 모델의 합성 정규식 과제에서 얻은 결과다.

## 출발점은 Geodesic Hypothesis

Next Token Prediction은 정답 token의 확률을 높이지만 hidden state의 정확한 위치까지 지정하지는 않는다. 저자들은 이상적인 token sequence가 hidden space의 매끄러운 manifold 위에서 국소적으로 geodesic, 즉 가장 짧은 경로를 따른다고 가정한다.

실제 hidden state가 이 경로에서 조금씩 벗어나면 오차가 누적되고 다른 의미의 trajectory로 넘어갈 수 있다는 설명이다. 이를 막기 위해 연속한 두 구간의 이동 방향을 정렬한다.

$$
\mathcal{L}_{\mathrm{STP}}
=
\mathbb{E}_{s<r<t}
\left[1-\cos(h_t-h_r,\,h_r-h_s)\right]
$$

- $h_r-h_s$: 앞 구간의 hidden-state 이동
- $h_t-h_r$: 뒤 구간의 hidden-state 이동
- $1-\cos(a,b)$: 두 방향이 같으면 0, 반대면 2

Loss가 작아질수록 trajectory는 중간점에서 덜 꺾인다. 관측할 수 없는 이상적 hidden state를 직접 맞히는 것이 아니라, 이상적인 경로가 locally linear하다는 가정 아래 쓰는 대리 목적함수다.

전체 학습은 NTP와 함께 진행한다.

$$
\mathcal{L}
=\mathcal{L}_{\mathrm{NTP}}
+\lambda\mathcal{L}_{\mathrm{STP}}
$$

$\lambda$가 너무 크면 의미상 휘어야 하는 trajectory까지 과도하게 직선화할 수 있다.

## LLM-JEPA보다 단순해진 부분

| LLM-JEPA | STP |
| --- | --- |
| 의미가 대응하는 두 view 필요 | 한 autoregressive sequence 사용 |
| 두 view를 독립적으로 encoding | 기존 per-token hidden state 재사용 |
| predictor token과 추가 forward 필요 | 추가 Transformer forward 없음 |
| 두 sequence의 최종 표현 정렬 | 한 sequence의 displacement 정렬 |

Pair dataset이 필요 없으므로 구조상 일반 말뭉치에도 붙일 수 있다. 하지만 원 논문이 실제로 검증한 범위는 pretrained model의 task fine-tuning이다.

## 여섯 과제의 결과

주 실험은 Llama-3.2-1B-Instruct를 NTP, LLM-JEPA, STP로 fine-tuning한 비교다. 수치는 5개 seed 평균이다.

| 데이터셋 | NTP | LLM-JEPA | STP |
| --- | ---: | ---: | ---: |
| NL-RX-SYNTH | 57.29 | 71.46 | **84.29** |
| NL-RX-TURK | 22.49 | 30.94 | **41.16** |
| GSM8K | 32.36 | 36.36 | **36.60** |
| Spider | 47.52 | 50.55 | **56.78** |
| NQ-Open | 20.12 | 21.59 | **26.59** |
| HellaSwag | 27.93 | 35.22 | **36.67** |

여섯 과제 모두 NTP보다 높았다. 합성·사람 작성 정규식과 Spider에서는 차이가 컸고, GSM8K와 HellaSwag에서는 LLM-JEPA와 비슷했다. 데이터셋마다 채점 방식이 달라 행 사이의 절대 수치를 직접 비교할 수는 없다.

## “16배 데이터 효율”의 정확한 조건

1B NL-RX-SYNTH 실험에서 STP는 500개 sample을 64 epochs 학습해 56.34를 얻었다. NTP는 8,000개를 4 epochs 학습해 57.29를 얻었다.

```text
STP:   500 unique samples × 64 epochs
NTP: 8,000 unique samples ×  4 epochs
```

고유 sample 수는 1/16이지만 반복 학습으로 update 수를 보상했다. 따라서 데이터셋의 고유 사례 수를 줄였다는 결과이지, 전체 compute도 1/16로 줄였다는 뜻은 아니다. 같은 설정의 3B 모델에서는 1/16 STP가 full-data NTP에 미치지 못했다.

논문이 Chinchilla-style scaling law를 넘어섰다고 표현한 부분도 이 합성 과제의 조건 안에서 읽어야 한다. 자연어 사전학습 전반의 데이터 요구량을 16배 줄였다는 증거는 아니다.

## 기하학적 설명은 아직 가설이다

STP의 이야기는 직관적이지만 몇 단계의 가정을 거친다.

1. Token append를 연속시간 ODE trajectory로 근사한다.
2. 추론 오차를 Brownian noise가 더해진 SDE로 본다.
3. 이상적인 semantic trajectory가 geodesic이라고 가정한다.
4. 국소 displacement 정렬이 그 경로 근처에 state를 붙잡는다고 본다.

실험은 accuracy 개선을 보여주지만 hidden trajectory가 실제 geodesic인지, signal-to-noise ratio가 설명한 방식으로 높아졌는지를 직접 검증하지는 않는다. 공개 코드의 기본 `random_span`도 논문식의 연속 구간과 구현상 차이가 있어 재현할 때 확인해야 한다.

후속 독립 연구에서는 STP 계열의 보조 loss가 hidden geometry를 바꾸더라도 decoded exact-match가 안정적으로 좋아지지는 않았다. 원 논문의 full fine-tuning과 16배 곡선을 그대로 재현한 결과는 아니므로 직접 반박이라기보다 적용 경계를 보여주는 근거에 가깝다.

STP는 계산 부담이 작은 hidden-trajectory regularizer로 볼 만하다. 현재 증거가 지지하는 범위는 특정 fine-tuning 과제에서의 개선이며, 보편적인 데이터 효율이나 hallucination 해결까지는 아니다.

## 참고 자료

- [Semantic Tube Prediction 논문](https://arxiv.org/abs/2602.22617)
- [공개 코드](https://github.com/galilai-group/llm-jepa#stp)
- [LLM-JEPA 리뷰](/papers/llm-jepa/)
- [Representation Without Reward](https://arxiv.org/abs/2605.15394)
