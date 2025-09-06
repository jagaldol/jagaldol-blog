---
layout: single
title: "[카테캠 BE] Git/Github 소개 및 활용법(1) - git, github action(CI/CD), branch 전략"
date: 2023-04-25 00:43:00 +0900
last_modified_at: 2023-05-02 12:38:00 +0900
categories:
  - kakao tech campus
---

4월 24일에 진행한 실시간 세션 - Git/Github 소개 및 활용법입니다. 첫 시간이라 git 개념과 git 명령어들을 배웠습니다.

(5월 2일 추가) - 이론 파트 추가하였습니다! `git flow`, `github flow`, `gitlab flow`의 브랜치 전략을 다룹니다.

## git

- VCS (Version Control System)
- 빠른속도, 단순한 구조
- 분산형 저장소 지원
- 비선형적 개발(수천개의 브랜치) 가능

## git 장점

- 소스코드 주고받기 없이 동시작업이 가능
- 수정내용은 commit 단위로 관리, 배포 뿐 아니라 원하는 시점으로 Checkout 가능
- 새로운 기능 추가는 Branch로 개발, 성공적으로 개발이 완료되면 Merge하여 반영
- 인터넷이 연결되지 않아도 개발할 수 있음

## git GUI 종류

- git GUI
- sourcetree
- kraken
- smartGit

> 하지만 CLI를 알아야한다!!  
> window에서는 git bash 사용해서 shell 명령어 사용 가능

## commit

커밋 순간의 스냅샷(Blob과 Tree로 이루어져있다)

- Blob: 파일 하나의 내용에 대한 정보
- Tree: Blob이나 subtree의 메타데이터
  ![commit](/assets/images/2023/04/25/data-model-3.png)

## Conventional Commits

- commit의 제목은 commit을 설명하는 하나의 구나 절로 완성
- Importance of Capitalize(대문자)
- prefix 달기
  - feat: 기능 개발 관련
  - fix: 오류 개선 혹은 버그 패치
  - docs: 문서화 작업
  - test: test 관련
  - conf: 환경설정 관련
  - build: 빌드 관련
  - ci: Continuous Integration 관련

> ref: [https://www.conventionalcommits.org/ko/v1.0.0/](https://www.conventionalcommits.org/ko/v1.0.0/)

## git 명령어

- `git status`
- `git add`
- `git commit`
- `git branch`

## git commit

![git-commit](/assets/images/2023/04/25/git-commit.png)

- `git commit`
  - 첫 줄은 commit 제목
  - 세번째 줄은 commit description(둘째 줄에 enter 꼭 들어가야함.)
- 저장하여 빠져나온 뒤 다시 수정하고 싶으면 `git commit --amend` 사용
- `git add`로 필요한 파일만

> `git config --global core.editor "vim"` 를 통해 editor를 vim으로 설정하면 commit 시 vim 사용 가능

## REAME.md

- 기본적인 REAME.md 구조

```md
# git-practice

git session을 위한 git 명령어 연습

see [Demo](https://www.google.com/)

## Documentation

### Installation

$ git clone {repo url}
$ cd {reponame}
$ pip install -r requirements.txt

### How To Start

$ python main.py

### Dependecy

- Python==3.10
- django=3.00
- reqests=0.10.0

### Features

- 행운의 숫자 추출
- 행운의 숫자 저장
- 행운의 숫자 기반 구매

### More Information

- [API docs]()
- [Official website]()

### Contributing

Please see [CONTRIBUTING.md]()

### License

Sesame is Free software, and may be redistributed under the terms of specified in the [LICENSE]() file.
```

## LICENSE

- MIT License
  - MIT에서 만든 라이센스로, 모든 행동에 제약이 없으며, 저작권자는 소프트웨어와 관련한 책임에서 자유롭습니다.
- Apache License 2.0
  - Apache 재단이 만든 라이센스로, 특허권 관련 내용이 포함되어 있습니다.
- GNU General Public License v3.0(GPL)
  - 가장 많이 알려져있으며, 의무사항(해당 라이센스가 적용된 소스코드 사용시 GPL을 따라야 함)이 존재합니다.

## DevOps

DevOps = Development + Operations

- 개발과 운영의 합성어
- Dev: Plan - Code - Build - Test
- Ops: Release - Deploy - Operate - Monitor
- Cross Functional Team: 개발과 운영을 한 팀으로 묶어 프로세스의 자동, 단일화
- CI/CD Tool 이용하여 Build, Test, Deploy 자동화
- Pros
  - 커뮤니케이션 리소스 개선
  - 개발, 배포 속도가 빨라짐
  - 프로세스 간소화
  - 짧은 릴리즈 주기

> CI(Continuous Integration)/CD(Continuous Development) 파이프라인 구축이 핵심

![ci-cd](/assets/images/2023/04/25/ci-cd.png)

### CI(Continuous Integration)

- 테스트가 통과한 코드만 merge
- 자동화된 프로세스
- 코드 변경사항의 정기적 빌드, 테스트 병합 자동화
- Pros
  - 빠른 디버깅
  - 코드 품질 개선
  - 검증 및 릴리즈 시간 단축

### CD(Continuous Deplyment)

- Continuous Delivery + Continuous Deplyment
- Continuous Delivery: 공유 저장소로 자동 Release(Test -> Staging)
- Continuous Deployment: Production Level까지 자동 Deploy(Test -> Staging -> Production)
- MSA(MicroService Architecture) + Agile 일 경우, 사용자에게 최대한 빠른 시간안에 Production 제공 필요

### github actions

github에서 공식 제공하는 CI/CD Tools

- Workflow
  - Job들로 구성. Event에 의해 트리거되는 자동화된 프로세스
  - 최상위 개념
  - YAML으로 작성되며, .github/workflows에 저장
- Event
  - Workflow를 실행하는 규칙
  - Push, pull request, Cron, webhook으로 연결된 외부 이벤트에 의해 실행
    - cron은 어떤 주기를 두어, 주기적으로 동작시킴
- Job
  - Step들로 구성
  - 가상환경의 인스턴스에서 실행
  - 다른 Job에 의존관계를 가질 수 있고, 독립 병렬 실행도 가능
- Step
  - Task들의 집합으로 커맨드를 실행하거나 action을 실행
- Action
  - 가장 작은 단위
  - Step을 연결해 Job을 구성
  - 재사용 가능
  - marketplace나 개인이 만든 action을 사용할 수 있음
- Runner
  - workflow가 실행될 인스턴스
  - github-hosted runner와 self-hosted runner로 나뉨

### github actions - example

```yaml
name: Hello

on:
  push:
    branches:
      - main
jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - name: Say Hello
        run: |
          echo "hello"
          echo "world!"
```

main 브랜치에 push가 일어날때마다 echo 명령어들이 일어남 -> Hello World!

## git branch

- 원래 있던 명령어가 `checkout`
  - 시간/공간 단위 점프
  - 상태 restore하는 명령어
- switch와 restore로 갈라짐
- 작업 과정
  - branch 생성 후 옮기기
    ```sh
    $ git branch
    $ git branch fb
    $ git switch fb
    ```
  - branch에서 작업 완료(commit 여러개)
  - 상위 branch로 이동 `git switch main`
  - `git merge fb` 후 `git branch -d fb`로 작업 끝난 branch 삭제

## branching models

- **git flow**
  - (hotfix) - `main` - (release) - `develop` - feature
    - main : 실제 사용자들에게 보여지는 브랜치
    - develop : 다음 버전을 위한 개발
    - feature : 기능 개발을 위한 브랜치
    - release : develop에서 main으로 넘어가기 위한 브랜치(선택)
    - hotfix : 오류 긴급 수정을 위한 브랜치(선택)
  - pros: 가장 많이 적용, 각 단계가 명확히 구분
  - cons: 너무 복잡하다
    ![git-flow](/assets/images/2023/04/25/git-flow.png)
- **github flow**
  - `main` - feature
    - 기능 개발 후 `main`에 합치는 순간 모든 커밋은 deploy
  - pros: 브랜치 모델 단순화
  - cons: CI 의존성이 높기 때문에, 실수 발생 시 위험(pull request로 방지)
    ![github-flow](/assets/images/2023/04/25/github-flow.png)
  - github UI를 사용하여 Pull Request를 통해 검토 후 배포된다.
- **gitlab flow**
  - `production` - `pre-production` - `main` - feature
  - deploy, issue에 대한 대응이 가능하도록 보완
    ![gitlab-flow](/assets/images/2023/04/25/gitlab-flow.png)

## Sane GitHub Labels

[Sane GitHub Labels](https://medium.com/@dave_lunny/sane-github-labels-c5d2e6004b63)

> github issues label 명칭에 관한 레퍼런스

---

## ✏️여담

원래 GitKraken으로 GUI 툴만 사용했었는데 이번에 CLI 명령어를 많이 배웠네요.

commit 작성도 주먹구구식으로 어디서 들은 대로 대강 작성했는데 commit 작성법도 방법이 있었네요. 이제 좀 똑바로 작성해야겠어요.

제 블로그도 github action으로 자동 CI/CD가 되는 중이였는데 정확한 방법을 잘 몰랐거든요. 이번에 CI/CD 개념과 github action을 배울 수 있어서 좋았어요! 다음 프로젝트에 써봐야겠어요🐤
