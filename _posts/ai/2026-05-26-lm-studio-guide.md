---
layout: single
title: "LM Studio 사용법 - 로컬 LLM을 GUI와 OpenAI 호환 API로 실행하기"
date: 2026-05-26 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/05/26/lm-studio-guide-hero.png
---

LM Studio는 로컬 LLM을 검색하고 내려받아 실행하는 데스크톱 앱이다. 모델 파일과 quantization을 GUI에서 고를 수 있고, 내려받은 모델을 OpenAI 호환 API 서버로 띄울 수도 있다.

![데스크톱 화면에서 로컬 모델을 고르고 API 클라이언트에 연결하는 흐름](/assets/images/2026/05/26/lm-studio-guide-hero.png){: .align-center }

Ollama보다 화면 중심이고 `llama.cpp`보다 실행 옵션을 직접 다룰 일이 적다. 로컬 LLM을 처음 비교하거나 여러 GGUF를 바꿔가며 테스트할 때 편하다.

## 설치

macOS에서는 Homebrew cask로 설치할 수 있다.

```bash
brew install --cask lm-studio
open -a "LM Studio"
```

앱의 모델 검색 화면에서 Hugging Face 모델을 찾고 quantization을 선택한다. 이름이 비슷해도 파일 크기와 요구 메모리가 다르므로 `Q4_K_M`, `Q5_K_M`, `Q8_0` 같은 suffix를 함께 확인한다.

## 모델을 고를 때 볼 것

| 항목 | 확인 이유 |
| --- | --- |
| 파라미터 수 | 모델의 대략적인 체급 |
| quantization | 파일 크기와 품질 손실의 균형 |
| context length | 한 번에 넣을 수 있는 입력 길이 |
| chat template | 대화·tool call 포맷 호환성 |
| 예상 메모리 | 내 하드웨어에서 실제 로드 가능한지 |

파일이 메모리에 들어간다고 해서 긴 context에서도 여유가 있다는 뜻은 아니다. KV cache와 runtime buffer까지 고려한다.

## Local Server 실행

`Developer` 또는 `Local Server` 화면에서 모델을 선택하고 서버를 시작한다. 기본 주소는 다음과 같다.

```text
http://127.0.0.1:1234/v1
```

모델 목록을 호출해 서버 상태를 확인한다.

```bash
curl http://127.0.0.1:1234/v1/models
```

응답에 로드한 모델이 나오면 외부 클라이언트가 연결할 수 있다.

### 간단한 Chat Completions 요청

```bash
curl http://127.0.0.1:1234/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "MODEL_ID",
    "messages": [
      {"role": "user", "content": "한국어로 한 문장만 답해줘."}
    ],
    "temperature": 0.7
  }'
```

`MODEL_ID`에는 `/v1/models`에서 확인한 ID를 넣는다.

## OpenCode 연결

OpenCode 설정에 OpenAI 호환 provider로 등록한다.

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio local",
      "options": {
        "baseURL": "http://127.0.0.1:1234/v1",
        "apiKey": "lm-studio"
      },
      "models": {
        "MODEL_ID": {
          "name": "Local model",
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

서버가 인증을 요구하지 않아도 클라이언트 설정 schema 때문에 API key가 필요한 경우가 있다. 이때 실제 비밀값 대신 `lm-studio` 같은 더미 문자열을 넣는다.

```bash
opencode models | rg lmstudio
```

목록에 모델이 나타나면 연결은 끝난다.

## 편한 runtime과 좋은 agent는 다르다

LM Studio는 모델 다운로드와 서버 실행 경험이 가장 편한 축에 속한다. 그러나 작은 로컬 모델을 coding agent에 연결했을 때 파일 탐색과 tool call이 안정적이 되는 것은 아니다.

다음 두 검증을 분리해야 한다.

1. API 연결이 되고 텍스트가 생성되는가
2. 지침을 읽고 올바른 도구와 파일을 골라 작업하는가

첫 번째는 `/v1/models`와 짧은 chat 요청으로 확인할 수 있다. 두 번째는 격리된 fixture에서 정답 파일, diff, 테스트 결과까지 별도로 검증해야 한다.

## 언제 LM Studio가 좋은가

- GGUF 모델을 검색하고 크기를 비교할 때
- 여러 quantization을 빠르게 바꿔 볼 때
- 로컬 채팅과 API 서버를 한 앱에서 관리할 때
- OpenAI 호환 클라이언트를 GUI 기반 로컬 서버에 붙일 때

터미널 자동화가 중심이면 Ollama가 단순하고, 실행 조건을 완전히 고정해야 하는 벤치마크라면 `llama.cpp`가 더 직접적이다.

## 참고 자료

- [LM Studio Docs](https://lmstudio.ai/docs)
- [LM Studio OpenAI Compatibility API](https://lmstudio.ai/docs/developer/openai-compat)
