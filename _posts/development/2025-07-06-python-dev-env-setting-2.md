---
layout: single
title: "[Python] VScode 개발 환경 설정(2)"
date: 2025-07-06 23:55:00 +0900
categories: development
---

[저번 편(python 개발 환경, 코드 포매팅)](/development/python-dev-env-setting-1/)에 이어 vscode의 git 관련 도구 및 기타 유용한 도구들을 소개하고자 한다.

## 설치할 Extension 목록

- **Python 개발**

  - `Python` – 기본 Python 지원
  - `Jupyter` – 노트북 파일(`.ipynb`) 실행 및 편집

- **코드 포매팅 및 Lint**

  - `Black Formatter` – 코드 스타일 포매터
  - `isort` – import 순서 자동 정렬
  - `flake8` – linting 도구
  - `prettier` – `Markdown`, `YAML`, `JSON` 등의 기타 파일 포매팅

- **Git 관련**

  - `Git Graph` – 시각적 Git 히스토리 확인

- **기타 유용한 도구**
  - `GitHub Markdown Preview` – GitHub 스타일의 마크다운 미리보기
  - `Rainbow CSV` – CSV 컬러링으로 가독성 향상
  - `indent-rainbow` – 들여쓰기 레벨별 색상 표시
  - `GitLens` – 각 줄의 마지막 커밋 정보 확인 (범인 색출용)

## Git Graph

`Git Graph`는 `vscode`에서 활용가능한 무료 Git GUI 툴이다. 나는 사실 GIT GUI툴로 유료인 `GitKraken`을 사용하고 있지만, `vscode` 위에서 해당 프로젝트를 손쉽게 확인가능한 `git graph`도 나쁘지 않다고 생각한다.

특히, 초보자 입장에서 유료툴을 결제하기에 부담이 될테니 무료인 `git graph`를 많이 사용하게 될 것이다.

> 누군가는 Git을 CLI로 써야한다고 주장한다. 하지만, GUI로 켜놔야지 모든 브랜치의 커밋 히스토리가 한눈에 들어오기 때문에 프로젝트의 git이 꼬이거나 충돌 날 확률을 줄일 수 있다.

### Git 추가 설정(remote 최신화) - auto fetch

초보자들과 함께 프로젝트를 하다보면 브랜치 최신화하라고 전달하였을때 최신화를 하지 않았으면서 했다고 하는 사람들이 꼭 있다. 무슨 소리인가 알아보면 `git fetch`를 안해놓고 자신이 최신 `main`에 있다고 주장하는 것이다.

혹은, 어디 강의에서 들은 내용대로 냅다 `git pull` 해버리고 충돌이 나거나 이상하게 `merge commit`을 만들어버린채 영문도 모르고 작업하고 있는 사람이 존재한다.

그렇기 때문에 항상 GUI에서 `fetch`를 해서 문제 없는 지 확인 후, `commit` 및 `push`를 하도록 가이드를 해주는 편이다.

#### Step 1: 설정(`ctrl + ,`) > auto fetch 검색

![git setting](/assets/images/2025/07/06/git1.png)

#### Step 2: true로 변경

![git setting](/assets/images/2025/07/06/git2.png)

### Git 추가 설정(remote 최신화) - auto prune

브랜치 작업물이 `main`에 반영되고 브랜치를 삭제하였을 때, `git graph`에 반영이 안되어 있을 수 있다.

이런 문제는 특히 `PR merge` 시 `squash and merge`로 처리하였을때 원격에서는 작업 브랜치가 지워졌지만 내 local `git graph`에서는 계속해서 보일 수도 있다.

이는 `prune`(가지치기)가 안 일어난 문제로 이러한 것도 자동으로 갱신하게 할 수 있다.

#### Step 1: 설정(`ctrl + ,`) > prune 검색

![git setting](/assets/images/2025/07/06/git3.png)

#### Step 2: 5가지 설정값 체크

![git setting](/assets/images/2025/07/06/git4.png)

### Git Graph fetch 버튼

위 설정을 마쳤다면 일정시간마다 알아서 `fetch`를 해오겠지만, 항상 `commit`이나 `push`를 하기 전에 직접 `fetch` 버튼을 눌러 원격 정보를 최신화해 충돌이 발생하지 않게 신경쓰자.

직접 `fetch`하는 버튼은 아래 스크린샷과 같이 우측 상단에 존재한다.

![git setting](/assets/images/2025/07/06/git5.png)

위 세팅들을 전부 마쳤다면 `vscode`와 `gitgraph`로 편리하게 git 협업도 할 수 있게 될 것이다.

## 기타 Extensions

### GitHub Markdown Preview

마크다운으로 README를 작성할 때 미리보기에서는 이뻤더라도 `github`에서는 안 이쁠 수가 있다. 이는 기본 `vscode`와 `github`의 랜더링 스타일이 다르기 때문이다.

이 확장을 사용함으로써 README 작성시 `github`에 어떻게 보일 지 처음부터 파악이 가능하다.

### Rainbow CSV

AI 개발이나, 데이터를 만질 때 `CSV`를 많이 쓰게 되는데 이때 가독성을 향상시켜주는 확장이다.

각 컬럼 데이터에 맞게 색깔을 바꿔주어 보기 편해진다.

### indent-rainbow

인덴트(들여쓰기) 레벨별로 색상이 표시된다. 크게 의미는 없지만 나름 가독성이 좋아져서 추천한다.

### GitLens

사실 `gitlens`도 `gitgraph`와 같이 gui 지원도 되는 git tool이지만 유료이다.

하지만 무료 사용자에게도 지원되는 기능이 있는데, 바로 코드 각 줄마다 어떤 커밋으로 누가 작성한 것인지 정보를 쉽게 볼 수 있는 것이다.

프로젝트 도중 버그가 발생 시 디버깅을 하며 문제 발생된 코드를 찾게되는데, 이 확장을 통해 누가 작성한 코드인지 바로 확인 후 고쳐달라고 바로 요청할 수 있다.

## 마무리

개발을 시작한 이후로 2대의 데스크탑과 4대의 노트북을 거치며, 매번 새로 컴퓨터를 세팅할 때마다 개발 환경을 하나하나 직접 설정해왔다.

> 요즘은 설정을 불러오는 기능 덕분에 이전 세팅을 쉽게 가져올 수 있지만, 예전에는 그런 기능이 있는 줄도 몰랐다\...

예전에는 필요한 설정이 보일 때마다 즉석에서 하나씩 추가하며 작업했지만, 이제는 내가 자주 사용하는 환경들을 정리해두고 싶어졌다. 특히, 다른 사람들과 설정을 공유할 때 옆에서 일일이 설명하는 것보다 정리된 문서를 보여주는 게 훨씬 효율적이라고 느꼈다.

이런 생각에서 이번 포스팅을 작성하게 되었다.

개발을 막 시작한 사람은 물론, 경험이 많은 개발자라도 놓치고 있던 유용한 설정이 있을 수 있다. 이 글이 모두에게 조금이라도 도움이 되기를 바란다.
