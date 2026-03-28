---
layout: single
title: "macOS Codex 앱 창 최소 너비가 안 줄어들 때 해결방법"
date: 2026-03-28 15:08:25 +0900
categories: development
---

macOS용 `Codex` 데스크톱 앱을 쓰다 보면 창 가로폭이 어느 지점 아래로 더 줄어들지 않는 경우가 있다.
사이드바를 닫아도 여전히 넓게 고정돼 있다면, 단순한 레이아웃 문제가 아니라 내부 상태값이 꼬인 경우일 수 있다.
내가 겪은 경우에는 창이 정확히 `1200px` 근처 아래로 더 줄어들지 않았고, 사이드바를 닫아도 그 최소 너비는 그대로였다.

비슷한 증상은 [openai/codex 이슈 #14010](https://github.com/openai/codex/issues/14010)에도 등록돼 있다.

## 증상

아래는 문제가 발생했을 때의 화면이다.

![Codex 창 최소 너비 문제 해결 전 - 사이드바 열림](/assets/images/2026/03/28/codex-app-min-width-before-sidebar.png)

- 스레드 목록 사이드바가 열린 상태에서 창이 넓게 고정돼 있다.

![Codex 창 최소 너비 문제 해결 전 - 사이드바 닫힘](/assets/images/2026/03/28/codex-app-min-width-before-closed-sidebar.png)

- 사이드바를 닫아도 창 너비가 충분히 줄어들지 않는다.
- 정확히는 창이 `1200px` 근처 아래로 더 줄어들지 않았고, 사이드바를 닫아도 최소로 줄일 수 있는 너비는 변하지 않았다.
- 즉 패널이 열려 있어서 넓어 보이는 문제가 아니라, 최소 너비 자체가 비정상적으로 크게 잡힌 상태에 가깝다.

## 원인

문제의 핵심은 `~/.codex/.codex-global-state.json` 안의 `electron-persisted-atom-state.review-open` 값이었다.
리뷰 패널이 실제로는 닫혀 있어도 이 값이 `true`로 stale하게 남아 있으면, 앱이 계속 `리뷰가 열린 창`으로 판단하면서 최소 너비를 크게 잡는다.

확인했을 때 메인 창 최소 크기는 일반 상태에서 `800 x 600`, `review-open: true` 상태에서는 `1200 x 800`으로 잡혀 있었다.
다만 macOS Retina 환경에서는 스크린샷의 물리 픽셀보다 `pt` 기준 최소 크기로 이해하는 편이 맞다.

## 해결 방법

> `Codex`가 실행 중이면 종료 시 상태 파일을 다시 덮어쓸 수 있다. 반드시 앱을 완전히 종료한 뒤 수정해야 한다.

1. `Codex`를 `Cmd+Q`로 완전히 종료한다.
2. 필요하면 상태 파일을 먼저 백업한다.

```bash
cp ~/.codex/.codex-global-state.json ~/.codex/.codex-global-state.json.bak
```

3. 아래 명령으로 `review-open` 값을 `false`로 바꾼다.

```bash
perl -0pi -e 's/"review-open":true/"review-open":false/g' ~/.codex/.codex-global-state.json
```

4. 값이 잘 바뀌었는지 확인한다.

```bash
jq '."electron-persisted-atom-state"["review-open"]' ~/.codex/.codex-global-state.json
```

5. `false`가 나오면 `Codex`를 다시 실행한다.

문제가 해결되면 같은 창이 훨씬 좁은 폭까지 줄어든다.

![Codex 창 최소 너비 문제 해결 후](/assets/images/2026/03/28/codex-app-min-width-after-fix.png)

## 그래도 정확히 반 화면까지는 안 갈 수 있는 이유

이 방법은 `stale 상태 때문에 비정상적으로 넓게 고정되는 문제`를 푸는 방법이지, 모든 환경에서 무조건 반 화면 이하까지 줄여 주는 방법은 아니다.

`Codex` 메인 창 자체에 기본 최소 너비가 있고, 현재 디스플레이의 실제 작업 영역 절반이 그보다 더 좁으면 반 화면 배치는 불가능하다.

예를 들어 내가 확인한 환경은 다음과 같았다.

- `MacBook Pro 14형 (Apple M4)`
- 디스플레이 기본 해상도 사용
- 내장 디스플레이 작업 영역 폭 약 `1512pt`
- 화면 절반 약 `756pt`

이 환경에서는 `review-open` 문제를 해결해도 기본 최소 너비 `800`보다 화면 절반 `756`이 더 좁기 때문에, 기본 해상도에서는 반 화면보다 약간 더 넓은 수준까지만 줄어드는 것이 정상이다.

## 다시 같은 문제가 생기면

- `~/.codex/.codex-global-state.json`의 `review-open` 값이 다시 `true`로 돌아갔는지 먼저 확인한다.
- 값이 `true`인데 리뷰 패널이 실제로 닫혀 있다면 같은 종류의 stale state 문제일 가능성이 높다.

화면이 넓게 고정되는 현상 때문에 `Codex`를 화면분할로 쓰기 어려웠다면, 먼저 이 상태값부터 확인해보면 된다.
