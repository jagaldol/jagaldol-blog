---
layout: single
title: "SkillOpt 사용 가이드 - skill 훈련과 SkillOpt-Sleep의 차이"
date: 2026-07-24 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/07/24/skillopt-guide-hero.png
---

SkillOpt를 설치하면 두 가지 경로를 선택할 수 있다. 연구용 trainer는 benchmark를 준비해 skill 문서 전체를 훈련하고, SkillOpt-Sleep은 코딩 에이전트의 과거 세션에서 작은 개선 규칙을 제안한다.

![작업 재생과 편집 및 승인 단계를 거쳐 스킬을 개선하는 흐름](/assets/images/2026/07/24/skillopt-guide-hero.png){: .align-center }

두 도구는 같은 목적을 갖지만 검증 범위와 산출물이 다르다. 먼저 어떤 결과를 평가할 수 있는지 정한 뒤 경로를 골라야 한다.

## 두 실행 경로

| 경로 | 적합한 경우 | 결과 |
| --- | --- | --- |
| `skillopt-train`, `skillopt-eval` | 고정 benchmark와 verifier를 만들 수 있음 | 독립적인 `best_skill.md` |
| `skillopt-sleep` | 과거 세션에서 반복되는 응답 규칙을 개선 | 기존 파일의 `LEARNED` 블록 후보 |

연구용 trainer는 논문의 전체 최적화 루프를 사용한다. Sleep은 배포 환경에서 세션을 수집하고, 제안을 staging한 뒤 사용자가 `adopt`하는 companion tool이다. 논문 benchmark와 Sleep의 기본 LLM judge를 같은 검증 수준으로 보면 안 된다.

## 설치

Python 3.10 이상에서 PyPI 패키지를 설치할 수 있다.

```bash
python -m pip install skillopt
skillopt-sleep --help
```

내장 benchmark config, materializer, agent integration까지 사용하려면 저장소를 checkout한다.

```bash
git clone https://github.com/microsoft/SkillOpt.git
cd SkillOpt
python -m pip install -e .
```

Benchmark별 의존성은 extra로 분리돼 있다.

```bash
python -m pip install -e ".[searchqa]"
python -m pip install -e ".[alfworld]"
python -m pip install -e ".[webui]"
```

Backend credential을 `.env`에 적었다면 셸 환경으로 명시적으로 올려야 한다. 저장소에 credential 파일을 commit하지 않는다.

## 연구용 trainer로 훈련하기

SearchQA 예시에서는 먼저 데이터를 만든다.

```bash
python scripts/materialize_searchqa.py
```

Config의 주요 값은 일반적인 학습 용어와 대응한다.

```yaml
train:
  num_epochs: 4
  batch_size: 40
optimizer:
  learning_rate: 4
  lr_scheduler: cosine
  use_slow_update: true
  use_meta_skill: true
gradient:
  analyst_workers: 16
evaluation:
  use_gate: true
```

여기서 `learning_rate: 4`는 숫자 gradient의 크기가 아니라 한 step에서 허용할 문서 편집 수다.

```bash
python scripts/train.py \
  --config configs/searchqa/default.yaml \
  --out_root outputs/searchqa_first_run

python scripts/eval_only.py \
  --config configs/searchqa/default.yaml \
  --skill outputs/searchqa_first_run/best_skill.md \
  --split valid_unseen
```

최고 validation checkpoint는 `best_skill.md`로 남는다. Test split은 학습 중 선택에 쓰지 않고 최종 평가에만 사용해야 한다.

## SkillOpt-Sleep의 한 사이클

Sleep은 다음 순서로 움직인다.

```text
세션 harvest
→ 반복 task mining
→ 격리된 replay
→ bounded edit 생성
→ held-out gate
→ staging
→ 사용자 adopt
```

가장 안전한 시작은 mock backend의 dry run이다.

```bash
skillopt-sleep dry-run
skillopt-sleep run
skillopt-sleep status
skillopt-sleep adopt
```

`dry-run`은 수집과 replay 보고서만 만들고 제안을 staging하지 않는다. `run`도 실제 skill을 바로 바꾸지는 않는다. `adopt`할 때 기존 파일을 백업한 뒤 staged proposal을 반영한다.

Codex 세션을 실제 backend로 평가하려면 다음처럼 실행할 수 있다.

```bash
python -m skillopt_sleep run \
  --project "$(pwd)" \
  --source codex \
  --backend codex \
  --max-sessions 5 \
  --max-tasks 3 \
  --progress
```

세션 transcript에는 민감한 문자열이 들어 있을 수 있다. 자동 마스킹만 믿고 곧바로 provider replay를 돌리기보다 harvest 결과와 task 파일을 먼저 검수하는 편이 안전하다.

## Sleep이 실제로 채점하는 것

기본 rubric 경로는 과거의 다중 turn 세션을 그대로 다시 실행하지 않는다. Miner가 사용자 의도, 최소 문맥, 평가 기준을 하나의 `TaskRecord`로 압축한다.

```text
TaskRecord + 현재 skill
→ backend의 단일 replay
→ 최종 response 문자열
→ LLM judge가 rubric과 비교
```

Backend가 중간에 파일과 셸을 사용하더라도 일반 rubric judge는 최종 문자열만 본다. 다음 결과는 확인하지 못한다.

- 파일이 실제로 올바르게 바뀌었는가
- 명령의 exit code가 0이었는가
- Git diff가 요구 범위 안에 있는가
- 브라우저에서 외부 상태가 바뀌었는가
- 생성한 문서나 workbook이 제대로 렌더링되는가

특정 tool을 호출했는지 검사하는 rule을 추가할 수는 있지만, 호출 결과까지 맞았다는 보장은 아니다.

## 어떤 skill에 잘 맞나

기본 Sleep 경로가 잘 맞는 대상은 최종 텍스트 자체가 산출물인 작업이다.

- 답변 형식과 길이
- 분류와 정보 추출
- 고객 응대 말투
- 정답이 있는 텍스트 QA
- 설명 순서와 난이도

외부 상태가 결과인 작업에는 전용 benchmark가 필요하다.

| 작업 | 필요한 verifier 예시 |
| --- | --- |
| 코드 수정 | Target test, typecheck, Git diff |
| 파일 생성 | Schema, 내용, 경로 검사 |
| 터미널 절차 | 명령 순서, exit code, 산출물 |
| Spreadsheet | 셀 값, 수식, 서식 비교 |
| 브라우저 작업 | 실제 destination 재조회 |

`gate_no_regression=true`로 엄격하게 만들어도 judge 자체가 틀리면 측정 타당성은 생기지 않는다. 먼저 gate가 실제 성공을 재는지 확인해야 한다.

## 문서 소유권을 나누는 `LEARNED` 블록

Sleep은 기존 skill 전체를 다시 쓰지 않고 다음 marker 사이의 bullet만 관리한다.

```markdown
<!-- SKILLOPT-SLEEP:LEARNED START -->
## Learned preferences & procedures

- 검증을 통과한 작은 규칙
<!-- SKILLOPT-SLEEP:LEARNED END -->
```

사람이 관리하는 절차·예시·reference는 block 밖에 두고, 반복적으로 검증된 작은 선호만 안에 쌓는 구조가 적합하다. 여러 파일과 script가 하나의 계약을 이루는 skill이라면 Sleep 한 파일만으로 전체를 최적화할 수 없다.

실전 순서는 단순하다. `dry-run`으로 수집 범위와 task 품질을 확인하고, 작은 평가셋과 verifier를 준비한 뒤 `run`을 수행한다. 마지막으로 점수가 아니라 staged diff와 task별 회귀를 읽고 `adopt` 여부를 판단한다.

## 참고 자료

- [SkillOpt 논문 리뷰](/papers/skillopt/)
- [microsoft/SkillOpt](https://github.com/microsoft/SkillOpt)
- [SkillOpt-Sleep 문서](https://github.com/microsoft/SkillOpt/blob/main/docs/sleep/README.md)
- [Codex integration](https://github.com/microsoft/SkillOpt/blob/main/plugins/codex/README.md)
