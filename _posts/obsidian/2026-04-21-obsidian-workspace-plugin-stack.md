---
layout: single
title: "Obsidian을 메모 앱이 아니라 작업 공간으로 만드는 핵심 플러그인 구성"
date: 2026-04-21 16:00:00 +0900
categories: obsidian
---

VS Code 위에서도 개인 비서용 환경을 만들 수는 있다. 문제는 일정 화면, 칸반 보드, 템플릿, 첨부 정리까지 붙이기 시작하면 다시 도구를 만드는 일로 돌아가기 쉽다는 점이다. 내가 Obsidian을 계속 쓰는 이유는 markdown 파일 기반이라서만이 아니라, 이미 잘 만들어진 UI/UX와 플러그인 생태계 위에 에이전트를 얹으면 바닥부터 다시 짓지 않고도 통일성 있는 작업 공간을 만들 수 있기 때문이다.

![Obsidian을 작업 공간으로 만드는 플러그인 조합과 AI 보조 운영을 상징적으로 표현한 대표 이미지](/assets/images/2026/04/21/obsidian-plugin-stack-hero.png){: .align-center }

## 작업 공간으로서의 Obsidian

이번 글은 실제로 남긴 핵심 플러그인 여섯 개가 각각 무엇을 맡고, 같이 붙었을 때 어떻게 작업 공간이 되는지를 정리한 글이다. Obsidian은 파일을 저장하는 앱에 그치지 않고, 이미 갖춰진 탐색 UI, 링크 구조, 사이드바, 뷰어, 플러그인 확장점 위에 작업 흐름을 얹을 수 있다. 그래서 에이전트도 바닥부터 앱을 다시 만들 필요 없이, 이미 만들어진 인터페이스와 고정된 규칙 위에서 움직이면 된다.

이 위에는 전역 Obsidian 스킬이 기본 읽기·쓰기 능력을 얹고, Vault 전용 로컬 스킬이 내 작업 절차를 얹는다. 다만 이번 글의 중심은 실제 플러그인 구성이고, 전역/로컬 스킬의 분업은 5편에서 따로 정리하겠다.

| 흐름 | 플러그인                                                       | 역할                                         | 왜 남겼는가                                                             |
| ---- | -------------------------------------------------------------- | -------------------------------------------- | ----------------------------------------------------------------------- |
| 기록 | `periodic-notes`, `templater-obsidian`, `obsidian-day-planner` | 날짜 경로 생성, 템플릿 고정, 타임라인 시각화 | 일간 기록이 같은 형식으로 생성되고 바로 실행 인터페이스로 이어지기 때문 |
| 실행 | `base-board`, `dataview`                                       | TODO 시각화, 조회와 집계 보강                | 데이터는 파일에 남기고 UI는 가볍게 얹을 수 있기 때문                    |
| 정리 | `obsidian-custom-attachment-location`                          | 첨부 경로 자동 정렬                          | 이미지와 첨부를 다룰 때 에이전트 실패가 크게 줄어들기 때문              |

## 기록 플러그인 구성

기록 쪽에서는 `Periodic Notes`, `Templater`, `Day Planner` 세 개가 한 묶음으로 움직인다. 먼저 각 플러그인이 하는 일을 짧게 보면 아래와 같다.

| 플러그인 | 기본 역할 | 내가 쓰는 방식 | 에이전트 입장에서 좋은 점 |
| --- | --- | --- | --- |
| `periodic-notes` | 날짜 기반 노트 생성 경로를 고정한다 | 일간 기록을 `12_일일기록/YYYY/MM/YYYY-MM-DD.md` 규칙으로 만든다 | 날짜 노트 위치를 매번 추측하지 않아도 된다 |
| `templater-obsidian` | 템플릿을 자동 적용한다 | 일간 기록 경로 regex에 맞춰 같은 템플릿을 붙인다 | 섹션 구조가 항상 같은 출발점에서 시작된다 |
| `obsidian-day-planner` | 일정 섹션을 타임라인으로 보여준다 | `## 일정` 섹션을 읽어 시간표 UI로 바꾼다 | 에이전트는 markdown만 정리하고 화면 출력은 플러그인이 맡는다 |

### Periodic Notes와 Templater

기록 흐름은 노트 생성 규칙과 기본 섹션이 고정될 때 비로소 에이전트 친화적으로 바뀐다. `Periodic Notes`는 날짜 기반 생성 경로를 고정하고, `Templater`는 그 경로에 맞는 템플릿을 자동 적용한다. 이 둘이 같이 붙어 있어야 일간 기록이 단순한 빈 파일이 아니라, 처음부터 `일기`, `일정`, `메모`, `에이전트 메모` 같은 고정된 구조를 가진 작업 단위가 된다.

실제 설정은 거의 이렇게 읽힌다.

```json
// periodic-notes
"daily": {
  "enabled": true,
  "format": "YYYY/MM/YYYY-MM-DD",
  "folder": "12_일일기록",
  "template": "_Templates/일일기록 일간 노트.md"
}

// templater-obsidian
{
  "regex": "^12_일일기록/\\d{4}/\\d{2}/\\d{4}-\\d{2}-\\d{2}\\.md$",
  "template": "_Templates/일일기록 일간 노트.md"
}
```

이 조합의 좋은 점은 “노트를 자동 생성한다”보다 “생성 규칙을 예측 가능하게 만든다”에 있다. `calendar`는 여기서 날짜 클릭과 이동을 도와주는 보조 유틸에 가깝다. 편하긴 하지만, 이 흐름의 핵심은 어디까지나 `Periodic Notes + Templater`다.

실제 일간 템플릿도 같은 방향이다.

```markdown
## 일기

## 일정

## 메모

## 에이전트 메모

## 사진
```

섹션 구조가 고정되면 사용자는 입력을 덜 고민하게 되고, 에이전트는 어디를 읽고 어디에 써야 하는지를 덜 틀리게 된다.

### Day Planner

`obsidian-day-planner`는 여기서 기록을 다시 실행 인터페이스로 바꾼다. 이 vault에서는 일간 노트의 `## 일정`을 표준 섹션으로 쓰고, Day Planner가 그 섹션을 읽어 타임라인으로 보여준다.

![Day Planner에 반영된 시간표](/assets/images/2026/04/14/obsidian-day-planner-timeline.png){: .align-center .width-300px }

내가 이 플러그인을 계속 남겨둔 이유는 AI 기능이 있어서가 아니다. 오히려 반대다. 에이전트는 markdown만 정리하면 되고, 화면 출력은 Day Planner가 맡는다. 그러면 사용자는 긴 노트 대신 “지금 무엇을 해야 하는가”를 바로 볼 수 있다.

## 실행 플러그인 구성

실행 쪽에서는 `Base Board`와 `Dataview`가 각각 시각화와 조회를 맡는다. 둘 다 데이터를 따로 소유하지 않고 note와 frontmatter 위에 읽기 레이어를 얹는다는 점이 핵심이다.

| 플러그인 | 기본 역할 | 내가 쓰는 방식 | 에이전트 입장에서 좋은 점 |
| --- | --- | --- | --- |
| `base-board` | markdown과 frontmatter를 칸반처럼 보여준다 | `06_TODO` 폴더의 note를 `status` 값 기준 보드로 묶는다 | 보드 UI를 직접 조작하지 않고 파일 속성만 맞추면 된다 |
| `dataview` | 조건 조회와 집계를 보강한다 | TODO, 일정, 메모를 속성 조건으로 다시 묶어 본다 | note를 authoritative copy로 유지한 채 다양한 읽기 뷰를 만들 수 있다 |

### Base Board

실행 흐름은 UI가 데이터를 소유하지 않을 때 오래 간다. 이 점에서 `Base Board`는 생각보다 중요한 플러그인이다. 보드를 따로 관리하는 데이터베이스가 아니라, markdown 파일과 frontmatter를 읽어 칸반 뷰를 얹는 레이어이기 때문이다.

![Base Board로 렌더링한 실제 TODO 보드 화면](/assets/images/2026/04/21/obsidian-base-board.png){: .align-center }

실제 `.base` 파일을 보면 보드가 무엇을 읽고 어떻게 묶는지 바로 드러난다.

```yaml
filters:
  and:
    - file.inFolder("06_TODO")
    - status != null
views:
  - type: kanban
    name: TODO 칸반
    groupBy:
      property: status
    boardColumns:
      - 시작 전
      - 진행 중
      - 완료
```

즉 카드 이동의 본질은 보드를 움직이는 일이 아니라, 결국 `status` 같은 frontmatter 값을 바꾸는 일이다. 그래서 에이전트도 보드 UI 자체를 조작하는 쪽보다 markdown 파일과 속성 값을 직접 다루는 쪽이 더 안정적이다.

TODO 템플릿도 이 원칙을 그대로 따른다.

```yaml
---
tags:
  - <% tp.file.folder(true).split("/").pop() %>
date: <% tp.date.now("YYYY-MM-DD") %>
scheduled_date:
completed_date:
protected: false
status: 시작 전
important: false
kanban_order: 0
---
```

이렇게 보면 `Base Board`의 장점은 예쁜 칸반이 아니라, “파일이 곧 데이터”라는 사실을 깨지 않는 데 있다.

### Dataview

`Dataview`는 여기서 조회 레이어를 보강한다. Bases만으로는 어려운 동적 목록, 조건식, 집계, 역링크 기반 조회를 메워 주기 때문이다. 실제로는 특정 폴더의 TODO를 날짜순으로 다시 모아 보거나, 속성 기준으로 조건 목록을 만들 때 이 플러그인이 들어간다. 중요한 건 Dataview도 데이터 소유층이 아니라는 점이다. 실제 정보는 여전히 note와 frontmatter에 있고, Dataview는 그 위에 더 유연한 읽기 방법을 얹는다.

## 정리 플러그인 구성

정리 쪽에서는 `obsidian-custom-attachment-location` 하나가 훨씬 큰 역할을 한다. 눈에 가장 덜 띄는 플러그인이지만, 에이전트 운영에서 실패를 가장 많이 줄이는 레이어이기도 하다.

| 플러그인 | 기본 역할 | 내가 쓰는 방식 | 에이전트 입장에서 좋은 점 |
| --- | --- | --- | --- |
| `obsidian-custom-attachment-location` | 첨부 파일 경로와 이름을 자동 정리한다 | 붙여넣는 순간 `_attachments/YYYY/MM/DD/` 규칙으로 저장한다 | loose file과 임시 이름이 줄어들어 이미지 처리 실패가 크게 줄어든다 |

### Custom Attachment Location

`obsidian-custom-attachment-location`은 붙여넣는 순간 첨부 파일 경로를 `_attachments/YYYY/MM/DD/` 규칙에 맞춰 주고, 이 단순한 자동화가 생각보다 큰 차이를 만든다.

실제 설정은 거의 그대로 이렇게 잡혀 있다.

```json
{
  "attachmentFolderPath": "_attachments/${date:{momentJsFormat:'YYYY'}}/${date:{momentJsFormat:'MM'}}/${date:{momentJsFormat:'DD'}}",
  "attachmentRenameMode": "All",
  "shouldHandleRenames": true,
  "generatedAttachmentFileName": "img-${date:{momentJsFormat:'YYYYMMDD-HHmmssSSS'}}"
}
```

여기에 첨부 운영 플레이북이 한 번 더 기준을 잡아 준다.

```markdown
- 기본 저장 위치는 `_attachments/YYYY/MM/DD/`다.
- `_attachments/` 루트에 첨부 파일을 직접 두지 않는다.
- 파일명은 이미지 내용을 설명하는 짧은 한국어 캡션으로 정한다.
- 옮기거나 이름을 바꿨다면 해당 파일을 참조하는 링크도 함께 갱신한다.
```

이 흐름이 중요한 이유는, 에이전트가 이미지를 다룰 때 “이 파일이 왜 여기 있지?” 같은 의문을 줄여 주기 때문이다. 경로와 이름 규칙이 고정돼 있으면 첨부 정리는 플레이북 문제로 내려가고, 매번 즉흥적으로 판단할 일이 크게 줄어든다.

## 에이전트와의 결합

에이전트는 플러그인을 직접 이해해서 똑똑해지는 게 아니라, 플러그인이 만들어 준 고정된 규칙 위에서 덜 틀리게 된다. 이번 시리즈에서 계속 등장한 `daily-note-followup`가 좋은 예다.

로컬 스킬은 작업 절차를 캡슐화하지만, 그 스킬이 매번 제멋대로 파일을 만들지는 않는다. 실제로는 아래처럼 canonical reference를 먼저 읽고 움직인다.

```markdown
- `00_agent_docs/플레이북/Base Board 작업 지침.md`
- `00_agent_docs/플레이북/일일기록 일정 작성 플레이북.md`
- `00_agent_docs/플레이북/개인 정보 업데이트 플레이북.md`
- `00_agent_docs/플레이북/13_my 업데이트 검증 체크리스트.md`
```

여기서 플러그인 조합의 의미가 분명해진다.

| 플러그인 조합                                | 실제 결과                                          | 에이전트 입장에서 좋은 점                                   |
| -------------------------------------------- | -------------------------------------------------- | ----------------------------------------------------------- |
| `Periodic Notes` + `Templater`               | 날짜별 노트가 같은 경로와 같은 섹션으로 생성된다   | 파일명, 경로, 기본 구조를 매번 추측하지 않아도 된다         |
| `Day Planner` + `## 일정` 규칙               | 같은 텍스트가 바로 실행 가능한 타임라인으로 보인다 | 에이전트는 markdown만 정리하고, UI 출력은 플러그인이 맡는다 |
| `Base Board` + TODO 템플릿                   | TODO 상태가 frontmatter로 고정된다                 | 보드 조작 대신 파일 속성만 정확히 맞추면 된다               |
| `Custom Attachment Location` + 첨부 플레이북 | 첨부가 날짜 경로와 고정 규칙으로 정리된다          | 이미지 이동, rename, 링크 보정에서 실패가 줄어든다          |
| 전역 Obsidian 스킬 + 로컬 스킬               | 기본 앱 조작 능력과 vault 맞춤 절차가 분리된다     | 공용 능력과 개인 규칙을 한 층에 섞지 않아도 된다            |

결국 전역 스킬은 Obsidian과 상호작용하는 기본 능력을 주고, 로컬 스킬은 내 Vault 규칙에 맞는 작업 절차를 얹는다. 이 둘이 잘 겹치려면, 그 사이에 있는 플러그인과 템플릿 규칙이 먼저 안정돼 있어야 한다.

## 운영 포인트

좋은 플러그인 구성이란 기능이 많은 구성이 아니라 authoritative copy가 분명한 구성이다. 실제로 오래 쓰다 보면 기능보다 “무엇을 기준 문서로 삼을 것인가”가 더 중요해진다.

| 운영 포인트                | 자주 생기는 문제                                           | 현재 운영 방식                                                           |
| -------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------ |
| Day Planner                | `## 일정` 안의 링크가 재직렬화 과정에서 깨질 수 있다       | 일정 섹션은 plain text로 유지하고, 링크는 `## 메모`나 관련 노트로 보낸다 |
| Base Board                 | 보드를 데이터베이스처럼 오해하거나 상태 값을 추측하기 쉽다 | `.base`와 플레이북을 먼저 읽고, 실제 파일과 frontmatter를 직접 수정한다  |
| Templater + Periodic Notes | 생성 경로와 템플릿 매핑이 어긋나면 노트 구조가 흔들린다    | 날짜 경로, regex, 템플릿 파일을 같이 맞춘다                              |
| Custom Attachment Location | 루트에 loose file이 쌓이거나 임시 이름이 남기 쉽다         | `_attachments/YYYY/MM/DD/` 규칙과 첨부 플레이북을 같이 유지한다          |
| Dataview                   | 조회 레이어를 데이터 소유층처럼 쓰고 싶어진다              | Dataview는 읽기와 집계만 맡기고, 실제 데이터는 note/frontmatter에 둔다   |

지금 내 기준에서 “핵심 플러그인”은 예쁜 기능을 추가하는 것보다, 에이전트가 덜 틀리고 사용자는 덜 헤매게 만드는 쪽에 더 가깝다.

## 마무리

내가 Obsidian을 메모 앱이 아니라 작업 공간으로 계속 쓰는 이유는, 모든 걸 바닥부터 만들지 않아도 되기 때문이다. `Periodic Notes`, `Templater`, `Day Planner`, `Base Board`, `Dataview`, `Custom Attachment Location`처럼 각자 작은 역할을 맡은 플러그인들이 연결되면, 기록은 기록대로, 실행은 실행대로, 정리는 정리대로 서로 다른 책임을 분담하게 된다.

그 위에서 에이전트는 갑자기 똑똑해지는 게 아니라, 더 적게 틀리게 된다. 이 시리즈의 5편에서는 Obsidian 기본 능력을 주는 전역 스킬을 왜 전역에 두고, 내 개인 비서용 절차는 왜 로컬 스킬로 분리했는지 정리해 보려 한다.
