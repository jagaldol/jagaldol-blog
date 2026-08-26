---
layout: single
title: "GitHub HTTPS 인증 원리 - Credential Helper와 gh auth login 사용법"
date: 2026-05-04 20:00:00 +0900
categories: development
header:
  teaser: /assets/images/2026/05/04/github-https-auth-hero.png
---

GitHub 저장소를 HTTPS로 clone한 뒤 `git push`에서 비밀번호를 요구받는다면 계정 비밀번호를 넣어도 인증되지 않는다. GitHub은 2021년 8월부터 Git 작업의 비밀번호 인증을 받지 않는다.

![터미널과 원격 저장소 사이에서 OAuth 자격증명을 안전하게 공급하는 credential helper](/assets/images/2026/05/04/github-https-auth-hero.png){: .align-center }

대신 Personal Access Token(PAT), OAuth token 또는 SSH key를 사용해야 한다. HTTPS를 유지하면서 브라우저로 로그인하고 싶다면 GitHub CLI인 `gh`가 가장 간단하다. 이 글에서는 그 과정에서 Git의 credential helper가 어떤 역할을 하는지부터 살펴본다.

## Git과 로그인 도구는 역할이 다르다

Git은 HTTPS 서버가 자격증명을 요구하면 설정된 credential helper에 사용자 이름과 비밀번호 또는 token을 요청한다. helper는 운영체제의 자격증명 저장소나 별도의 로그인 도구에서 값을 가져와 Git에 돌려준다.

```text
git pull / push
        ↓
Git credential helper 호출
        ↓
저장된 OAuth token 반환
        ↓
GitHub HTTPS 인증
```

Git 자체가 브라우저 OAuth를 수행하는 것은 아니다. `gh`, Git Credential Manager(GCM), macOS의 `osxkeychain` 같은 도구가 helper 또는 저장소 역할을 맡는다.

## gh와 GCM은 별개의 credential helper다

`gh`가 GCM을 내장하는 것은 아니다. 두 도구는 Git의 credential helper 규격을 각각 구현한다.

| 항목 | GitHub CLI (`gh`) | Git Credential Manager |
| --- | --- | --- |
| 주 용도 | GitHub CLI와 GitHub 인증을 함께 관리 | Git HTTPS 자격증명을 전담 관리 |
| 로그인 시작 | `gh auth login` | 인증이 필요한 Git 명령에서 시작 |
| Git 연결 | `gh auth setup-git` | `credential.helper=manager` |
| 자격증명 보관 | 가능한 경우 OS credential store | OS credential store |
| 적합한 경우 | GitHub를 쓰고 이미 `gh`를 사용하는 환경 | 전용 Git 인증 관리나 별도 운영 정책이 필요한 환경 |

GitHub만 사용하고 `gh`도 설치할 계획이라면 GCM을 추가로 설치할 이유가 크지 않다. 반대로 조직에서 GCM을 표준으로 배포하거나 GitHub CLI와 인증 수명주기를 분리하려면 GCM이 자연스럽다.

## 가장 간단한 설정: gh auth login

macOS에서 Homebrew를 사용한다면 먼저 GitHub CLI를 설치한다.

```bash
brew install gh
```

브라우저 OAuth 로그인과 HTTPS 프로토콜을 선택한다.

```bash
gh auth login --web --git-protocol https
```

대화형 과정에서 `Authenticate Git with your GitHub credentials?`가 나오면 `Yes`를 선택한다. 로그인 token은 가능한 경우 시스템 credential store에 보관된다.

이미 로그인했거나 Git helper 설정만 확실히 맞추고 싶다면 다음 명령을 실행한다.

```bash
gh auth setup-git
```

이 명령은 인증된 GitHub host에 대해 `gh auth git-credential`을 Git credential helper로 등록한다. 한 번 설정하면 `git pull`, `git fetch`, `git push`가 활성 `gh` 계정의 token을 사용한다.

## 설정 확인

먼저 `gh` 로그인 상태와 Git 프로토콜을 확인한다.

```bash
gh auth status
```

Git이 GitHub HTTPS 요청에 실제로 사용할 helper는 다음 명령으로 확인한다.

```bash
git config --show-origin \
  --get-all credential.https://github.com.helper
```

`gh auth setup-git`이 적용됐다면 설치 경로에 따라 다음과 비슷한 항목이 보인다.

```text
!/path/to/gh auth git-credential
```

GitHub 저장소에 접근할 수 있는지 읽기 전용으로 확인하려면 다음처럼 `HEAD`만 조회한다.

```bash
git ls-remote https://github.com/OWNER/REPOSITORY.git HEAD
```

private 저장소에서도 commit hash가 출력되면 HTTPS 인증이 정상이다.

## 여러 GitHub 계정을 사용할 때

`gh`에는 같은 host의 계정을 여러 개 등록할 수 있다. 현재 계정을 확인하고 전환하는 명령은 다음과 같다.

```bash
gh auth status
gh auth switch --user ACCOUNT_NAME
```

Git의 GitHub 전용 helper가 `gh auth git-credential`로 설정돼 있으면 활성 계정의 자격증명이 Git 작업에도 사용된다. 저장소마다 다른 계정을 고정해야 한다면 계정 전환만으로 운영하기보다 저장소별 URL이나 credential context까지 별도로 설계하는 편이 안전하다.

## 인증이 여전히 꼬일 때

`gh auth login` 성공은 Git helper 설정까지 항상 보장하지 않는다. 로그인 과정에서 Git 인증 연결을 거절했거나 기존 helper가 남아 있다면 Git과 `gh`가 서로 다른 자격증명을 사용할 수 있다.

이때는 다음 순서로 확인한다.

1. `gh auth status`에서 원하는 계정이 활성 상태인지 본다.
2. `gh auth setup-git`으로 helper를 다시 등록한다.
3. `git config --show-origin --get-all credential.helper`와 GitHub 전용 helper를 함께 확인한다.
4. `git ls-remote`로 Git HTTPS 인증을 직접 시험한다.

token 문자열을 셸 설정이나 저장소 파일에 직접 적는 방식은 피한다. 인증 방식을 바꿔도 기존 commit과 branch에는 영향이 없으므로 문제 해결을 위해 저장소를 다시 clone할 필요도 없다.

## 참고 자료

- [GitHub CLI - gh auth login](https://cli.github.com/manual/gh_auth_login)
- [GitHub CLI - gh auth setup-git](https://cli.github.com/manual/gh_auth_setup-git)
- [GitHub CLI - gh auth switch](https://cli.github.com/manual/gh_auth_switch)
- [Git - gitcredentials](https://git-scm.com/docs/gitcredentials)
- [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager)
- [GitHub Docs - About authentication to GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-authentication-to-github)
