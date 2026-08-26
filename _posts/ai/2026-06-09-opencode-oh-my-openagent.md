---
layout: single
title: "OpenCode와 Oh My OpenAgent는 무엇이 다를까 - 실행 런타임과 오케스트레이션의 차이"
date: 2026-06-09 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/06/09/opencode-omo-hero.png
---

OpenCode와 Oh My OpenAgent(OMO)를 함께 소개하는 글을 보면 둘의 경계가 흐리게 느껴진다. OpenCode는 모델이 파일과 터미널 도구를 사용하는 **코딩 에이전트 런타임**이고, OMO는 그 위에 역할 분담과 작업 루프를 얹는 **플러그인형 오케스트레이션 레이어**다.

![코드 편집기와 검색 편집 테스트 모듈을 조율하는 오케스트레이션 구조](/assets/images/2026/06/09/opencode-omo-hero.png){: .align-center }

둘은 경쟁 제품이 아니다. OpenCode가 실행면을 제공하고 OMO가 그 실행면을 여러 specialist와 workflow로 확장한다.

## OpenCode가 맡는 것

[OpenCode](https://github.com/anomalyco/opencode)는 터미널에서 동작하는 오픈소스 코딩 에이전트다. 모델 provider를 고르고, 저장소 파일을 읽고 수정하며, 셸 명령과 외부 도구를 호출하는 기반을 제공한다.

| 기능 | 역할 |
| --- | --- |
| TUI·CLI | 터미널에서 모델과 대화하고 작업 실행 |
| Web UI | 같은 세션을 브라우저에서 사용 |
| 파일·셸 도구 | Read, Edit, Glob, Grep, Bash 등 |
| MCP | 브라우저, 문서, 데이터 도구 연결 |
| Permission | 도구 호출 허용과 차단 |
| Agent | 계획·구현처럼 역할별 지침 정의 |
| Skill | 반복 작업의 규칙과 참고 자료 로드 |
| Provider | OpenAI, Anthropic, 로컬 OpenAI 호환 서버 등 연결 |

전역 설정은 보통 다음 위치에 둔다.

```text
~/.config/opencode/opencode.jsonc
```

프로젝트 자체의 작업 규칙은 저장소의 `AGENTS.md` 같은 파일로 분리한다. 여기까지가 플러그인을 붙이지 않은 OpenCode의 책임이다.

## OMO가 추가하는 것

[Oh My OpenAgent](https://github.com/code-yeongyu/oh-my-openagent)는 OpenCode plugin으로 설치된다. 본체를 대체하지 않고 agent 역할, 위임, 모델 routing, 계획–실행–검증 workflow를 한 묶음으로 추가한다.

```jsonc
{
  "plugin": ["oh-my-openagent"]
}
```

설정과 제공 기능은 버전에 따라 빠르게 바뀐다. 아래 구조는 2026년 6월에 살펴본 계열을 기준으로 한다.

### 메인 agent와 specialist

하나의 agent가 탐색, 구현, 문서 조사, 검증을 모두 끌어안는 대신 역할을 나눈다.

| 역할 | 맡는 작업 |
| --- | --- |
| 메인 오케스트레이터 | 사용자 목표 해석, 계획, 위임, 결과 통합 |
| Explore | 현재 저장소의 파일과 패턴 탐색 |
| Librarian | 공식 문서와 외부 자료 조사 |
| Oracle | 설계·디버깅·검증 관점의 검토 |
| 멀티모달 specialist | 이미지와 스크린샷 분석 |

역할을 나누는 이유는 agent 이름을 늘리기 위해서가 아니다. 빠른 저장소 검색과 느린 설계 검토를 같은 모델, 같은 context에 넣지 않고 작업 성격에 맞춰 분리하기 위해서다.

### Category routing

위임할 때 특정 모델명을 직접 고르는 대신 `quick`, `writing`, `visual-engineering`, `deep` 같은 작업 category를 넘긴다. category 설정이 실제 모델과 fallback chain을 결정한다.

```text
메인 agent
→ category: quick
→ 빠르고 저렴한 모델

메인 agent
→ category: deep
→ 긴 추론에 맞는 모델
```

provider를 바꾸더라도 workflow 본문에서 모델명을 일일이 교체하지 않아도 되는 구조다.

### 계획–실행–검증 workflow

OMO는 단발성 prompt보다 긴 작업을 위한 workflow를 제공한다.

- 요구사항을 인터뷰와 계획으로 구체화
- 승인된 계획을 구현 단계로 전달
- 저장소 탐색과 외부 조사를 specialist에 위임
- 구현 후 별도 검토 단계 실행
- context가 길어지면 다음 세션으로 handoff
- 완료 조건을 만족할 때까지 반복 실행

기능 이름과 명령은 버전별로 바뀔 수 있지만, 중심은 **계획과 실행, 검증을 같은 agent의 즉흥 판단에만 맡기지 않는 것**이다.

## 편집과 탐색 안정성

에이전트가 긴 작업에서 자주 실패하는 곳은 모델 답변보다 파일 편집 경계다.

| 보조 기능 | 해결하려는 문제 |
| --- | --- |
| Hash-anchored edit | 마지막으로 읽은 뒤 파일이 바뀌었을 때 오래된 기준으로 덮어쓰기 방지 |
| LSP 도구 | 심볼 정의·참조·rename을 문자열 검색보다 정확하게 처리 |
| AST 기반 검색 | 언어 구조를 기준으로 코드 패턴 탐색과 수정 |
| MCP 묶음 | 문서와 코드 검색 도구를 필요할 때 연결 |
| 작업 검토 | 구현이 끝났다는 자기 판단을 별도 단계에서 재검증 |

이런 기능은 모델의 지능을 바꾸지 않는다. 대신 모델이 실수했을 때 저장소가 손상되는 범위를 줄이고, 실패를 더 일찍 드러낸다.

## OpenCode만으로 충분한 경우

- 한 저장소에서 한 모델로 짧은 작업을 수행한다.
- 사용하는 agent와 provider가 많지 않다.
- 계획과 검토를 사람이 직접 통제한다.
- 추가 plugin이 넣는 context와 동작 복잡도를 원하지 않는다.

## OMO가 유용한 경우

- 저장소 탐색, 외부 조사, 구현, 검증을 분리하고 싶다.
- 작업 성격에 따라 여러 모델을 자동 routing하고 싶다.
- 긴 작업을 계획과 acceptance criteria에 맞춰 반복 실행한다.
- 여러 specialist의 결과를 하나의 흐름으로 통합해야 한다.

오케스트레이션을 추가하면 context와 실패 지점도 함께 늘어난다. 작은 수정까지 무조건 다중 agent로 보내는 것보다, 작업 규모가 커질 때 켜는 편이 낫다.

## 문제를 어느 층에서 볼까

| 증상 | 먼저 볼 곳 |
| --- | --- |
| 모델 연결, 파일 도구, MCP, permission 오류 | OpenCode |
| specialist 선택, category routing, 반복 workflow 오류 | OMO |
| 특정 저장소에서만 잘못된 행동 | 저장소 지침과 skill |
| 모델별 tool-call 형식 문제 | provider adapter와 모델 template |

OpenCode는 에이전트가 움직이는 바닥이고 OMO는 그 위의 작업 운영 방식이다. 이 층을 나눠 보면 설치와 문제 진단이 훨씬 단순해진다.

## 참고 자료

- [OpenCode](https://github.com/anomalyco/opencode)
- [OpenCode Docs](https://opencode.ai/docs/)
- [Oh My OpenAgent](https://github.com/code-yeongyu/oh-my-openagent)
