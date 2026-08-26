---
layout: single
title: "Multi-token Prediction 논문 리뷰 - 한 위치에서 여러 미래 토큰을 학습하기"
date: 2026-08-26 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/08/26/multi-token-prediction-hero.png
---

일반적인 언어 모델은 각 위치에서 바로 다음 token 하나를 예측한다. Multi-token Prediction(MTP)은 같은 context representation으로 앞으로 올 여러 token을 함께 맞히도록 학습한다.

![미래 토큰을 병렬로 예측하는 구조와 hidden state를 순차 전달하는 구조의 비교](/assets/images/2026/08/26/multi-token-prediction-hero.png){: .align-center }

원 논문의 MTP와 DeepSeek-V3·V4의 구현은 구조가 다르다. 원 논문은 서로 독립된 future head를 병렬로 두고, DeepSeek는 이전 prediction depth의 hidden state와 정답 token embedding을 다음 depth에 넘기는 causal chain을 사용한다.

## 원 논문의 학습 목표

Next-token loss는 다음과 같다.

$$
\mathcal{L}_1=-\sum_t \log P_\theta(x_{t+1}\mid x_{1:t})
$$

MTP는 $x_{t+1}$부터 $x_{t+n}$까지를 같은 위치에서 감독한다.

$$
\mathcal{L}_n=-\sum_t\sum_{i=1}^{n}\log P_\theta(x_{t+i}\mid z_t)
$$

- $z_t=f_s(x_{1:t})$: shared Transformer trunk가 만든 현재 context representation
- $n$: next token을 포함해 예측할 미래 token 수
- $i$: 현재 위치에서 미래 target까지의 거리

각 future head는 중간의 정답 token을 입력으로 받지 않는다. 모두 같은 $z_t$에서 출발한다.

## Shared trunk와 독립 head

$$
P_\theta(x_{t+i}\mid x_{1:t})
=\operatorname{softmax}\left(f_u\left(f_{h_i}\left(f_s(x_{1:t})\right)\right)\right)
$$

```text
context x₁:ₜ
    ↓
shared Transformer trunk
    ↓ zₜ
    ├─ future head 1 ─┐ → xₜ₊₁
    ├─ future head 2 ─┤ → xₜ₊₂
    ├─ ...            ├─ shared unembedding
    └─ future head n ─┘ → xₜ₊ₙ
```

각 $f_{h_i}$는 독립적인 Transformer layer이고 큰 vocabulary projection인 unembedding matrix는 공유한다. MTP 모델과 NTP baseline의 총 parameter 수를 맞추기 위해 future layer를 추가한 만큼 shared trunk layer를 줄였다. 성능 차이가 단순한 parameter 증가에서 나오지 않도록 한 것이다.

## 병렬 구조지만 구현은 순차적일 수 있다

Head 사이에는 data dependency가 없으므로 모델 구조상 병렬이다. 그러나 모든 head의 vocabulary logits를 동시에 메모리에 올리면 peak memory가 커진다.

논문의 memory-efficient 구현은 trunk를 한 번 계산한 뒤 head별 forward와 backward를 하나씩 수행하고 logits를 바로 해제한다. 실행 schedule은 순차적이어도 앞 head의 예측이 뒤 head의 입력이 되는 autoregressive chain은 아니다.

본문은 training overhead가 없다고 설명하지만 공개 측정에서는 4-token 모델이 약 7–9% 더 오래 걸렸다. 계산과 FSDP 통신을 이상적으로 겹치지 못한 영향이다.

## 어디에서 효과가 컸나

300M부터 13B까지 code model을 비교했을 때 작은 모델에서는 MTP가 약했지만 규모가 커질수록 관계가 뒤집혔다. 13B MTP 모델은 비교 가능한 NTP 모델보다 HumanEval에서 12%, MBPP에서 17% 더 많은 문제를 풀었다고 보고됐다.

7B code model에서는 네 token 예측이 MBPP와 HumanEval에서 가장 일관되게 좋았다. 하지만 최적 $n$은 고정되지 않았다.

- APPS/Intro에서는 $n=6$이 더 좋았다.
- Byte-level model에서는 8-byte prediction이 유리했다.
- 자연어 객관식 평가에서 2-token model은 비슷했고 4-token model은 다소 낮았다.
- 요약에서는 2-token과 4-token model의 평균 ROUGE-L이 높았다.
- GSM8K는 학습량과 $n$에 따라 우위가 바뀌었다.

근거가 가장 강한 범위는 큰 모델의 code generation과 일부 생성 과제다. 더 먼 token을 맞힌다는 사실만으로 일반 reasoning이 좋아진다고 볼 수는 없다.

## 추론에서는 버리거나 draft로 쓴다

학습 뒤 추가 head를 제거하고 next-token head만 남기면 기존 autoregressive decoding과 같다. 이 경우 MTP는 shared representation을 바꾸는 auxiliary objective로만 작동한다.

Future head를 유지하면 self-speculative decoding에 사용할 수 있다.

1. 여러 head가 다음 token 후보를 한꺼번에 제안한다.
2. Main next-token model이 후보를 검증한다.
3. 연속해서 맞은 후보를 한 번에 수락한다.

별도의 draft model이 필요 없다. 논문의 7B 4-token model은 특정 greedy decoding 설정에서 code 3.0배, text 2.7배의 속도 향상을 보였다. Model, hardware, acceptance rate가 다른 환경에 그대로 적용되는 상수는 아니다.

## 왜 도움이 될까

저자들은 미래 전개를 크게 바꾸는 token에 더 강한 학습 신호가 생긴다고 설명한다. 중요한 token을 틀리면 여러 future loss가 함께 커지므로 shared representation이 장기 결과에 민감해진다는 가설이다.

Teacher forcing과 실제 generation의 차이를 완화한다는 설명도 있다. 다만 원 논문의 future head는 여전히 정답 context에서 학습하며 자신이 생성한 token을 다음 head에 넣지 않는다. Exposure bias를 직접 제거한다기보다 더 긴 결과를 설명하는 representation을 학습한다고 보는 편이 정확하다.

## DeepSeek의 MTP는 causal chain이다

DeepSeek-V3와 V4는 원 논문의 독립 head 대신 prediction depth를 순서대로 연결한다.

| 항목 | 원 논문 | DeepSeek-V3/V4 |
| --- | --- | --- |
| Future 관계 | 같은 trunk hidden에서 독립 예측 | 이전 depth hidden을 다음 depth가 사용 |
| 정답 future embedding | Head 입력에 없음 | 각 depth의 입력에 결합 |
| Prediction module | 독립 Transformer head | Depth별 Transformer block |
| Vocabulary output | Unembedding 공유 | Main output head 공유 |
| 공개 설정 | $n=2,4,6,8$ 등 | $D=1$ |

Main model이 위치 $i$에서 만든 hidden state를 $h_i^0$라고 하면 첫 MTP block은 그 state와 정답 $t_{i+1}$ embedding을 결합한다.

$$
h_i^{\prime 1}=M_1[
\operatorname{RMSNorm}(h_i^0);
\operatorname{RMSNorm}(\operatorname{Emb}(t_{i+1}))]
$$

결합한 sequence를 별도 Transformer block에 통과시키고 main output head로 $t_{i+2}$를 예측한다.

```text
main hidden hᵢ⁰ ───────────────→ output head → tᵢ₊₁
      │
      └─ + GT Emb(tᵢ₊₁)
         → MTP block 1 → hᵢ¹ → output head → tᵢ₊₂
```

$D=2$라면 다음 block은 $h_i^1$과 정답 $t_{i+2}$ embedding을 받는다. 이 때문에 depth 방향에는 causal dependency가 생긴다.

DeepSeek에서 $D$는 main next-token prediction 뒤에 붙는 **추가 prediction depth 수**다. 원 논문의 $n$은 next token을 포함한 전체 target 수다. 따라서 DeepSeek의 $D=1$은 원 논문의 $n=2$와 target 개수만 놓고 보면 대응한다.

## DeepSeek-V3와 V4의 실제 설정

공개된 V3와 V4는 모두 $D=1$이다. 각 위치에서 main model이 next token을 예측하고 MTP block 하나가 그다음 token을 추가로 맞힌다.

$$
\mathcal{L}_{\mathrm{total}}
=\mathcal{L}_{\mathrm{main}}
+\frac{\lambda}{D}\sum_{k=1}^{D}\mathcal{L}_{\mathrm{MTP}}^k
$$

V4는 V3의 prediction chain과 objective를 유지하지만 MTP block 내부는 V4 architecture를 따른다. mHC residual mixing, attention, MoE가 들어가며 embedding과 output head는 main model과 공유한다. “전략을 수정 없이 계승했다”는 말과 block 내부 구현이 완전히 같다는 말은 구분해야 한다.

학습 후 MTP block을 제거하면 일반 autoregressive model로 사용할 수 있다. Proposal module로 남겨 speculative decoding에 활용할 수도 있다. 즉 MTP의 학습 효과와 추론 가속은 연결돼 있지만 별도의 선택지다.

Multi-token Prediction은 모든 모델에 붙이면 reasoning이 좋아지는 만능 objective가 아니다. 더 먼 미래를 함께 설명하도록 representation을 압박하는 방법이며, 효과는 model scale, 데이터, tokenizer, $n$과 head 설계에 의존한다. DeepSeek 사례를 읽을 때도 원 논문의 병렬 head와 causal-chain 변형을 같은 구조로 부르지 않는 것이 중요하다.

## 참고 자료

- [Better & Faster Large Language Models via Multi-token Prediction](https://proceedings.mlr.press/v235/gloeckle24a.html)
- [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437)
- [DeepSeek-V4 Technical Report](https://arxiv.org/abs/2606.19348)
- [DeepSeek-V4 리뷰](/papers/deepseek-v4/)
