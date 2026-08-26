---
layout: single
title: "QAT 양자화란 무엇인가 - 4비트 로컬 LLM의 품질 손실을 줄이는 방법"
date: 2026-06-04 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/06/04/qat-local-llm-hero.png
---

로컬 LLM은 같은 모델도 BF16, Q8, Q4에 따라 필요한 메모리와 품질이 크게 달라진다. 4비트로 줄이면 실행 가능한 모델의 체급이 올라가지만, 압축 과정에서 다음 토큰 확률분포가 흔들릴 수 있다.

![큰 신경망을 보정 장치로 압축해 작은 신경망으로 만드는 QAT 개념](/assets/images/2026/06/04/qat-local-llm-hero.png){: .align-center }

QAT(Quantization-Aware Training)는 모델이 낮은 비트로 실행될 상황을 학습 과정에서 미리 겪게 해 이 손실을 줄이는 방법이다.

## PTQ와 QAT의 차이

일반적인 로컬 GGUF는 학습이 끝난 모델을 나중에 줄이는 PTQ(Post-Training Quantization)로 만든다.

```text
PTQ
학습 완료된 BF16/FP16 모델
→ calibration 또는 변환
→ Q4/Q8 모델 배포
```

QAT는 학습이나 후반 보정 중에 양자화 오차를 시뮬레이션한다.

```text
QAT
고정밀 모델
→ 낮은 비트에서 생길 rounding·clamping을 학습 중 모사
→ 그 오차에 맞춰 weight 보정
→ 실제 quantized checkpoint export
```

| 방식 | 장점 | 단점 |
| --- | --- | --- |
| PTQ | 이미 학습된 모델을 빠르고 싸게 여러 포맷으로 변환 | 공격적인 4비트에서 품질 손실이 커질 수 있음 |
| QAT | 목표 bit-width의 오차를 모델이 미리 보며 적응 | 추가 학습과 검증이 필요 |

QAT 모델이라고 해서 사전학습 전체를 4비트로 수행했다는 뜻은 아니다. 완성된 모델에 짧은 accuracy-recovery fine-tuning을 적용한 경우도 많다.

## 학습 중에는 fake quantization을 쓴다

QAT가 모든 계산을 실제 4비트로 수행하는 것은 아니다. 보통 고정밀 weight를 유지하면서 forward pass에서 양자화 효과를 흉내 낸다.

```text
고정밀 weight
→ scale에 맞춰 rounding과 clipping
→ 양자화된 것처럼 forward
→ loss 계산
→ 고정밀 weight 업데이트
```

역전파가 가능한 고정밀 값을 남겨두고, 모델이 실제 배포 시 생길 오차를 반복해서 보게 만드는 방식이다.

### scale과 zero-point

연속적인 실수 weight를 제한된 정수 구간에 옮길 때 두 값이 필요하다.

- `scale`: 정수 한 칸이 원래 실수 공간에서 차지하는 간격
- `zero-point`: 정수 표현의 0이 원래 실수 공간에서 대응하는 위치

4비트는 표현할 수 있는 값이 적다. scale을 어떻게 잡고 weight를 어떤 block으로 나누는지가 품질에 큰 영향을 준다.

## 품질 보존은 무엇으로 측정하나

### Mean KLD

Kullback-Leibler Divergence는 원본 모델과 양자화 모델의 다음 토큰 확률분포가 얼마나 다른지 본다. 낮을수록 원본 분포에 가깝다.

```text
BF16의 다음 토큰 분포
vs
Q4 모델의 다음 토큰 분포
```

### Top-1 일치율

원본 모델과 양자화 모델이 같은 1등 토큰을 고른 비율이다. 높을수록 좋지만, 2등 이하 분포와 긴 생성의 누적 오차까지 보장하지는 않는다.

### 실제 작업 평가

KLD와 Top-1이 좋아도 다음 작업은 직접 확인해야 한다.

- 한국어 장문 생성
- 코드와 수학 추론
- 긴 context
- JSON 같은 구조화 출력
- tool calling
- 멀티턴 대화

토큰 수준 유사도는 작업 성공률과 같은 지표가 아니다.

## Gemma QAT 사례에서 볼 것

Google은 Gemma 계열의 공식 QAT checkpoint와 여러 배포 형식을 제공한다.

| 형식 | 용도 |
| --- | --- |
| Unquantized QAT checkpoint | 연구와 custom compilation |
| GGUF Q4 | llama.cpp·LM Studio·Ollama 계열 로컬 실행 |
| Mobile optimized | 모바일·온디바이스 runtime |
| Compressed Tensors | vLLM 같은 서버 runtime |

여기서 Google의 공식 QAT checkpoint와, 제3자가 이를 GGUF로 변환하며 수행한 accuracy recovery를 구분해야 한다.

```text
Google
→ QAT checkpoint 공개

GGUF 변환 도구
→ checkpoint를 특정 quant 포맷으로 변환
→ 변환 손실을 줄이기 위한 추가 보정과 벤치마크
```

커뮤니티 결과가 좋더라도 공식 모델 전체의 보장으로 확대하면 안 된다.

## 벤치마크를 읽는 네 가지 질문

1. **QAT checkpoint가 공식인가?** 모델 제작사가 공개한 것인지, 제3자의 자체 보정인지 구분한다.
2. **비교 bit-width가 같은가?** QAT Q4와 Q8을 비교하면 속도와 메모리 이득이 자연스럽게 발생한다.
3. **무슨 품질을 측정했는가?** KLD, Top-1, benchmark accuracy, 실제 생성 성공률은 서로 다르다.
4. **어떤 하드웨어와 runtime인가?** CUDA, ROCm, Metal, CPU와 context 길이에 따라 결과가 달라진다.

가장 의미 있는 비교는 같은 모델과 runtime에서 `QAT Q4`, naive `Q4_0`, `Q4_K_M`, 가능하면 BF16을 함께 재는 것이다.

## QLoRA와는 목적이 다르다

QLoRA는 적은 VRAM으로 fine-tuning하기 위한 방법이다.

```text
기존 모델을 4비트로 로드
→ LoRA adapter만 학습
```

QAT는 최종 배포될 양자화 모델의 품질을 보존하는 방법이다.

```text
양자화 오차를 학습 중 시뮬레이션
→ 낮은 비트로 export해도 덜 망가지게 보정
```

둘 다 양자화를 사용하지만 해결하려는 문제가 다르다.

## 로컬 LLM에서의 의미

QAT의 실용적 가치는 **같은 메모리에서 한 단계 큰 모델을 실행할 가능성**이다. 다만 QAT라는 이름만으로 품질과 속도를 단정하지 않는다.

실제로 고를 때는 다음 순서가 안전하다.

1. 내 runtime이 해당 quant 포맷을 지원하는지 확인한다.
2. 모델과 context가 메모리에 들어오는지 확인한다.
3. BF16 또는 더 높은 bit quant와 짧은 기준 과제를 비교한다.
4. 구조화 출력과 tool calling까지 쓸 예정이면 별도 fixture로 검증한다.

QAT는 무손실 압축이 아니다. 4비트에서 발생할 손실을 학습으로 줄이는 방법이며, 최종 판단은 모델·변환기·runtime·작업을 함께 놓고 해야 한다.

## 참고 자료

- [Google - Quantization-aware training](https://ai.google.dev/gemma/docs/core)
- [Gemma QAT models](https://huggingface.co/models?search=google%20qat%20gguf)
- [PyTorch Quantization Aware Training](https://pytorch.org/blog/quantization-aware-training/)
