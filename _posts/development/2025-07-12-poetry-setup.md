---
layout: single
title: "[Python] Poetry로 각 project 내에 venv 가상환경 만들기"
date: 2025-07-12 15:27:00 +0900
categories: development
---

Python에서도 프로젝트마다 독립적으로 패키지를 관리할 수 있도록 가상환경을 구성한다.  
가장 기본적인 방식은 `venv`를 사용하는 것이며, 프로젝트 루트에 `.venv` 폴더를 만들어 그 안에 필요한 패키지를 설치하고 `requirements.txt`로 패키지 목록을 관리한다. 이렇게 하면 프로젝트 폴더를 통째로 옮기거나 삭제할 때 가상환경도 함께 정리되기 때문에 관리가 편하다.

하지만 요즘은 `venv`보다 더 발전된 툴들이 널리 사용된다. 대표적으로 `poetry`, `pipenv`, `conda` 등이 있으며, 그중에서도 **Poetry**는 의존성 관리와 빌드, 배포까지 통합적으로 지원해 많은 인기를 끌고 있다.

단, Poetry는 기본적으로 시스템 전역에 모든 가상환경을 모아두는 구조를 사용한다. (`conda`도 동일하다.) 이 구조는 특정 프로젝트의 가상환경을 직접 보고 관리하고 싶은 사람에게는 불편하게 느껴질 수 있다. 예를 들어 가상환경을 삭제하고 싶을 때 폴더가 눈에 보이지 않으면 처리하기 까다롭다.

다행히도 Poetry는 설정을 통해 `.venv` 폴더를 프로젝트 루트에 직접 생성하게 만들 수 있다. 이 방식은 JavaScript에서 `node_modules`가 프로젝트 내에 존재하듯 Python 프로젝트도 자체적인 독립 환경을 갖게 해주므로 더 직관적이고 관리도 쉬워진다.

이 포스팅에서는 Poetry로 가상환경을 프로젝트 내부에 생성하도록 설정하는 방법을 소개하고자 한다.

## Poetry 설치하기

### Windows

Windows에서는 PowerShell을 열고 아래 명령어를 입력하면 Poetry를 설치할 수 있다.

```powershell
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | python -
```

> 위 명령어는 Poetry 공식 문서에서 제공하는 최신 설치 방법이다. 만약 설치가 제대로 되지 않는다면 [공식 문서](https://python-poetry.org/docs/)를 참고하자.

### 환경 변수 설정

Poetry는 기본적으로 아래 경로에 설치된다. 시스템 어디서나 Poetry 명령어를 사용할 수 있도록 해당 경로를 환경 변수(Path)에 등록해주자.

```
C:\Users\hyejun\AppData\Roaming\Python\Scripts
```

> 윈도우 환경 변수 설정 방법은 자명하니 생략한다.

### macOS

macOS에서는 아래 명령어 한 줄로 간단하게 설치할 수 있다.

```sh
brew install poetry
```

## 🌟 Poetry 가상환경을 프로젝트 내부에 생성하기

이 설정이 이 포스팅의 핵심이다. 아래 명령어를 입력하면 Poetry가 `.venv` 폴더를 프로젝트 루트에 생성하도록 동작 방식을 바꿔준다.

```sh
poetry config virtualenvs.in-project true
```

### 결과 예시

설정을 적용한 후 `poetry install` 명령어를 실행하면, 프로젝트 루트에 `.venv` 폴더가 자동으로 생성된다. 이 폴더는 가상환경이 위치한 곳으로, 다른 파일들과 함께 한눈에 관리할 수 있어 편리하다.

![Poetry .venv 폴더 예시](/assets/images/2025/07/12/poetry-venv-result.png){: .width-300px}

이제 프로젝트를 삭제하거나 복사할 때 가상환경도 함께 정리되므로, 전역 디렉토리를 신경 쓸 필요가 없다.

## Poetry 기본 사용법

`poetry` 첫 사용자를 위해 기본적인 사용법도 함께 안내한다.

### 신규 프로젝트 시작

```bash
# 프로젝트 디렉토리 생성
mkdir my-project
cd my-project

# poetry로 pyproject.toml 생성
poetry init

# 필요한 패키지 설치
poetry add requests
```

`poetry init` 명령어는 `pyproject.toml` 파일을 생성하며, 설치할 패키지를 묻는 인터랙티브 모드로 진행된다. 이후 `poetry add` 명령어로 필요한 패키지를 추가하면 된다.

### 기존 프로젝트 설정

```bash
# 의존성 설치 및 가상환경 생성
poetry install
```

> `.venv` 폴더가 프로젝트 내에 있으므로, IDE에서 해당 디렉토리를 Python 인터프리터로 지정하면 된다.

### Poetry 환경에서 Python 스크립트 실행

```bash
poetry run python script.py
```

## 마무리

이제 Poetry도 `.venv`로 프로젝트 내부에 깔끔하게 정리하면서, 가상환경 관리를 더욱 직관적으로 할 수 있다.  
프로젝트를 복사하거나 백업할 때 가상환경이 함께 따라가기 때문에 외부 디렉토리나 숨겨진 위치를 따로 찾아다닐 필요도 없다.

Poetry의 전역 가상환경 설정이 불편했던 사람이라면 꼭 한 번 이 방식으로 설정해보길 추천한다!
