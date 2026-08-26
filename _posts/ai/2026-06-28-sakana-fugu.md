---
layout: single
title: "Sakana Fugu - 여러 프론티어 모델을 조율하는 learned orchestration"
date: 2026-06-28 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/06/28/fugu-architecture.webp
---

Sakana Fugu는 일본산 단일 프론티어 LLM이 아니다. 하나의 모델처럼 호출하지만 내부에서는 여러 모델에 역할을 나누고 결과를 검증·합성하는 learned orchestration system이다.

여러 외부 모델을 호출한다는 점 때문에 "결국 라우터 아닌가"라는 비판을 받았다. 다만 고정 규칙 라우터와 달리 어떤 모델을 어떤 구조로 엮을지 학습한다는 점은 구분할 필요가 있다.

## Fugu의 정체

| 항목 | 내용 |
| --- | --- |
| 제품 형태 | Multi-Agent System as a Model |
| 사용 방식 | OpenAI 호환 API |
| 내부 역할 | 요청 분해, expert 선택, topology 구성, 검증과 합성 |
| 모델군 | Fugu, Fugu Ultra |
| 기반 연구 | TRINITY, Conductor, Fugu Technical Report |

Fugu는 직접 답할 수 있는 요청은 처리하고, 복잡한 요청은 agent pool에 위임한다. pool에는 외부 모델뿐 아니라 Fugu 자신도 들어갈 수 있어 재귀적인 test-time scaling이 가능하다.

![Sakana Fugu architecture](/assets/images/2026/06/28/fugu-architecture.webp)

## 고정 라우터와 무엇이 다른가

일반적인 router는 사람이 정한 규칙이나 benchmark 점수로 모델을 고른다.

```text
코딩 질문 → coding model
긴 문서 → long-context model
싼 요청 → small model
```

Fugu가 표방하는 흐름은 동적이다.

1. 요청을 직접 풀지 위임할지 판단한다.
2. Thinker, Worker, Verifier 같은 역할을 배정한다.
3. 병렬, 직렬, 토론, 계층형 중 작업 topology를 구성한다.
4. 중간 결과를 보고 critic이나 추가 expert를 붙인다.
5. 검증 결과를 하나의 답으로 합친다.

| topology | 예시 |
| --- | --- |
| 직렬 | 분석 → 구현 → 검증 |
| 병렬 | 여러 모델이 답을 제안한 뒤 비교 |
| 토론 | 제안과 반박을 여러 차례 반복 |
| 계층 | 하위 작업을 나눠 수행하고 상위 모델이 취합 |

이 topology와 위임 prompt를 매 요청에 맞춰 고르는 것이 제품의 핵심이다.

## TRINITY와 Conductor

Fugu는 역할 배정과 소통 구조를 학습하는 두 연구 흐름 위에 있다.

| 연구 | 학습 대상 | 접근 |
| --- | --- | --- |
| TRINITY | 누구에게 어떤 역할을 맡길지 | 진화전략 기반 coordinator |
| Conductor | agent를 어떤 구조로 연결하고 무엇을 요청할지 | 강화학습 기반 자연어 orchestration |

공통점은 거대한 단일 모델을 새로 학습하는 대신 작은 coordinator가 기존 모델의 강점을 조합한다는 것이다. Fugu가 이 두 연구를 제품 안에서 정확히 어떻게 결합했는지는 전부 공개되지 않았다.

## 공식 성능은 어떻게 읽어야 하나

Sakana AI가 공개한 결과에서 Fugu Ultra는 여러 coding·terminal benchmark에서 강한 점수를 기록했다.

![Sakana Fugu benchmark](/assets/images/2026/06/28/fugu-benchmark.webp)

수치만 볼 때는 세 조건을 함께 확인해야 한다.

- 비교 모델에도 같은 agent scaffold가 적용됐는가
- 한 문제에 사용한 총 token과 model call은 얼마인가
- provider가 보고한 수치가 독립적으로 재현됐는가

multi-agent system은 단일 모델 호출보다 test-time compute를 더 쓸 수 있다. 성능이 올랐다는 주장과 같은 비용으로 올랐다는 주장은 다르다.

## 비용은 단가보다 총 호출량에서 커진다

오케스트레이션 API가 단일 token rate를 제시해도 내부에서 여러 expert가 답하고 검증하면 전체 token 사용량이 늘어난다.

```text
사용자 입력
→ coordinator
→ expert A + expert B + verifier
→ 합성 답변
```

같은 입력을 단일 모델 한 번에 보낼 때보다 처리 token이 많아질 수 있다. 따라서 가격표의 1M token 단가뿐 아니라 요청 한 건의 실제 usage와 latency를 봐야 한다.

잘 맞는 위치는 모든 코딩 요청의 기본 모델보다 다음과 같은 마지막 검수 단계에 가깝다.

- 큰 코드 변경의 독립 리뷰
- 보안·ML처럼 실패 비용이 큰 검증
- 긴 문헌 조사와 논문 재현
- 단일 모델이 반복해서 막힌 난도 높은 문제

## "라우터일 뿐"이라는 비판

비판은 세 갈래다.

| 비판 | 확인해야 할 반론과 한계 |
| --- | --- |
| 외부 모델 성능을 가져다 쓴다 | orchestration도 실제 성능을 좌우하지만 agent pool 공개 범위가 중요 |
| 호출량과 지연이 커진다 | 어려운 작업에서만 선택적으로 쓰는 전략이 필요 |
| 독립 재현이 부족하다 | 동일 scaffold와 token budget으로 비교해야 함 |

Fugu를 자체 foundation model로 홍보하면 기대와 실체의 간극이 커진다. 반대로 단순 설정 라우터라고만 부르면 역할 배정과 topology를 학습하는 부분을 놓친다.

## 모델 경쟁에서 orchestration 경쟁으로

Fugu가 보여주는 흐름은 단일 benchmark 1위보다 더 흥미롭다.

```text
더 큰 단일 모델
        ↓
test-time compute
multi-agent + verifier
context와 routing 최적화
```

프론티어 모델의 성능 차이가 줄수록 어떤 모델에 무엇을 맡기고, 결과를 어떻게 검증하는지가 제품 차별점이 된다. Fugu는 그 routing과 검증을 별도 모델이 학습하는 상용 사례다.

현재로서는 강한 공식 수치와 제한적인 독립 검증이 함께 존재한다. 새 프론티어 LLM이라기보다 **여러 프론티어 모델을 하나의 실행 시스템으로 묶는 모델**로 보는 편이 정확하다.

## 참고 자료

- [Sakana Fugu](https://sakana.ai/fugu/)
- [Fugu Technical Report](https://arxiv.org/abs/2606.21228)
- [TRINITY: An Evolved LLM Coordinator](https://arxiv.org/abs/2512.04695)
- [Learning to Orchestrate Agents in Natural Language with the Conductor](https://arxiv.org/abs/2512.04388)
- [Sakana AI](https://sakana.ai/)
