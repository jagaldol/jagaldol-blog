---
layout: single
title: "[Git] commit 날짜 / 시간 변경 방법(feat. github 잔디)"
date: 2025-07-01 23:05:00 +0900
last_modified_at: 2025-07-02 08:12:00 +0900
categories: development
---

`Github`에는 커밋이나 `Issue`, `Pull Request` 등 `contribution`한 기록이 그래프로 남는다. 이 `contribution graph`를 한국 사람들은 github 잔디라고 부르며 성실함과 꾸준함을 드러내기 위해 사용하며, 또한 하나의 일일목표로 삼는 사람이 많다.

![github graph](/assets/images/2025/07/01/profile-green-animate.svg)

> 나도 그 중 한 명으로 너무 바쁜 일정이 있으면 군데군데 구멍이 나긴하지만 꾸준히 1일 1 기여를 목표로 하고 있다.

하지만 매일 커밋을 하기란 쉽지 않고 밤 11시부터 개발을 시작했는데 커밋을 하니 12시가 넘는 경우가 종종 있다. 이런 경우 잔디에 구멍이 나면 억울 하지 않을까?

이런 억울함을 달래기 위한 **tricky한 명령어를 소개**한다.

## 명령어

일단 먼저 커밋을 한 뒤, 해당 커밋의 시간을 `git commit --amend` 명령을 통해 덮어씌우게 된다.

1시간 전:

```shell
GIT_AUTHOR_DATE="$(date -d '1 hours ago')" GIT_COMMITTER_DATE="$(date -d '1 hours ago')" git commit --amend --no-edit --date "$(date -d '1 hours ago')"
```

하루 전:

```shell
GIT_AUTHOR_DATE="$(date -d '1 days ago')" GIT_COMMITTER_DATE="$(date -d '1 days ago')" git commit --amend --no-edit --date "$(date -d '1 days ago')"
```

위 명령어를 통해 `committer` 시간 뿐만아니라 `author` 시간까지 변경되어 완벽하게 커밋 시간을 변경가능하다.

하지만 해당 명령어는 linux 계열에서만 가능한 명령어로 윈도우의 CMD에서는 동작하지 않는다. 따라서 꼭 **Git Bash를 켜서 명령어를 입력**해야한다!
