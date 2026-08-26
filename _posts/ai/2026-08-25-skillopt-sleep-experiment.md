---
layout: single
title: "SkillOpt-Sleep 실제 평가 - 점수는 올랐지만 스킬은 좋아지지 않았다"
date: 2026-08-25 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/08/25/skillopt-sleep-experiment-hero.png
---

SkillOpt-Sleep을 실제 코딩 에이전트의 작업형 skill에 연결해 전체 사이클을 돌려봤다. 세션 수집부터 후보 규칙 생성, held-out gate, staging과 수동 적용까지 pipeline은 정상 작동했다.

![점수 상승과 회귀 경고를 함께 저울질해 후보 스킬을 승인하지 않는 실험](/assets/images/2026/08/25/skillopt-sleep-experiment-hero.png){: .align-center }

점수도 0.4829에서 0.5193으로 올랐다. 그러나 task별 결과와 실제 응답을 읽어보니 품질 개선으로 인정할 수 없었다. 최종 후보는 제거하고 원래 skill을 유지했다.

## 실행 범위

| 항목 | 값 |
| --- | ---: |
| 수집한 세션 | 4개 |
| 재구성된 task | 8개 |
| Train / validation | 1개 / 7개 |
| 실행 시간 | 약 52분 |
| 사용 token | 72,138 |

대상은 파일, terminal, browser 등 외부 상태를 다루는 범용 작업 skill이었다. 민감한 원본 요청과 계정 정보는 평가에서 필요한 구조만 남기고 공개하지 않는다.

```text
session harvest
→ task mining
→ baseline replay
→ reflection과 bounded edit
→ candidate replay
→ held-out gate
→ staging
→ manual review
```

## Train 점수는 0.42에서 0.98로 올랐다

Train task는 주어진 기술 자료를 읽고 수식을 실제로 설명하는 문제였다. Baseline은 설명 대신 자료를 보내달라고 요청해 낮은 점수를 받았다.

Optimizer는 skill에 두 규칙을 추가했다.

1. 기호 의미, 자연어 해석, 간단한 예시, 문맥 속 역할 순으로 설명한다.
2. 정보가 불완전해도 가능한 설명을 먼저 하고 가정을 밝힌다.

규칙 자체는 합리적이다. Candidate의 train score도 0.42에서 0.98로 뛰었다. 문제는 candidate 역시 실제 수식을 풀지 않고 “자료를 보내 달라”는 요청과 설명 순서만 제시했다는 점이다.

Judge는 rubric에 나온 형식적 표현이 들어갔다는 이유로 거의 만점을 줬다. 측정값은 좋아졌지만 원래 실패는 해결되지 않은 false positive였다.

## Validation 평균은 왜 통과했나

7개 held-out task의 결과는 다음과 같았다.

| 결과 | Task 수 |
| --- | ---: |
| 개선 | 2 |
| 회귀 | 2 |
| 동일 | 3 |

| 평균 | Baseline | Candidate |
| --- | ---: | ---: |
| Mixed score | 0.4829 | 0.5193 |

실행 설정은 `gate_no_regression: false`였다. 개별 task 두 개가 나빠져도 평균이 오르면 후보가 gate를 통과한다. 추가 규칙과 직접 관련이 없는 task의 점수도 오르내렸기 때문에 실제 skill 변화보다 생성 응답과 LLM judge의 표본 변동으로 보는 편이 타당했다.

`gate_no_regression: true`였다면 이 후보는 막을 수 있다. 그래도 잘못된 judge를 더 엄격하게 적용할 뿐, 평가가 실제 작업 성공을 측정하게 되는 것은 아니다.

## 기본 judge는 최종 응답만 봤다

이번 task는 모두 정답 문자열이나 외부 verifier가 없는 rubric 평가였다.

```text
replay의 최종 response
+ task rubric
→ LLM judge
→ soft score 0..1
```

Codex가 답변을 만들면서 어떤 파일을 읽었는지, 명령이 성공했는지, 브라우저의 실제 상태가 바뀌었는지는 judge 입력에 포함되지 않았다. 특정 tool 호출 여부를 확인하는 rule도 설정하지 않았다.

따라서 다음 두 응답을 구분하기 어렵다.

```text
A: 실제 파일을 수정하고 검증한 뒤 완료를 보고
B: 작업하지 않았지만 그럴듯하게 완료했다고 설명
```

최종 문장만 보면 B가 더 높은 점수를 받을 수도 있다. 작업형 agent를 평가할 때 치명적인 경계다.

## Pipeline에서 확인된 것

평가가 무의미했던 것은 아니다. 다음 배관은 실제 환경에서 확인됐다.

- 최근 세션을 읽어 재사용 가능한 task로 압축했다.
- Train task에서 작은 add/delete/replace 편집을 만들었다.
- Validation replay와 gate가 실행됐다.
- `adopt` 전까지 실제 skill은 바뀌지 않았다.
- 적용 전 backup과 기계 소유 `LEARNED` block 경계가 작동했다.

즉 **개선안을 만들고 안전하게 staging하는 시스템**은 작동했다. 확인하지 못한 것은 **그 개선안이 실제 작업 능력을 높였는가**다.

## 작업형 skill에는 무엇이 더 필요한가

결과가 응답 밖에 있다면 그 상태를 직접 검사해야 한다.

| Skill 종류 | 평가 신호 |
| --- | --- |
| 코드 수정 | Fixture repo, test, typecheck, diff |
| CLI 절차 | Exit code, 실행 순서, 생성 파일 |
| 브라우저 작업 | 실제 destination과 field 상태 재조회 |
| 문서 생성 | 구조 검사와 rendering |
| Spreadsheet | 셀 값, 수식, 서식 비교 |

Task마다 격리된 환경을 새로 만들고, agent가 남긴 실제 산출물을 독립 verifier로 채점해야 한다. SkillOpt 논문의 ALFWorld와 SpreadsheetBench도 simulator state와 workbook cell을 비교했기 때문에 작업형 task를 다룰 수 있었다.

## 최종 판단

후보 규칙은 최종적으로 유지하지 않았다. 평균 점수는 올랐지만 train에 명확한 false positive가 있었고, validation에서도 회귀 두 개가 허용됐으며, 외부 상태를 확인할 verifier가 없었다.

SkillOpt-Sleep의 기본 경로는 답변 형식, 분류, 추출, 텍스트 QA처럼 최종 문자열이 곧 결과인 skill에 더 잘 맞는다. 파일·terminal·browser 작업은 먼저 fixture와 verifier를 만든 뒤 최적화해야 한다. 자동 개선에서 가장 먼저 설계할 것은 optimizer가 아니라 채점 방식이다.

## 참고 자료

- [SkillOpt 논문 리뷰](/papers/skillopt/)
- [SkillOpt 사용 가이드](/ai/skillopt-guide/)
- [SkillOpt-Sleep 문서](https://github.com/microsoft/SkillOpt/blob/main/docs/sleep/README.md)
