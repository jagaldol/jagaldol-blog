---
layout: single
title: "llama.cpp 사용법 - GGUF 모델을 CLI와 OpenAI 호환 서버로 실행하기"
date: 2026-05-30 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/05/30/llama-cpp-guide-hero.png
---

`llama.cpp`는 GGUF 모델을 직접 실행하는 로컬 추론 런타임이다. Ollama처럼 모델 관리를 감싸기보다 모델 파일, quantization, context, sampling, 서버 포트를 명령으로 직접 제어한다.

![모델 파일을 추론 엔진에 넣어 토큰을 빠르게 생성하는 llama.cpp 구조](/assets/images/2026/05/30/llama-cpp-guide-hero.png){: .align-center }

대화형 실행은 `llama-cli`, 다른 앱이 붙을 API 서버는 `llama-server`를 사용한다.

## 설치와 장치 확인

macOS에서는 Homebrew로 설치할 수 있다.

```bash
brew install llama.cpp
```

버전과 사용 가능한 장치를 확인한다.

```bash
llama-cli --version
llama-cli --list-devices
```

Apple Silicon에서는 Metal 장치가 보이는지 확인한다. 모델이 CPU로만 실행되면 속도가 크게 달라질 수 있다.

## Hugging Face에서 GGUF 바로 실행하기

`-hf` 옵션에 Hugging Face repository와 quant를 지정하면 모델을 내려받아 캐시에 저장하고 실행한다.

```bash
llama-cli \
  -hf ggml-org/gemma-4-E4B-it-GGUF:Q4_K_M \
  -p "한국어로 한 문장만 답해줘." \
  -n 64 \
  --single-turn \
  --no-display-prompt
```

캐시 목록은 다음 명령으로 본다.

```bash
llama-cli --cache-list
```

같은 repository에 GGUF가 여러 개라면 파일명을 직접 고정할 수 있다.

```bash
llama-cli \
  -hf ggml-org/gemma-4-E4B-it-GGUF \
  -hff gemma-4-E4B-it-Q4_K_M.gguf
```

성능을 비교할 때는 repository만 넘겨 자동 선택에 맡기지 말고 quant 파일까지 명시한다.

## 대화형 CLI

```bash
llama-cli \
  -hf ggml-org/gemma-4-E4B-it-GGUF:Q4_K_M \
  -cnv -mli -c 4096 \
  --temp 1 --top-k 64 --top-p 0.95
```

| 옵션 | 의미 |
| --- | --- |
| `-cnv` | 여러 턴을 잇는 conversation mode |
| `-mli` | 여러 줄 입력 mode |
| `-c 4096` | context window 크기 |
| `--temp`, `--top-k`, `--top-p` | sampling 설정 |

`-mli`에서는 Enter가 줄바꿈이고 `Ctrl+D`가 현재 입력 전송이다. 종료는 `Ctrl+C`를 사용한다.

reasoning 출력을 지원하는 template은 다음 옵션으로 조절할 수 있다.

```bash
-rea off
-rea on
-rea auto
```

모델과 `llama.cpp` 버전에 따라 지원 여부가 다르므로 `llama-cli --help`도 함께 확인한다.

## OpenAI 호환 서버

```bash
llama-server \
  -hf ggml-org/gemma-4-E4B-it-GGUF:Q4_K_M \
  -c 32768 \
  -rea off \
  --host 127.0.0.1 \
  --port 8080
```

서버 상태와 모델 ID를 확인한다.

```bash
curl http://127.0.0.1:8080/v1/models
```

브라우저에서 `http://127.0.0.1:8080`을 열면 기본 Web UI도 사용할 수 있다.

### context는 클라이언트 설정과 맞춘다

coding agent는 사용자 질문 외에도 system prompt, 도구 schema, 저장소 지침을 함께 보낸다. 서버를 4k context로 띄우면 요청이 시작부터 잘릴 수 있다.

```text
request (...) exceeds the available context size (4096 tokens)
```

클라이언트 설정에 32k를 적었더라도 `llama-server`가 실제로 4k로 떠 있으면 소용없다. 서버와 클라이언트 양쪽 context 한도를 맞춘다.

## OpenCode provider 등록

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "llamacpp": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama.cpp local",
      "options": {
        "baseURL": "http://127.0.0.1:8080/v1"
      },
      "models": {
        "MODEL_ID": {
          "name": "Local GGUF",
          "limit": {
            "context": 32768,
            "output": 4096
          }
        }
      }
    }
  }
}
```

```bash
opencode models | rg llamacpp
```

API 연결과 tool calling은 별도 문제다. 모델의 chat template이 도구 호출을 지원하는지, agent가 올바른 파일을 읽고 결과를 검증하는지 따로 시험한다.

## Ollama와 역할 차이

| 항목 | llama.cpp | Ollama |
| --- | --- | --- |
| 사용감 | CLI와 API를 직접 제어 | 모델 관리와 채팅을 감쌈 |
| 모델 지정 | GGUF repository·파일·quant | 모델 이름과 태그 |
| context | `-c`로 직접 지정 | 모델과 runtime 설정으로 관리 |
| 비교 실험 | 조건 고정이 쉬움 | 빠른 설치와 일반 사용이 쉬움 |

Ollama의 내부 blob을 찾아 `llama-cli -m`으로 직접 여는 방식은 권장하지 않는다. manifest와 멀티모달 projector, template이 함께 필요한 모델은 blob 하나만으로 재현되지 않는다. 같은 계열을 비교하려면 원본 GGUF repository에서 파일을 따로 받는 편이 명확하다.

## 자주 쓰는 명령

```bash
# 버전과 장치
llama-cli --version
llama-cli --list-devices

# 캐시
llama-cli --cache-list

# 서버 확인
curl http://127.0.0.1:8080/v1/models

# 포트 확인
lsof -nP -iTCP:8080 -sTCP:LISTEN
```

`llama.cpp`는 처음에는 옵션이 많아 보이지만, 실행 조건이 명령에 남는다는 점이 강점이다. GGUF와 quantization을 직접 비교하거나 로컬 추론 실험을 재현할 때 가장 투명한 도구다.

## 참고 자료

- [llama.cpp](https://github.com/ggml-org/llama.cpp)
- [llama.cpp server README](https://github.com/ggml-org/llama.cpp/tree/master/tools/server)
