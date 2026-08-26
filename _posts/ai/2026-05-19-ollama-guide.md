---
layout: single
title: "Ollama 사용법 - 로컬 LLM 설치, 실행, API 서버 관리까지"
date: 2026-05-19 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/05/19/ollama-guide-hero.png
---

Ollama는 로컬 LLM을 내려받고 실행하며, 다른 앱이 사용할 수 있는 API 서버까지 함께 제공한다. 모델 파일과 실행 옵션을 직접 다루는 `llama.cpp`보다 시작이 쉽고, GUI 중심인 LM Studio보다 터미널 자동화에 잘 맞는다.

![로컬 모델 카트리지를 서버에 연결해 토큰을 생성하는 Ollama 흐름](/assets/images/2026/05/19/ollama-guide-hero.png){: .align-center }

이 글에서는 설치부터 모델 관리, 로컬 API 확인까지 Ollama 자체 사용법에 집중한다. Codex App과 OpenCode 연결 결과는 별도 글에서 다룬다.

## 설치와 첫 실행

macOS에서는 Homebrew로 설치할 수 있다.

```bash
brew install ollama
```

모델을 내려받으면서 바로 대화하려면 `ollama run`을 사용한다.

```bash
ollama run qwen2.5-coder:7b
```

한 번만 질문하고 끝낼 수도 있다.

```bash
ollama run qwen2.5-coder:7b \
  "Python으로 add 함수를 작성해줘. 코드만 출력해."
```

대화형 세션은 `/bye`, `exit`, `Ctrl+D` 중 하나로 끝낸다.

## API 서버로 사용하기

Ollama는 기본적으로 `127.0.0.1:11434`에서 API를 제공한다.

```bash
ollama serve
```

서버가 준비됐는지는 모델 목록 API로 확인한다.

```bash
curl http://127.0.0.1:11434/api/tags
```

macOS에서 계속 백그라운드로 띄우려면 Homebrew service를 사용할 수 있다.

```bash
brew services start ollama
brew services stop ollama
```

이미 서비스가 떠 있는데 `ollama serve`를 다시 실행하면 `address already in use`가 발생한다. 새 서버를 띄우기 전에 프로세스와 포트를 확인한다.

```bash
ollama ps
lsof -nP -iTCP:11434 -sTCP:LISTEN
```

## 모델 관리 명령

### 다운로드와 목록

```bash
ollama pull qwen2.5-coder:7b
ollama list
```

### 모델 정보 확인

```bash
ollama show qwen2.5-coder:7b
ollama show qwen2.5-coder:7b --modelfile
```

로컬 모델을 agent에 연결할 때는 파라미터 수만 보지 않는다.

- context length
- quantization
- `tools` capability
- chat template과 tool-call 형식
- 실제 메모리 사용량

모델이 tool calling을 지원한다고 표시되어도 특정 agent harness에서 안정적으로 파일을 찾고 수정한다는 보장은 없다.

### 메모리와 디스크에서 내리기

```bash
ollama ps
ollama stop qwen2.5-coder:7b
ollama rm qwen2.5-coder:7b
```

`stop`은 실행 중인 runner만 메모리에서 내리고 모델 파일은 남긴다. `rm`은 디스크의 모델을 삭제한다.

## OpenAI 호환 API로 연결하기

Ollama는 OpenAI 호환 endpoint도 제공한다. 이를 지원하는 클라이언트에서는 base URL을 다음처럼 지정한다.

```text
http://127.0.0.1:11434/v1
```

OpenCode 설정 예시는 다음과 같다.

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama local",
      "options": {
        "baseURL": "http://127.0.0.1:11434/v1"
      },
      "models": {
        "qwen2.5-coder:7b": {
          "name": "Qwen2.5 Coder 7B",
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

등록 결과는 다음 명령으로 확인한다.

```bash
opencode models | rg ollama
```

## 어떤 모델 크기가 현실적인가

모델이 메모리에 올라가는 것과 쾌적하게 동작하는 것은 다르다. weight뿐 아니라 KV cache, runtime buffer, agent의 긴 context도 메모리를 사용한다.

| 환경 | 시작점 |
| --- | --- |
| 16GB 통합 메모리 | 7B~14B 양자화 모델 |
| 단순 채팅·요약 | 작은 instruct 모델 |
| 코딩 대화 | coder 계열 7B 이상 |
| 긴 도구 호출 agent | 모델 크기보다 context와 tool-call 안정성을 먼저 검증 |

20B 모델이 실행되더라도 swap이 크게 늘면 토큰 생성 속도가 무너진다. 느려졌을 때는 `ollama ps`와 운영체제의 memory pressure부터 확인한다.

## Ollama가 잘 맞는 작업

- 모델을 빠르게 받아 로컬에서 대화하기
- 인터넷 없이 짧은 코드와 문서를 다루기
- Open WebUI나 개발 도구의 로컬 백엔드로 사용하기
- quantization과 모델 크기별 메모리 사용량 비교하기
- 외부 API에 보내고 싶지 않은 짧은 텍스트 처리하기

반대로 큰 저장소를 탐색하고 여러 도구를 연속 호출하는 coding agent는 별도 검증이 필요하다. 로컬 실행 여부와 agent 신뢰성은 서로 다른 문제다.

## 문제별 빠른 확인

```bash
# 너무 느릴 때
ollama ps

# 모델을 메모리에서 내릴 때
ollama stop MODEL

# API 서버 확인
curl http://127.0.0.1:11434/api/tags

# 모델 디스크 사용량 확인
du -sh ~/.ollama/models
```

Ollama의 장점은 복잡한 런타임 설정을 모델 이름과 몇 개의 명령으로 감싸준다는 점이다. 세밀한 GGUF 옵션이 필요해질 때는 `llama.cpp`, GUI로 모델을 비교하고 싶다면 LM Studio가 다음 선택지다.

## 참고 자료

- [Ollama](https://github.com/ollama/ollama)
- [Ollama API](https://docs.ollama.com/api)
- [Ollama OpenAI compatibility](https://docs.ollama.com/openai)
