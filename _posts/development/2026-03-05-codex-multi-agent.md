---
layout: single
title: "Codex에 Multi Agent 기능 추가"
date: 2026-03-05 21:00:00 +0900
categories: development
---

Claude Code에만 있던 **Sub Agent(서브 에이전트)** 기능이
[OpenAI Codex에도 공식 추가](https://developers.openai.com/codex/multi-agent/)되었다.

---

## Multi Agent 핵심

2월 2일 출시된 Codex 앱은 여러 AI 코딩 에이전트를 하나의 인터페이스에서 관리하고 병렬 실행할 수 있다.

- 현재 대화에서 **서브 에이전트로 분기(fork)** 가능
- CSV 기반으로 작업을 팬아웃하는 `spawn_agents_on_csv`
- 각 에이전트가 **별도의 worktree**에서 격리 작업 (같은 레포에서 충돌 없이 병렬 개발)
- 닉네임, 진행률/ETA 표시 등 관리 기능 강화

---

## 실험 기능 켜는 법

공식 문서 기준으로 Multi Agent는 `experimental` 기능이다.

### 1) Codex CLI 실행

```bash
codex
```

### 2) 실험 메뉴에서 Multi-agent 활성화

대화창에서 `/experimental`을 입력한 뒤 **Multi-agents** 옵션을 켠다.

### 3) Codex 재시작

옵션을 켠 뒤 Codex를 재시작하면 바로 Multi Agent를 사용할 수 있다.

---

## Multi Agent 사용법

가장 간단한 방법은 Codex에게 작업 분할을 직접 지시하는 것이다.

```text
멀티 에이전트로 병렬 처리해줘.
- agent A: 점심 메뉴 3개 추천(한식)
- agent B: 점심 메뉴 3개 추천(일식)
마지막에 두 목록을 합쳐 최종 1개만 추천해줘.
```

Codex는 작업을 쪼개 여러 에이전트를 생성하고, 각 에이전트 진행 상태를 메인 대화에 인라인으로 보여준다.

도구 레벨로 보면 흐름은 보통 다음 순서다.

1. `spawn_agent`로 작업별 에이전트 생성
2. `send_input`으로 각 에이전트에 상세 지시 전달
3. `wait`로 완료 대기 후 결과 수집

---

## 출력 예시

아래는 위 프롬프트를 실제로 실행했을 때의 화면 예시다.

![Codex Multi Agent 점심 메뉴 병렬 실행 예시](/assets/images/2026/03/05/codex-multi-agent-lunch-example-real.png)

---

## 제한 사항

Claude Code 사용자 입장에서는 이미 익숙한 기능이지만,
Codex가 공식적으로 지원하면서 AI 코딩 도구의 멀티 에이전트 아키텍처가 업계 표준으로 자리잡는 흐름이다.

복잡한 프로젝트에서 여러 에이전트가 각자 독립된 브랜치에서 작업하고,
결과를 병합하는 방식은 기존 개발자들의 협업 방식과 유사하다.
AI 코딩 도구가 단순한 자동완성을 넘어,
팀 단위의 개발 워크플로우를 시뮬레이션하는 방향으로 진화하고 있다.

참고로 공식 문서 기준으로 **Codex Cloud task에서는 Multi Agent를 아직 지원하지 않는다.**

---

## 참고 문서

- [Codex Multi-Agent 문서](https://developers.openai.com/codex/multi-agent/)
- [Codex Config - Experimental features](https://developers.openai.com/codex/config#experimental-features)
