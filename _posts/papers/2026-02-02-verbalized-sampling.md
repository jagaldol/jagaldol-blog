---
layout: single
title: "[Review] Verbalized Sampling: How to Mitigate Mode Collapse and Unlock LLM Diversity"
date: 2026-02-02 16:51:00 +0900
categories: papers
---

똑같은 말만 하던 LLM에게 **다양한 대답**을 얻는 방법이 있다고한다. 훈련 없이, **오직 프롬프트만으로** 다양성을 끌어올리는 접근이다.

데이터 합성, 아이디어 브레인스토밍처럼 **다양한 샘플이 필요한 상황**에서 특히 더 유용하다.

{% linkpreview "https://arxiv.org/abs/2510.01171" %}

> 논문 리뷰 - Weekly Tech Trend Talk 스터디(25.01.15)

## 0. 문제 상황: Mode Collapse

생성 모델이 데이터의 **다양한 패턴(모드)을 배우지 못하고**(특히 RLHF로 특정 패턴에 편향이 생기며),
**특정 패턴만 반복적으로 생성**해버리는 문제다.

그 결과, “정답처럼 보이는 한 가지 표현”으로 수렴해 다양성이 급격히 줄어든다.

---

## 1. 해결법: Verbalized Sampling Prompt(VS)

![verbalized-sampling](/assets/images/2026/02/02/verbalized-sampling.png)

그림 왼쪽은 **typicality bias**로 인해 하나의 농담으로 수렴하는 모습이다.
오른쪽은 “여러 개의 응답과 확률을 함께 말하라”는 **verbalized sampling**을 통해
서로 다른 농담들이 한 번에 샘플링되는 과정을 보여준다.

### 1.1. Simple Prompt

```
You are a  helpful assistant.
For each query, please generate a set of five possible responses, each within a separate <response> tag.
Responses should each include a <text> and a numeric <probability>.
Please sample at random from the [full distribution / tails of the distribution, such that the probability of each response is less than 0.10].
```

핵심은 **모델이 확률까지 “말로” 표현**하게 만들어 분포의 꼬리까지 탐색하도록 유도하는 것이다.

### 1.2. 왜 효과적인가?

- **직접 샘플링**은 가장 그럴듯한 한 문장에 쏠리기 쉽다.
- VS는 **“다섯 개를 뽑아 확률까지 붙이라”**는 제약이 있어
  모델이 자연스럽게 **다른 모드들을 찾아** 분산된 응답을 내놓게 만든다.

---

## 2. 결과

### 2.1. 정량 평가

![quantitative-results](/assets/images/2026/02/02/quantitative-results.png)

<div class="notice--info" markdown="1">

💡 **g~i에 사용되는 프롬프트**

Generate five responses with probabilities below {threshold}

</div>

요약하면, **VS 계열 프롬프트가 다양성 점수를 가장 크게 끌어올린다**.
또한 일부 설정에서는 **품질 저하 없이 다양성이 증가**하는 영역(Pareto optimal)이 존재한다.

### 2.2. 정성 평가

![qualitative-results](/assets/images/2026/02/02/qualitative-results.png){: .width-300px}

사람이 평가한 결과에서도 VS가 가장 높은 다양성 점수를 기록한다.

### 2.3. Direct vs. Verbalized Sampling

![direct-vs-verbalized](/assets/images/2026/02/02/direct-vs-verbalized.png)

스토리 작성, 대화 시뮬레이션, 열린 질문 등 **다양한 태스크에서 VS가 더 폭넓은 분포**를 만든다.
즉, 단순히 “다섯 개를 뽑아라”가 아니라 **“확률까지 말하게 하는 것”이 핵심**임을 보여준다.

---

## 정리

Verbalized Sampling은 **학습 없이도** LLM의 다양성을 크게 높이는 간단한 프롬프트 기법이다.
다양성이 중요한 작업이라면, **VS 프롬프트를 기본 옵션으로 고려**할 만하다.

## A. Verbalized Sampling을 제작한 GPTs 공유

[ChatGPT - Verbalized Sampling](https://chatgpt.com/g/g-69686f3d6eb4819199c9d33938bd5fdc-verbalized-sampling)

잘 만든 GPTs가 존재해서 공유한다. 직접 사용해보고 Verbalized Sampling의 효과를 체감해보자!
