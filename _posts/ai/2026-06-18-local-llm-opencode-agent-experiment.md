---
layout: single
title: "16GB Mac에서 로컬 LLM을 OpenCode agent로 써본 결과 - 모델 크기보다 context가 먼저 막혔다"
date: 2026-06-18 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/06/18/local-llm-agent-experiment-hero.png
---

로컬 모델이 코드를 잘 생성하는지와 coding agent로 저장소를 안전하게 다루는지는 다른 문제다. OpenCode에 LM Studio provider를 고정하고 모델만 바꿔, 파일 탐색과 근거 해석이 필요한 같은 과제를 반복했다.

![두 로컬 모델이 파일 미로에서 탐색 편집 검증 과정을 비교하는 실험](/assets/images/2026/06/18/local-llm-agent-experiment-hero.png){: .align-center }

16GB Apple Silicon 환경에서 가장 크게 드러난 제약은 모델 파일 크기보다 **agent harness가 먼저 차지하는 context와 메모리**였다.

## 실험 과제

테스트 저장소에는 여러 문서와 템플릿이 있고, 특정 조건의 정답은 템플릿 한 파일에만 들어 있다. 모델은 다음 과정을 스스로 수행해야 한다.

1. 저장소 지침을 읽는다.
2. 정답이 있을 만한 폴더를 찾는다.
3. 템플릿 파일을 직접 읽는다.
4. 조건문의 숫자를 사람이 읽는 규칙으로 변환한다.
5. 근거를 빠짐없이 인용해 답한다.

정답 위치를 알려주는 질문과 힌트를 제거한 질문을 각각 사용했다. 이 차이가 agent의 독립적인 탐색 능력을 드러냈다.

## 환경

| 항목 | 값 |
| --- | --- |
| 하드웨어 | Apple Silicon, 16GB unified memory |
| Harness | OpenCode Build agent |
| Provider | LM Studio OpenAI-compatible server |
| Context | 32k |
| 평가 | 파일 도달, tool call, 조건 해석, 반복 재현 |

## 결과 요약

| 모델 | 크기·형식 | 결과 | 핵심 실패 |
| --- | --- | --- | --- |
| 작은 Gemma | 4B급 | 실패 | 지침을 읽고도 정답 파일로 이동하지 못함 |
| gpt-oss 20B | MXFP4 | 실행 불가 | 32k context와 harness를 함께 올리면 메모리 부족 |
| Qwen 9B | Q4_K_M | 부분 성공 | 첫 실행 정답, 재실행에서 항목 누락 |
| Nemotron 4B | Q4_K_M | 실패 | 루트 검색과 tool 반복, 정답 파일 미도달 |
| 14B 멀티모달 | Q4_K_M | 실행 불가 | projector와 context가 메모리 한도 초과 |
| Gemma 12B QAT | Q4_0 | 표준 질문 성공 | 힌트 제거 질문에서는 탐색 실패 |

## 20B와 14B는 context를 확보하지 못했다

20B 모델은 weight만으로도 메모리 대부분을 차지했다. 32k KV cache와 OpenCode, 운영체제를 함께 올리면 runtime의 안전 가드가 로딩을 막았다. 가드를 끄고 무리하게 실행했을 때는 시스템이 불안정해졌다.

14B 멀티모달 모델도 비슷했다. context를 줄이면 입력 자체가 들어가지 않고, 32k를 유지하면 추론 중 메모리가 부족했다.

```text
모델 weight
+ KV cache
+ agent system prompt
+ 도구 schema
+ 저장소 지침과 읽은 파일
+ 운영체제와 앱
= 실제 필요 메모리
```

"16GB에서 실행 가능한 모델"이라는 표시는 단발 채팅 기준일 수 있다. 두꺼운 agent harness를 함께 돌리는 조건은 훨씬 무겁다.

## 9B는 탐색했지만 결과가 재현되지 않았다

Qwen 9B 모델은 처음으로 검색–파일 읽기–조건 해석의 전체 흐름을 수행했다. 첫 실행에서는 정답에 도달했지만, 같은 요청을 다시 실행하자 항목 하나를 빠뜨렸다.

context 사용량은 두 실행에서 거의 같았다. 차이는 입력 부족보다 sampling과 작은 모델의 열거 정확도에 가까웠다.

이 실패는 눈에 잘 띄지 않는다. 파일을 찾지 못한 agent보다 **정답 파일을 읽고도 일부를 누락한 agent**가 더 그럴듯하게 보이기 때문이다. 기록과 코드를 수정하는 작업에서는 반드시 독립 검증이 필요하다.

## 4B는 tool을 쓰는 방식부터 흔들렸다

작은 모델 중 하나는 검색을 너무 일찍 멈췄고, 다른 모델은 반대로 tool을 반복 호출하면서도 범위를 좁히지 못했다.

- 저장소가 아닌 파일시스템 루트에서 검색
- 같은 검색어를 조금씩 바꾸어 반복
- 평가용 기록을 정답 근거로 착각
- 정답을 못 찾은 뒤 새 구조를 만들자고 역제안

tool call 횟수가 많다고 agent 성능이 좋은 것은 아니다. 검색 범위를 줄이고, 읽은 파일이 권위 있는 근거인지 판단하며, 실패했을 때 쓰기 작업으로 넘어가지 않는 능력이 더 중요하다.

## QAT 12B가 체급 상한을 올렸지만

Gemma 12B QAT Q4 모델은 9B 양자화 모델과 비슷한 크기로 로드됐고, 표준 질문에서는 정답을 한 번에 찾았다. QAT가 모델을 더 똑똑하게 만든 것이 아니라, 16GB에서 실행 가능한 체급을 9B에서 12B로 올려 더 큰 모델의 능력을 사용할 수 있게 한 사례다.

그러나 질문에서 템플릿 힌트를 제거하자 정답 폴더에 도달하지 못했다. 모델과 환경은 같았고 질문의 scaffolding만 바뀌었다.

```text
구체적인 질문
→ 정답 위치를 사실상 지목
→ 성공

자연스러운 질문
→ 검색 전략을 스스로 세워야 함
→ 실패
```

1회 성공만으로 주 agent 후보라고 결론 내릴 수 없는 이유다.

## 온디바이스 sLLM에는 context engineering이 중요하다

agent harness는 시작부터 많은 token을 사용한다.

- 저장소 지침
- skill 목록
- tool schema
- 대화 기록
- 탐색 중 읽은 파일

작은 모델은 이 context를 처리할 능력이 빠듯하고, 큰 모델은 메모리에 올리기 어렵다. 로컬 환경에서는 모델이 저장소 전체를 돌아다니게 하는 agentic search보다 외부 파이프라인이 근거를 먼저 좁혀 주는 방식이 현실적이다.

```text
검색·선별 파이프라인
→ 짧고 정확한 근거 context
→ 로컬 모델의 요약·추출·판단
```

로컬 모델에 모든 탐색과 편집 권한을 주기보다, 검색과 검증을 결정적인 코드가 맡고 모델은 제한된 판단만 수행하게 한다.

## 실사용 기준

- 실행 가능 여부와 agent 성공 여부를 분리한다.
- 같은 과제를 여러 번 돌려 누락과 비결정성을 확인한다.
- 최종 답변이 아니라 실제 파일, diff, 테스트를 채점한다.
- 작은 모델에는 짧고 선별된 context를 제공한다.
- 파일 삭제와 경로 변경은 사람이 검토하기 전까지 허용하지 않는다.

16GB 환경에서도 로컬 coding agent는 가능성을 보였다. 다만 현재의 병목은 모델을 한 번 띄우는 일이 아니라, 긴 context 안에서 올바른 파일을 찾고 결과를 재현하는 일이다.

## 참고 자료

- [OpenCode](https://github.com/anomalyco/opencode)
- [LM Studio Docs](https://lmstudio.ai/docs)
- [Ollama, LM Studio, llama.cpp 비교 글](/ai/local-llm-provider-comparison/)
- [QAT 양자화 글](/ai/quantization-aware-training-local-llm/)
