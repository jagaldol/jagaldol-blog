---
layout: single
title: "tmux에서 Claude Code를 편하게 쓰는 두 가지 설정 - 화면 깜박임과 이미지 붙여넣기"
date: 2026-05-10 20:00:00 +0900
categories: development
header:
  teaser: /assets/images/2026/05/10/claude-terminal-tips-hero.png
---

Claude Code를 tmux 안에서 오래 실행하면 두 군데에서 자주 막힌다. 응답이 갱신될 때 화면이 아래로 튀고, macOS 클립보드의 이미지는 평소처럼 `Cmd+V`를 눌러도 첨부되지 않는다.

![터미널의 스크롤과 텍스트 이미지 클립보드 파일 입력을 정리한 작업대](/assets/images/2026/05/10/claude-terminal-tips-hero.png){: .align-center }

화면 문제는 환경 변수 하나로 줄일 수 있고, 이미지와 텍스트는 붙여넣기 키가 다르다는 점만 알면 된다.

## 응답 중 화면이 계속 아래로 튈 때

Claude Code TUI와 tmux가 화면을 갱신하는 방식이 맞물리면, 위쪽 출력을 읽는 도중 새 출력에 밀려 맨 아래로 돌아가는 일이 생긴다. 이때 `CLAUDE_CODE_NO_FLICKER`를 켠다.

```bash
export CLAUDE_CODE_NO_FLICKER=1
```

계속 사용할 설정이라면 `~/.zshrc`에 넣는다.

```bash
# Claude Code: tmux 출력 점프와 깜박임 줄이기
export CLAUDE_CODE_NO_FLICKER=1
```

현재 셸에 적용한 뒤 Claude Code를 다시 실행한다.

```bash
source ~/.zshrc
echo "$CLAUDE_CODE_NO_FLICKER"
claude
```

`echo` 결과가 `1`이면 새로 실행한 Claude Code가 설정을 읽는다. 이미 실행 중인 프로세스에는 뒤늦게 반영되지 않는다.

### tmux에서는 언제 반영되나

환경 변수는 셸이 시작될 때 상속된다.

- 새 tmux window나 pane: 새 셸이 뜨므로 자동 반영
- 기존 pane: `source ~/.zshrc` 후 Claude Code 재실행
- 이미 실행 중인 Claude Code: 종료 후 다시 실행

설정은 응답을 줄이거나 출력을 멈추지 않는다. TUI가 화면을 그리는 방식을 바꿔 읽는 동안의 점프를 줄인다.

## macOS에서 텍스트와 이미지는 키가 다르다

Claude Code TUI는 macOS 클립보드 입력을 다음처럼 나눈다.

| 키 | 입력 |
| --- | --- |
| `Cmd+V` | 텍스트 붙여넣기 |
| `Ctrl+V` | 클립보드 이미지 첨부 |

이미지를 복사한 뒤 `Ctrl+V`를 누르면 입력창에 `[Image #1]` 같은 첨부 표시가 생긴다. tmux 안에서도 이 키 입력은 Claude Code로 전달된다.

`Cmd+V`는 macOS에서 익숙한 붙여넣기 키지만, Claude Code 안에서는 텍스트용이다. 이미지가 아무 반응 없이 사라지는 것처럼 보인다면 먼저 `Ctrl+V`를 확인한다.

### 이미지 키가 동작하지 않을 때

이미지 붙여넣기 동작은 Claude Code 버전과 터미널 조합에 영향을 받을 수 있다. `Ctrl+V`가 무반응이면 파일을 저장하고 경로와 함께 읽어 달라고 요청하는 방법이 가장 단순하다.

```text
/tmp/screenshot.png 파일을 열어서 오류 메시지를 확인해줘.
```

경로만 입력하면 일반 텍스트로 전달될 수 있으므로, 이미지를 읽으라는 요청을 같이 적는다. 오래된 글에 나오는 `/image` 명령이나 복잡한 tmux 우회 스크립트부터 적용할 필요는 없다.

## 점검 순서

두 문제가 섞여 보일 때는 다음 순서로 확인하면 된다.

1. `echo "$CLAUDE_CODE_NO_FLICKER"`가 `1`인지 확인한다.
2. Claude Code를 환경 변수 적용 이후에 다시 실행했는지 확인한다.
3. 텍스트는 `Cmd+V`, 이미지는 `Ctrl+V`로 구분한다.
4. 이미지 키가 깨졌다면 파일 경로를 명시해 읽게 한다.

작은 설정이지만 tmux에서 긴 에이전트 작업을 지켜볼 때 체감 차이가 크다.

## 참고 자료

- [Claude Code terminal configuration](https://docs.anthropic.com/en/docs/claude-code/terminal-config)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)
