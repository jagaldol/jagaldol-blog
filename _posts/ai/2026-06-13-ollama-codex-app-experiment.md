---
layout: single
title: "Ollama 로컬 모델을 Codex App에 연결해 본 결과 - 실행 가능과 agent 가능은 다르다"
date: 2026-06-13 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/06/13/ollama-codex-experiment-hero.png
---

Ollama는 로컬 모델을 Codex App의 provider로 연결하는 명령을 제공한다. 앱 화면은 그대로 사용하면서 실제 응답은 로컬 모델이 만든다.

![로컬 서버와 코드 편집기를 파일 편집 및 테스트 단계에 연결한 실험 흐름](/assets/images/2026/06/13/ollama-codex-experiment-hero.png){: .align-center }

연결 자체는 간단했지만 coding agent로서의 결과는 모델마다 크게 달랐다. 모델이 텍스트를 생성할 수 있다는 것과 Codex의 도구 호출 규약을 따라 파일 작업을 이어갈 수 있다는 것은 별개의 능력이었다.

## 연결과 복구

```bash
ollama launch codex-app --model qwen2.5-coder:7b
```

실험을 끝내고 원래 provider로 돌아갈 때는 다음 명령을 사용한다.

```bash
ollama launch codex-app --restore
```

이 명령은 Codex 설정의 model provider와 base URL을 바꾼다. 읽기 전용 테스트가 아니므로 실험 전후에 현재 provider를 확인하고, 끝나면 `--restore`까지 실행한다.

## tool calling이 실제로 성립하려면

모델이 JSON처럼 보이는 문장을 출력한다고 도구가 호출된 것은 아니다.

```text
Codex App이 도구 schema를 모델에 전달
→ 모델이 tool call 선택
→ runtime이 구조화된 tool event로 변환
→ Codex App이 실제 도구 실행
→ 실행 결과가 모델에 다시 전달
→ 다음 응답 계속
```

중간 변환이 깨지면 다음처럼 tool call 모양의 JSON이 일반 텍스트로 화면에 찍힌다.

```json
{"name":"read_file","arguments":{"path":"README.md"}}
```

이 상태에서는 파일을 읽지 않았다. 모델과 Ollama, Codex App 사이의 tool-call protocol이 맞지 않은 것이다.

## 실험 1: Qwen2.5 Coder 7B

7B급이라 설치와 실행이 가볍고 짧은 코드 생성은 빨랐다. 그러나 저장소 파일을 찾아 수정하는 요청에서는 다음 문제가 반복됐다.

- tool call처럼 생긴 JSON을 일반 답변으로 출력
- 이전 turn의 요청을 안정적으로 이어가지 못함
- 도구 실행 결과 없이 완료한 것처럼 응답

`coder` 모델이라는 이름은 코드 데이터에 맞춰졌다는 뜻이지, Codex App의 구조화된 tool event를 지원한다는 뜻은 아니다. 로컬 코드 질의응답에는 쓸 수 있지만 agent 작업은 맡기기 어려웠다.

## 실험 2: gpt-oss 20B

20B 모델은 `tools`, `thinking`, 큰 context를 지원해 구조적으로 더 유력해 보였다. 문제는 16GB 통합 메모리 환경에서의 실제 사용성이었다.

| 항목 | 관찰 |
| --- | --- |
| 모델 weight | 약 12~13GB 수준 |
| 추가 메모리 | KV cache, Codex system prompt, 앱과 운영체제 |
| 결과 | swap 증가와 극심한 생성 지연 |

모델이 로드됐다는 사실만으로 실사용 가능한 것은 아니다. Codex App은 긴 지침과 도구 schema를 함께 보내므로 단순 채팅보다 context와 메모리 요구가 크다.

느린 생성과 tool-call 호환성도 분리해서 봐야 한다. 메모리 부족은 보통 속도를 낮추거나 프로세스를 종료시키지만, tool JSON을 일반 텍스트로 만드는 직접 원인은 provider protocol 쪽에 가깝다.

## 실험 3: Gemma 계열

작은 Gemma 모델은 말투가 자연스럽고 메모리 부담도 상대적으로 낮았다. 하지만 저장소 지침을 읽고 적절한 파일을 찾은 뒤 근거에 따라 답하는 흐름이 자주 끊겼다.

- 관련 없는 지침을 먼저 선택
- 필요한 파일 검색으로 이어지지 않음
- web이나 파일 도구가 있어도 사용할 수 없다고 응답
- 더 가벼운 CLI harness에서도 같은 탐색 실패 반복

멀티모달 capability와 자연스러운 대화가 coding agent 능력을 대신하지는 못했다.

## 모델별 결론

| 모델군 | 장점 | Codex App 판단 |
| --- | --- | --- |
| 7B coder | 빠르고 가벼움 | 일반 코드 챗에는 가능, tool protocol은 불안정 |
| 20B tool 모델 | 지침과 tool 개념은 더 잘 이해 | 16GB 환경에서는 메모리와 속도 한계 |
| 작은 Gemma | 자연스러운 대화와 낮은 부담 | 파일 탐색과 근거 검증이 불안정 |

이번 결과는 모든 로컬 모델과 모든 버전에 대한 결론이 아니다. 특정 시점의 Ollama–Codex App 연결과 제한된 하드웨어에서 관찰한 결과다. 모델과 adapter가 바뀌면 다시 검증해야 한다.

## 로컬 모델을 agent로 평가하는 방법

단순히 "코드를 작성해줘"라고 묻는 것보다 다음 조건이 들어간 fixture가 필요하다.

1. 정답 파일을 직접 찾아야 한다.
2. 파일 내용을 읽고 계산해야 한다.
3. tool call이 실제 event로 기록되어야 한다.
4. 수정 후 diff와 테스트가 정답을 증명해야 한다.
5. 같은 요청을 여러 번 실행해 결과가 재현되어야 한다.

최종 답변만 그럴듯한지 보는 테스트로는 agent 신뢰성을 확인할 수 없다.

## 지금의 사용 구분

Ollama는 여전히 유용하다.

- 로컬 모델 다운로드와 관리
- 짧은 코드 질의응답
- 요약·분류·초안 생성
- 외부 API로 보내기 싫은 단순 텍스트 처리

다만 복잡한 저장소에서 파일을 수정하는 주 agent로 쓰려면 모델 이름보다 tool-call protocol, context 여유, provider adapter, 반복 검증이 먼저다.

## 참고 자료

- [Ollama Codex App integration](https://docs.ollama.com/integrations/codex-app)
- [Ollama](https://github.com/ollama/ollama)
- [OpenAI gpt-oss](https://openai.com/index/introducing-gpt-oss)
