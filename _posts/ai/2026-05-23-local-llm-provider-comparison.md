---
layout: single
title: "Ollama vs LM Studio vs llama.cpp - 로컬 LLM 실행 도구를 고르는 기준"
date: 2026-05-23 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/05/23/local-llm-provider-comparison-hero.png
---

Ollama, LM Studio, `llama.cpp`는 모두 로컬 모델을 실행하고 OpenAI 호환 API로 노출할 수 있다. 같은 GGUF 계열 모델을 띄울 수 있어도 사용 목적과 제어 범위는 꽤 다르다.

![하나의 로컬 모델을 서버와 GUI와 CLI 방식으로 비교하는 구성](/assets/images/2026/05/23/local-llm-provider-comparison-hero.png){: .align-center }

세 도구를 OpenCode의 로컬 provider로 연결해 보면서 설치 편의, 실행 옵션, context 부담, tool calling 흐름을 비교했다. 용도는 선명하게 갈렸다. **모델을 쉽게 쓰려면 Ollama, GUI로 관리하려면 LM Studio, 실행 조건을 고정하려면 `llama.cpp`**가 알맞았다.

## 세 도구의 위치

| 항목 | Ollama | LM Studio | llama.cpp |
| --- | --- | --- | --- |
| 주 사용면 | CLI와 백그라운드 서버 | 데스크톱 GUI | CLI와 직접 실행 서버 |
| 모델 관리 | 이름과 태그 중심 | 검색·다운로드·선택을 GUI에서 | GGUF repo와 파일을 직접 지정 |
| API | Ollama API + OpenAI 호환 | OpenAI 호환 | OpenAI 호환 |
| 옵션 제어 | 단순함 | GUI에서 주요 옵션 제공 | context·sampling·GPU offload 등을 세밀하게 지정 |
| 시작 난이도 | 낮음 | 가장 낮음 | 상대적으로 높음 |
| 재현 가능한 비교 | 보통 | 보통 | 가장 유리 |

### Ollama

```bash
ollama run qwen2.5-coder:7b
```

모델 이름만 알면 다운로드와 실행이 한 번에 끝난다. 터미널 채팅, 자동 실행, 백그라운드 API 서버가 필요한 경우 편하다.

### LM Studio

모델 검색과 다운로드, 메모리 예상치 확인, 서버 실행을 GUI에서 처리한다. GGUF 이름과 quantization 표기가 낯설 때 가장 접근하기 쉽다.

### llama.cpp

```bash
llama-server \
  -hf ggml-org/gemma-4-E4B-it-GGUF:Q4_K_M \
  -c 32768 \
  --host 127.0.0.1 \
  --port 8080
```

어떤 파일과 quant를 쓰는지, context를 얼마로 잡는지 명령에 그대로 남는다. 모델 비교와 성능 재현에는 이 방식이 가장 명확하다.

## OpenCode에 연결하는 주소

세 도구 모두 OpenAI 호환 provider로 등록할 수 있다.

| Provider | 기본 base URL |
| --- | --- |
| Ollama | `http://127.0.0.1:11434/v1` |
| LM Studio | `http://127.0.0.1:1234/v1` |
| llama.cpp | `http://127.0.0.1:8080/v1` |

연결이 됐는지는 모델 목록 endpoint로 먼저 확인한다.

```bash
curl http://127.0.0.1:11434/v1/models
curl http://127.0.0.1:1234/v1/models
curl http://127.0.0.1:8080/v1/models
```

## 같은 모델인데 결과가 달라지는 이유

provider는 단순한 파일 로더가 아니다. 모델 앞뒤에 붙는 chat template, tool-call 변환, context 처리, sampling 기본값이 모두 실행 결과에 영향을 준다.

```text
사용자 요청
→ agent harness의 system prompt와 도구 schema
→ provider adapter
→ runtime의 chat template와 sampling
→ 로컬 모델
→ tool call 또는 텍스트 응답
```

따라서 같은 모델 계열과 같은 32k context를 지정해도 tool call 진입 시점과 메모리 사용감이 달라질 수 있다.

## 구조화된 파일 탐색 과제로 비교해 본 결과

비교 과제는 저장소 지침을 읽고, 특정 템플릿 파일을 찾아 조건문을 해석하는 작업이었다. 단순 지식 질문과 달리 검색 경로 선택, 파일 읽기, 근거 계산이 연속으로 필요하다.

| Provider | 관찰 | 판단 |
| --- | --- | --- |
| llama.cpp | 실제 파일 검색과 읽기까지 가장 적극적으로 진입했지만 검색 위치를 잘못 골랐다 | tool use 가능성은 보였으나 정답 검증 실패 |
| Ollama | 지침 하나를 읽은 뒤 후속 파일 탐색이 일찍 멈췄다 | 연결은 쉬웠지만 이번 agent loop는 약함 |
| LM Studio | 초기에는 탐색이 멈췄고, 위치 힌트를 준 뒤 파일을 읽었지만 조건 해석을 틀렸다 | 사용성은 좋지만 파일 작업 신뢰성은 별개 |

이 결과로 특정 provider의 우열을 일반화할 수는 없다. 모델, quant, runtime 버전, harness 설정을 한꺼번에 통제하지 못한 소규모 실험이기 때문이다. 분명했던 것은 **도구 호출을 시작했다는 사실만으로 파일 작업이 안전해지지는 않는다**는 점이다.

## context가 agent 성능의 병목이 된다

로컬 채팅은 4k context로도 충분할 수 있지만, coding agent는 시작할 때부터 지침, 스킬 목록, 도구 schema를 넣는다.

```text
request (...) exceeds the available context size (4096 tokens)
```

이런 오류가 나면 모델 답변을 평가하기도 전에 입력이 잘린다. OpenCode 같은 harness에 붙일 때는 32k 이상을 시작점으로 두되, KV cache가 늘어나는 만큼 메모리 압박도 같이 확인해야 한다.

작은 모델에서는 prompt가 20k token만 되어도 전체 context의 절반 이상을 이미 사용한다. 모델이 스스로 탐색하면서 파일 내용을 더 읽으면 여유가 빠르게 사라진다.

## 선택 기준

| 하고 싶은 일 | 먼저 고를 도구 |
| --- | --- |
| 명령 한 줄로 모델을 받고 대화 | Ollama |
| 여러 모델을 GUI에서 탐색·교체 | LM Studio |
| GGUF와 quant, context를 정확히 고정 | llama.cpp |
| 반복 가능한 provider 성능 비교 | llama.cpp |
| 비개발자에게 로컬 서버 제공 | LM Studio |
| 셸 스크립트와 백그라운드 서비스 자동화 | Ollama |

provider는 로컬 모델을 쓰기 위한 실행면이다. 복잡한 저장소 작업에서 결과를 결정하는 쪽은 모델의 추론 능력, context 여유, tool-call 학습, harness와의 궁합에 더 가깝다.

## 참고 자료

- [Ollama](https://github.com/ollama/ollama)
- [LM Studio Docs](https://lmstudio.ai/docs)
- [llama.cpp](https://github.com/ggml-org/llama.cpp)
- [OpenCode providers](https://opencode.ai/docs/providers/)
