---
layout: single
title: "Obsidian AI 비서에게 어떤 스킬을 주면 좋을까 - 기본 스킬과 직접 만든 비서 운영 스킬까지"
date: 2026-04-22 20:00:26 +0900
categories: obsidian
---

처음에는 나도 Obsidian용 기본 스킬 몇 개만 있으면 AI 비서가 금방 돌아갈 줄 알았다. markdown을 읽고, `.base`를 만지고, CLI로 앱까지 건드릴 수 있으니 얼핏 보면 충분해 보이기 때문이다. 그런데 며칠만 써 보면 금방 차이가 드러난다. 파일을 다룰 수 있는 것과, 내 vault에서 무엇을 먼저 읽고 어떻게 판단해야 하는지를 아는 것은 전혀 다른 문제였다. 이번 글은 그 차이를 정리한 글이다. 공식 기본 스킬 5개는 어디까지 해 주는지 짧게 보고, 그다음부터는 내가 이 Obsidian 위에서 비서를 운영하려고 직접 만든 스킬이 왜 필요해졌는지를 실제 사례로 풀어보려 한다.

![Obsidian AI 비서를 위한 기본 스킬과 비서 운영 스킬의 계층을 표현한 대표 이미지](/assets/images/2026/04/22/obsidian-skill-stack-hero.png){: .align-center }

## 기본 스킬의 기준선

이번 글에서 말하는 기본 스킬은 공식 GitHub [`kepano/obsidian-skills`](https://github.com/kepano/obsidian-skills)를 기준으로 잡는다. README를 보면 현재 공식 팩에는 5개가 들어 있다. 이 다섯 개는 Obsidian을 다루는 바닥 능력을 넓게 깔아 주는 층에 가깝다. 노트 문법, Bases, Canvas, CLI, 웹 본문 정리처럼 “공통으로 필요한 일”은 대부분 여기서 출발한다.

| 기본 스킬 | 한 줄 역할 | 왜 필요한가 |
| --- | --- | --- |
| `obsidian-markdown` | Obsidian 문법에 맞는 markdown을 읽고 쓴다 | 위키링크, callout, frontmatter를 덜 틀리게 다룰 수 있다 |
| `obsidian-bases` | `.base` 파일 구조와 뷰 구성을 다룬다 | Bases를 표, 카드, 칸반 관점에서 수정할 수 있다 |
| `json-canvas` | `.canvas` 파일의 node와 edge 구조를 다룬다 | 시각 구조도도 결국 파일이라는 관점으로 다룰 수 있다 |
| `obsidian-cli` | 실행 중인 Obsidian과 상호작용한다 | 읽기, 검색, 생성, 스크린샷까지 앱 레벨 작업이 가능하다 |
| `defuddle` | 웹페이지 본문을 markdown으로 정리한다 | 외부 글을 vault로 들여오기 전에 잡음을 줄일 수 있다 |

여기까지만 보면 꽤 든든하다. 실제로도 이 다섯 개만으로 할 수 있는 일이 적지 않다. 다만 이 스킬들은 어디까지나 Obsidian을 다루는 공용 능력이다. 내 vault의 기준 문서가 무엇인지, 어떤 정보는 장기 메모로 올리고 어떤 것은 그냥 메모에 남겨야 하는지 같은 판단까지 대신해 주지는 않는다.

## 전체 스킬 맵

지금 내가 실제로 굴리는 구성은 단순하다. 공식 기본 스킬 5개가 바닥에 있고, 그 위에 직접 만든 비서 운영 스킬 2개가 얹혀 있다. 위층은 공용 능력이고, 아래층은 내 운영 방식이다.

| 스킬 | 층위 | 맡는 일 | 실제 비중 |
| --- | --- | --- | --- |
| `obsidian-markdown` | 기본 스킬 | 노트 문법과 링크 구조를 안정적으로 다룬다 | 기준선 |
| `obsidian-bases` | 기본 스킬 | `.base` 파일의 필터, 뷰, order를 수정한다 | 기준선 |
| `json-canvas` | 기본 스킬 | `.canvas` 파일의 node, edge, group을 편집한다 | 보조 |
| `obsidian-cli` | 기본 스킬 | Obsidian 앱과 vault를 읽고 쓰고 검색한다 | 높음 |
| `defuddle` | 기본 스킬 | 웹 글을 깔끔한 markdown으로 바꾼다 | 보조 |
| `daily-note-followup` | 비서 운영 스킬 | 일간 기록 후처리와 장기 메모, TODO, 플레이스 분기를 묶는다 | 핵심 |
| `organize-instructions` | 비서 운영 스킬 | 지침 구조를 점검하고 다시 정리한다 | 핵심 |

표로 보면 그냥 7개 나열처럼 보이지만, 성격은 꽤 다르다. 기본 스킬은 범위가 넓은 대신 얕다. 반대로 비서 운영 스킬은 범위는 좁지만 훨씬 깊다. 실제로는 읽어야 하는 기준 문서와 응답 형식까지 한 덩어리로 붙는다.

## 비서 운영 스킬이 붙는 자리

기본 스킬만으로도 꽤 많은 일은 된다. 그런데 어느 지점부터는 금방 한계가 보인다. 파일 포맷을 아는 것과, 내 vault에서 무엇이 기준 문서인지 아는 것은 완전히 다른 일이기 때문이다.

| 작업 유형 | 기본 스킬만으로 되는가 | 비서 운영 스킬이 필요한 이유 |
| --- | --- | --- |
| 위키링크와 frontmatter를 맞춰 노트를 쓴다 | 대부분 된다 | Obsidian 문법 자체는 공용 규칙이기 때문이다 |
| `.base`나 `.canvas` 파일 구조를 수정한다 | 대부분 된다 | 파일 스키마를 이해하는 일이 중심이기 때문이다 |
| 웹 글을 읽어 markdown으로 정리한다 | 대부분 된다 | 본문 추출 자체는 공용 도구 문제이기 때문이다 |
| 일간 기록을 읽고 일정, 장기 메모, TODO로 분기한다 | 어렵다 | 내 vault의 섹션 경계, 기준 문서, 응답 형식이 로컬 규칙이기 때문이다 |
| 지침 문서 구조를 정리하고 기준 문서를 다시 잡는다 | 어렵다 | 어떤 문서가 기준인지와 어디로 detail을 내려야 하는지가 저장소마다 다르기 때문이다 |

내가 직접 써 보며 가장 크게 느낀 차이도 여기였다. 기본 스킬은 “이 파일이 무엇인가”를 안다. 비서 운영 스킬은 “이 vault에서는 무엇을 먼저 읽고, 어디에 적고, 무엇은 섣불리 올리면 안 되는가”를 안다. 개인 비서처럼 오래 쓰려면 결국 뒤쪽이 필요해진다.

스킬을 언제 만들지도 여기서 갈린다. 그냥 시켜도 늘 잘 되는 일이라면 굳이 스킬로 만들지 않아도 된다. 반대로 원하는 능력은 분명한데 결과가 계속 마음에 들지 않아서, 피드백을 주고 결과물을 여러 번 고치게 되는 일이 있다. 내 경우에는 그런 작업이야말로 스킬 후보였다. 몇 번의 피드백 끝에 “이 정도면 내가 원하던 결과다” 싶은 상태가 되면, 그때까지의 피드백과 최종 결과물을 바탕으로 “다음에도 같은 요구가 오면 이렇게 처리해 달라”고 정리해서 스킬로 올리는 쪽이 가장 잘 맞았다.

## 비서 운영 스킬 사례

내가 직접 붙인 비서 운영 스킬은 `daily-note-followup`와 `organize-instructions` 두 개다. 둘 다 공용 스킬 위에서 움직이지만, 실제 느낌은 “프롬프트 두 개”와는 거리가 멀다. 차라리 작은 작업 패키지에 가깝다. `SKILL.md`가 있고, 세부 판단을 담은 참조 문서가 있고, 필요하면 `agents/openai.yaml`까지 붙는다.

| 비서 운영 스킬 | 해결하는 문제 | 읽는 참조 문서 | 왜 직접 만들어야 하는가 |
| --- | --- | --- | --- |
| `daily-note-followup` | 일간 기록을 후처리하고 장기 메모, TODO, 플레이스, 미래 일정으로 분기한다 | 플레이북, 체크리스트, `references/action-rubric.md` | 섹션 구조와 승격 기준이 내 vault 운영 방식에 묶여 있기 때문이다 |
| `organize-instructions` | 지침 문서를 점검하고 더 안정적인 구조로 다시 묶는다 | `references/discovery-audit-workflow.md`, `classification-framework.md`, `output-contract.md` | 같은 구조 정리라도 저장소마다 라우터 문서와 기준 문서가 다르기 때문이다 |

### daily-note-followup

이 스킬은 이름만 보면 “일기 정리” 정도로 들린다. 그런데 실제로 뜯어 보면 훨씬 넓다. 완성된 일간 기록 하나를 읽고 나서, 어디까지 자동 반영할지 정해 둔 운영 절차에 가깝다. 일기 교정, 일정 보정, 장기 메모 승격, TODO 생성, 장소 노트 승격, 미래 노트 생성이 한 흐름으로 이어진다.

폴더 구조도 이렇게 생겨 있다. `SKILL.md`만 있는 게 아니라, 판단 기준과 예시가 `references` 아래로 빠져 있다.

```text
daily-note-followup/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── action-rubric.md
    ├── examples.md
    └── place-note-pattern.md
```

이 구조를 보면 역할도 바로 나뉜다. `SKILL.md`는 메인 흐름을 잡고, `action-rubric.md`는 자동 반영 기준과 섹션 경계를 더 세게 못 박는다. `place-note-pattern.md`는 장소 노트를 어떻게 만들지 패턴을 잡아 주고, `examples.md`는 실제 결과 모양을 맞출 때 참고점으로 쓴다.

`SKILL.md`의 가운데 부분만 추리면 흐름은 대략 이렇게 읽힌다.

```markdown
1. Read the target daily note first.
2. Locate the user-only block and the `## 일기` section before touching anything else.
7. Sync the agent-managed sections after diary correction.
8. Before mutating the vault, use these canonical references:
   - `00_agent_docs/플레이북/Base Board 작업 지침.md`
   - `00_agent_docs/플레이북/일일기록 일정 작성 플레이북.md`
   - `00_agent_docs/플레이북/개인 정보 업데이트 플레이북.md`
9. Apply the repository-specific rules in `references/action-rubric.md`.
17. Create or update `06_TODO`, future daily notes, `10_아이디어`, and month or year hubs only when the rubric says the evidence is strong enough.
```

이 발췌에서 중요한 건 문장이 길다는 게 아니다. 스킬이 혼자 판단하지 않는다는 점이다. 먼저 일간 기록을 읽고, 그다음 기준 문서를 읽고, 그 기준 위에서만 움직인다. 그래서 이 스킬은 공용으로 돌리기 어렵다. 일간 기록에서 무엇을 장기 메모로 올릴지, 어떤 TODO를 자동 생성할지, `## 메모`와 `## 에이전트 메모`를 어디서 나눌지는 결국 내 운영 규칙이기 때문이다.

하위 참조 문서는 이 판단을 더 촘촘하게 만든다. 예를 들어 `action-rubric.md`는 섹션 경계와 자동 반영 기준을 이렇게 적어 둔다.

```markdown
- `## 일정`: Day Planner-compatible list only.
- `## 메모`: Supplementary facts not already covered in the diary.
- `## 에이전트 메모`: agent's own work log and future reference observations.
- Auto-create a TODO only when the next action, actor, and intent are clear.
```

`references` 내부 파일도 역할이 나뉘어 있다.

- `action-rubric.md`: 자동 반영 범위, 섹션 경계, TODO 생성 조건처럼 가장 자주 흔들리는 판단 기준을 잡는다.
- `examples.md`: 이 스킬이 언제 발동하고 어떤 결과 모양으로 끝나야 하는지 예시로 맞춘다.
- `place-note-pattern.md`: 장소 노트를 언제 승격하고 어떤 구조로 만들지 패턴을 정리한다.

그리고 `agents/openai.yaml`은 로직보다 표면을 다듬는 층에 가깝다. 여기서 이 스킬이 어떤 이름으로 보이고, 암묵 호출을 열어둘지 같은 사용감을 정리한다.

```yaml
interface:
  display_name: "일기 정리"
policy:
  allow_implicit_invocation: true
```

### organize-instructions

`organize-instructions`는 2편과 3편에서 계속 이야기했던 문제를 실제로 다루는 스킬이다. 문서가 많아졌을 때 무엇이 라우터이고, 무엇이 기준 문서이고, detail을 어디로 내려야 하는지를 감으로 정리하지 않게 해 준다. 한마디로 하면 지침 구조를 점검하는 정리 스킬이다.

이 스킬도 마찬가지로, 프롬프트 한 장이라기보다 작은 점검 도구 세트에 가깝다.

```text
organize-instructions/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── classification-framework.md
    ├── discovery-audit-workflow.md
    ├── local-example-obsidian-vault.md
    └── output-contract.md
```

여기서는 `discovery-audit-workflow.md`가 실제 지침 면을 어떻게 훑을지 안내하고, `classification-framework.md`가 규칙을 어떤 층으로 분류할지 잡아 준다. `output-contract.md`는 결과를 어떤 형식으로 내야 하는지 정리하고, `local-example-obsidian-vault.md`는 필요할 때만 참고하는 예시 축에 가깝다.

이쪽 `SKILL.md`도 중심 흐름은 꽤 분명하다.

```markdown
1. Discover the instruction surface.
2. Classify each rule before moving it.
3. Diagnose the current problems.
4. Design the target architecture.
5. Translate the design into concrete actions.
6. Produce the final output with the correct contract.
```

내가 이 스킬을 따로 둔 이유도 여기에 있다. 단순히 “문서를 정리한다”가 아니라, 발견하고, 분류하고, 문제를 짚고, 구조를 다시 그리는 순서를 강제하기 때문이다. 그냥 손에 잡히는 파일부터 옮기기 시작하면 중복만 더 늘어날 때가 많았다.

하위 참조 문서는 이 흐름을 더 구체적으로 잘라 준다. `discovery-audit-workflow.md`만 봐도, 처음부터 파일을 옮기지 말고 실제 지침 면을 먼저 훑으라고 못 박는다.

```markdown
1. Top-level router docs
2. Named operating areas
3. Task-specific instruction packages
4. Preference and operating context docs
```

`references` 내부 파일도 각각 맡는 역할이 다르다.

- `discovery-audit-workflow.md`: 실제 지침 표면을 어떤 순서로 훑고 inventory할지 정리한다.
- `classification-framework.md`: 발견한 규칙을 어떤 층과 문서로 보낼지 분류 기준을 잡는다.
- `output-contract.md`: 점검 결과를 어떤 형식으로 정리해 내놓을지 맞춘다.
- `local-example-obsidian-vault.md`: 필요할 때만 참고하는 예시 자료로, 실제 구조를 감 잡는 데 쓴다.

이 스킬은 `daily-note-followup`와 달리 암묵 호출을 열어두지 않았다. 매일 도는 루틴이라기보다, “지금 지침 구조를 점검해야겠다” 싶을 때 일부러 꺼내는 도구에 더 가깝기 때문이다.

```yaml
interface:
  display_name: "Organize Instructions"
policy:
  allow_implicit_invocation: false
```

## 직접 커스텀 스킬을 만들어야 하는 시점

스킬은 많다고 좋은 게 아니다. 먼저 내 비서에게 어떤 능력이 필요한지 파악하는 게 먼저다. 그냥 시켜도 늘 잘 되는 일이라면 굳이 스킬로 고정할 필요가 없다. 내가 실제로 스킬을 만들게 되는 시점은, 결과가 계속 흔들리거나 같은 작업을 반복해서 시키게 될 때였다.

### 에이전트의 결과물이 마음에 안 들 때

결과가 조금 아쉽다고 바로 스킬을 만들지는 않는다. 보통은 먼저 피드백을 주면서 내가 원하는 결과가 나올 때까지 계속 다듬는다. 이 단계가 중요한 이유는, 내가 원하는 결과물의 기준이 아직 선명하지 않을 때가 많기 때문이다.

내가 보통 거치는 흐름은 이렇다.

- 먼저 그냥 시켜 본다.
- 결과가 자꾸 흔들리면 피드백을 주면서 원하는 모양으로 계속 고친다.
- 몇 번 반복한 뒤 “이 정도면 다음에도 같은 요구를 잘 처리하겠다” 싶은 결과물이 나온다.
- 그때까지 쌓인 피드백과 최종 결과물을 바탕으로 플레이북을 만든다.

이때 플레이북에 남기는 재료도 꽤 분명하다.

- 지금까지의 피드백
- 대화 내역
- 실제 작업 흐름
- 최종 결과물

플레이북을 먼저 만드는 이유도 분명하다.

- 시행착오를 줄이기 위해
- 다음에 같은 실수를 하지 않게 하기 위해
- 같은 요구가 다시 왔을 때 기준 문서로 삼기 위해

즉, 스킬보다 플레이북이 먼저 생기고, 스킬은 그 플레이북이 반복 실행 단위가 될 때 올라간다.

### 반복해서 시키는 작업이 생겼을 때

어떤 작업은 한 번 잘 끝나는 것으로는 부족하다. 다음 주에도, 다음 달에도 비슷한 요청을 또 하게 된다. 이런 작업은 단순 플레이북을 넘어 스킬 후보가 된다.

여기서 내가 나누는 기준은 비교적 단순하다. 플레이북은 사람이 읽어도 되는 절차 문서이고, 스킬은 에이전트가 명시적으로 호출해서 같은 작업을 반복 처리하게 만드는 실행 단위다.

특히 아래 조건이 겹치면 스킬로 올리기 좋았다.

- 요청 형태가 반복된다.
- 원하는 결과물 모양이 비교적 명확하다.
- 이미 플레이북과 피드백이 쌓여 있다.
- 다음에도 같은 기준으로 처리되길 원한다.

내 경우에는 이런 작업이 쌓일 때, 기존 플레이북과 원하는 결과물을 바탕으로 하나의 스킬로 승화시키는 편이 잘 맞았다. 그러면 다음부터는 매번 긴 설명을 다시 하지 않아도 되고, 비서도 같은 기준 위에서 움직이기 쉬워진다.

여기서 중요한 건 두 가지다. 하나는 스킬을 너무 빨리 만들지 않는 것이다. 그냥 시켜도 잘 되는 일은 굳이 스킬로 고정할 필요가 없다. 다른 하나는 공용 기본 스킬과 역할을 겹치지 않는 것이다. 기본 스킬이 이미 markdown, `.base`, `.canvas`, CLI, 웹 인입을 맡아주고 있다면, 비서 운영 스킬은 그 위에 내 판단 기준과 참조 문서만 얹는 편이 훨씬 깔끔하다. 모든 걸 로컬 스킬 하나에 우겨 넣기 시작하면 금방 다시 두꺼워진다.

## 마무리

지금 돌아보면 공식 기본 스킬 5개는 꽤 좋은 출발선이었다. 다만 거기서 멈췄다면 그냥 “Obsidian을 잘 다루는 AI” 정도였을 것이다. 내 vault에서 실제로 오래 쓰는 개인 비서가 되기 시작한 건, 직접 만든 비서 운영 스킬이 붙은 뒤였다.

결국 기본 스킬은 바닥이고, 비서 운영 스킬은 내 운영 방식이다. 내가 지금 쓰는 구조도 두 층을 섞기보다 겹쳐 놓는 쪽에 가깝다. 기본 스킬은 공용 능력을 맡고, 비서 운영 스킬은 내 기준 문서와 작업 습관을 맡는다. 지금으로서는 그 분리가 가장 덜 흔들린다.
