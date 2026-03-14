---
layout: single
title: "Playwright 사용해보기 - MCP와 Skill의 차이 (feat. Codex)"
date: 2026-03-14 14:00:00 +0900
categories: ai
---

AI가 손쉽게 브라우저를 조작할 수 있도록 만들어주는 Playwright를 사용해보자.
GPT 5.4에서 `playwright-interactive` 스킬 기반으로 웹 앱/게임을 만들었다고 하는데, 이 Playwright가 뭘까?

> Weekly Tech Trend Talk 스터디(26.03.12)

---

## Playwright MCP

Codex에 Playwright MCP를 추가하면 AI가 직접 브라우저를 조작할 수 있게 된다.

```bash
codex mcp add playwright -- npx @playwright/mcp@latest
```

### 추가되는 Tool 목록

MCP를 추가하면 다양한 브라우저 조작 도구들이 생긴다. 의미별로 묶으면 다음과 같다.

- **페이지 이동:** `navigate`, `navigate_back`, `tabs`, `close`
- **입력/조작:** `click`, `hover`, `type`, `fill_form`, `select_option`, `press_key`, `drag`, `handle_dialog`, `file_upload`
- **관찰/디버깅:** `snapshot`, `take_screenshot`, `console_messages`, `network_requests`, `evaluate`, `run_code`
- **환경 보조:** `resize`, `wait_for`, `install`

이 tool들을 AI가 적절히 조합해서 브라우저를 자동화한다.

---

## Playwright Skills

Codex에는 `skill-installer` 스킬로 설치할 수 있는 `playwright`, `playwright-interactive` 같은 Playwright 관련 스킬들이 있다.
`skill-installer`는 [openai/skills](https://github.com/openai/skills) 레포의 curated 목록에서 스킬을 내려받아 설치한다.

MCP와 Skill은 경쟁 관계가 아니라 역할이 다르다.

- **MCP**는 Codex에 실제 브라우저 tool을 연결해 주는 방식이다. 외부 시스템에 대한 **실행 능력**을 추가한다.
- **Skill**은 반복해서 쓰는 절차를 `SKILL.md`, scripts, references로 묶어 둔 **워크플로우 패키지**다.

Skill은 metadata만 먼저 보이고, 선택되었을 때만 `SKILL.md`와 필요한 references/scripts를 읽는 progressive disclosure 방식으로 설계되어 있다.
반면 MCP는 tool 표면이 넓어질수록 action surface가 커지기 때문에, OpenAI 문서에서도 처음부터 모든 tool을 붙이지 말고 실제 workflow에 필요한 몇 개부터 연결하라고 권장한다.

### playwright 스킬

`playwright` 스킬은 브라우저 tool을 새로 추가하는 것이 아니라,
Codex가 `playwright-cli` 또는 번들 wrapper script를 사용해 브라우저를 자동화하도록 안내하는 **CLI 중심 워크플로우**다.

- MCP처럼 `mcp__playwright__...` tool이 생기지는 않는다
- `open`, `snapshot`, `click`, `fill`, `screenshot` 같은 절차를 CLI 명령으로 재사용하도록 돕는다
- 브라우저 capability를 늘리는 것보다, **절차를 재사용 가능하게 만드는 것**에 가깝다

### playwright-interactive 스킬

`playwright-interactive` 스킬은 `js_repl` 안에서 Playwright 세션을 계속 유지하면서,
같은 브라우저나 Electron 앱 핸들을 재사용해 반복 디버깅하는 워크플로우다.

![playwright-interactive로 만든 게임들](/assets/images/2026/03/14/playwright-interactive-games.png)

**GPT 5.4가 이 스킬을 사용하여 위와 같은 게임들을 만들었다고 한다.**
스크린샷은 테마파크 시뮬레이션 게임으로, 이소메트릭 뷰의 놀이공원을 운영하는 "Miniature" 게임이다.
이 외에도 RPG 게임, 골든 게이트 브리지 상공 비행 게임 등 다양한 장르를 생성했다.

- MCP처럼 tool을 추가하는 것이 아니라, **지속 세션 기반 디버깅 절차**를 제공한다
- 코드 변경 후 reload/relaunch를 반복하며 QA와 시각 확인을 수행하기 좋다
- 단발성 CLI 실행보다 로컬 웹 앱이나 Electron 앱을 오래 붙들고 점검하는 데 더 적합하다

---

## 정리

| 구분 | MCP                       | Skill                                                |
| ---- | ------------------------- | ---------------------------------------------------- |
| 역할 | 실제 tool을 붙이는 레이어 | tool/CLI를 어떤 순서와 방식으로 쓸지 정의하는 레이어 |
| 동작 | 브라우저 조작 도구 추가   | 워크플로우 절차 제공                                 |
| 비용 | action surface 증가       | progressive disclosure로 최소화                      |

MCP는 실행 능력을 제공하고, Skill은 그 능력을 효과적으로 사용하는 방법을 제공하는 관계라고 볼 수 있다.
