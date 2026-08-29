---
layout: single
title: "Muon optimizer 원리 - Momentum update를 Newton-Schulz로 직교화하기"
date: 2026-08-14 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/08/14/muon-optimizer-hero.png
---

Muon은 SGD momentum이 만든 2차원 행렬 update를 Newton–Schulz iteration으로 근사 직교화한 뒤 weight에 적용하는 optimizer다. 이름도 **MomentUm Orthogonalized by Newton-Schulz**에서 왔다.

![불균형한 행렬 업데이트를 반복적인 행렬곱으로 직교화해 방향별 크기를 정돈하는 Muon optimizer](/assets/images/2026/08/14/muon-optimizer-hero.png){: .align-center }

AdamW가 parameter 원소마다 step size를 조절한다면, Muon은 hidden weight matrix 전체를 하나의 선형 연산자로 본다. Update가 몇 개의 강한 singular direction에 몰리는 현상을 줄이고, 이미 존재하지만 작았던 방향의 비중을 키우는 접근이다.

Muon은 AdamW의 완전한 대체제가 아니다. Attention·MLP·MoE의 2D weight에는 Muon을 쓰고, embedding·prediction head·bias·normalization scale에는 AdamW를 함께 쓰는 구성이 일반적이다.

## AdamW와 무엇이 다른가

| | AdamW | Muon |
| --- | --- | --- |
| 보는 단위 | 행렬을 이루는 개별 원소 | 2D weight matrix 전체 |
| 핵심 상태 | 원소별 1차·2차 moment | 행렬 형태의 momentum 하나 |
| Update 보정 | 원소별 gradient 크기와 분산으로 나눔 | Singular direction별 크기를 평탄화 |
| 주 적용 대상 | 거의 모든 종류의 parameter | Hidden layer의 2D weight matrix |
| 추가 연산 | Element-wise 연산 | Newton–Schulz matrix multiplication |

Muon은 gradient 자체보다 **momentum으로 누적된 행렬 update의 방향별 세기**를 다시 맞춘다.

## Muon update의 흐름

Weight matrix $W\in\mathbb R^{n\times m}$와 현재 gradient $G_t$가 있다고 하자.

$$
M_t=\mu M_{t-1}+G_t,
$$

$$
N_t=\mu M_t+G_t,
$$

$$
O_t\approx\operatorname{Ortho}(N_t),
$$

$$
W_t=W_{t-1}-\eta O_t.
$$

- $M_t$: SGD momentum buffer
- $N_t$: Nesterov-style momentum을 반영한 raw update
- $O_t$: Newton–Schulz iteration으로 근사 직교화한 update
- $\mu$: momentum coefficient
- $\eta$: learning rate

핵심 순서는 **momentum을 먼저 만들고 그 결과를 직교화한다**는 것이다. Gradient를 먼저 직교화한 뒤 momentum을 누적하는 Orthogonal-SGDM과 순서가 다르다.

원조 공개 구현의 일반적인 설정은 momentum 0.95, Nesterov 사용, Newton–Schulz 5회다.

## Update를 직교화한다는 뜻

Raw update $G$의 singular value decomposition이 다음과 같다고 하자.

$$
G=U\Sigma V^T
$$

$G$와 가장 가까운 semi-orthogonal matrix는 $UV^T$다.

$$
\operatorname{Ortho}(G)=UV^T
$$

$U$와 $V$가 나타내는 singular direction은 유지하고, $\Sigma$에 들어 있던 서로 다른 singular value를 같은 크기로 맞춘다.

예를 들어 raw update의 singular value가

$$
\Sigma=\operatorname{diag}(20,2,0.1)
$$

이라면 이상적인 직교화 결과는

$$
\Sigma'=\operatorname{diag}(1,1,1)
$$

에 해당한다. 첫 번째 방향이 update를 독점하지 못하게 하고, 작았던 방향도 비슷한 크기로 반영한다.

직교화 대상은 weight $W$ 자체가 아니라 이번 step의 update다. Muon을 사용한다고 학습된 weight matrix가 orthogonal constraint를 만족하는 것은 아니다. 또한 raw update에서 singular value가 정확히 0인 방향은 여전히 0이므로, gradient에 없던 새 방향을 만들어내지도 않는다.

## Newton–Schulz로 SVD를 근사하기

매 step마다 모든 matrix에 정확한 SVD를 수행하면 너무 비싸다. Muon은 행렬곱만으로 $UV^T$에 가까워지는 Newton–Schulz iteration을 사용한다.

먼저 Frobenius norm으로 normalize한다.

$$
X_0=\frac{G}{\lVert G\rVert_F+\epsilon}
$$

그다음 다음 식을 반복한다.

$$
\begin{aligned}
X_k={}&aX_{k-1} \\
&+b(X_{k-1}X_{k-1}^T)X_{k-1} \\
&+c(X_{k-1}X_{k-1}^T)^2X_{k-1}
\end{aligned}
$$

SVD를 대입하면 singular vector는 유지되고 singular value에 다음 다항식이 반복 적용된다.

$$
\varphi(x)=ax+bx^3+cx^5
$$

원조 Muon의 tuned coefficient는 다음과 같다.

$$
(a,b,c)=(3.4445,-4.7750,2.0315)
$$

작은 singular value를 빠르게 키워 5회 정도의 반복으로 대략 비슷한 크기에 모은다. 연산 대부분이 matrix multiplication이라 GPU에 잘 맞고 BF16에서도 실행할 수 있다.

Newton–Schulz 5회는 model forward를 5번 더 실행한다는 뜻이 아니다. Backward가 끝나 gradient matrix가 만들어진 뒤 optimizer 내부에서 행렬곱을 반복하는 과정이다.

## 어떤 parameter에 적용하는가

| Parameter 종류 | 일반적인 optimizer | 이유 |
| --- | --- | --- |
| Attention·MLP·MoE의 hidden weight | Muon | 2D 선형 연산자의 singular structure를 이용할 수 있음 |
| Q·K·V projection | Muon | 논리적 matrix별로 분리해 적용 가능 |
| Embedding | AdamW | 원조 Muon의 경험적 제외 대상 |
| 최종 classifier·LM head | AdamW | 원조 Muon의 경험적 제외 대상 |
| Bias·RMSNorm scale·scalar gate | AdamW | 1D 또는 scalar라 행렬 직교화 대상이 아님 |

Muon의 persistent optimizer state는 momentum matrix 하나다. AdamW처럼 1차·2차 moment를 모두 저장하지 않아 상태는 단순하지만, 직교화할 때 온전한 matrix와 임시 buffer가 필요하다. ZeRO처럼 parameter를 원소 단위로 나누는 분산 학습과 결합할 때는 matrix를 다시 모으는 비용을 고려해야 한다.

## 대규모 LLM 학습으로 확장하기

2025년 *Muon is Scalable for LLM Training*은 두 가지 보정을 추가했다.

### Decoupled weight decay

장기 학습에서 weight와 layer output의 RMS가 계속 커지는 문제를 막기 위해 AdamW 방식의 weight decay를 결합한다.

$$
W_t=W_{t-1}-\eta_t(O_t+\lambda W_{t-1})
$$

### Matrix shape에 따른 update RMS 보정

Full-rank $A\times B$ semi-orthogonal update의 원소별 RMS는 다음과 같다.

$$
\operatorname{RMS}(O_t)=\frac{1}{\sqrt{\max(A,B)}}
$$

Matrix가 클수록 update 원소가 자동으로 작아지므로 $\sqrt{\max(A,B)}$를 곱해 shape 차이를 상쇄한다. 해당 연구는 AdamW와 update RMS를 맞추기 위한 경험적 상수 0.2도 사용했다.

$$
\begin{aligned}
W_t={}&W_{t-1} \\
&-\eta_t\left(
0.2\,O_t\sqrt{\max(A,B)}
+\lambda W_{t-1}
\right)
\end{aligned}
$$

이 연구는 자체 scaling-law 설정에서 AdamW와 같은 성능에 약 52%의 training FLOPs가 필요했다고 보고했고, 3B activated·16B total MoE인 Moonlight를 5.7T token까지 학습했다. 대규모 pre-training에서도 작동한다는 근거지만, 모든 architecture와 학습 단계에서 2배 효율을 보장하는 법칙은 아니다.

## DeepSeek-V4에서 사용한 Muon

[DeepSeek-V4](/papers/deepseek-v4/)는 대부분의 2D weight에 Muon을 적용하고 embedding, prediction head, RMSNorm과 mHC gate에는 AdamW를 사용한다.

원조 계수를 8회 적용해 singular value를 빠르게 끌어올린 뒤, 안정적인 계수 $(2,-1.5,0.5)$를 2회 적용하는 Hybrid Newton–Schulz를 사용한다. Momentum은 0.95, weight decay는 0.1이며 shape 보정 뒤 update RMS는 약 0.18로 맞춘다.

다만 DeepSeek-V4 보고서에는 같은 architecture를 AdamW로 학습한 직접 ablation이 없다. 최종 성능 중 Muon의 기여만 분리해서 읽을 수는 없다.

## 근거의 범위

- 가장 강한 근거는 pre-training이다. SFT에서도 사용할 수 있지만 pre-training과 다른 optimizer로 전환했을 때의 우위는 불분명하다.
- AdamW보다 sample efficiency가 높아도 Newton–Schulz와 분산 통신 때문에 step당 wall-clock이 항상 더 짧은 것은 아니다.
- Update 직교화는 gradient에 없던 방향을 새로 만들지 않는다.
- 결과는 matrix grouping, update RMS scale, learning rate, weight decay와 AdamW 보조 parameter 분리에 민감하다.
- RL에서도 같은 이득이 유지되는지는 아직 충분히 확인되지 않았다.

Muon의 핵심은 weight를 직교행렬로 만드는 데 있지 않다. Momentum update의 singular spectrum을 평탄화해, 학습 step 하나가 몇 개의 강한 방향에만 끌려가지 않도록 만드는 데 있다.

## 참고 자료

- [Muon: An optimizer for hidden layers in neural networks](https://kellerjordan.github.io/posts/muon/)
- [KellerJordan/Muon](https://github.com/KellerJordan/Muon)
- [Muon is Scalable for LLM Training](https://arxiv.org/abs/2502.16982)
- [DeepSeek-V4 Technical Report](https://arxiv.org/abs/2606.19348)
