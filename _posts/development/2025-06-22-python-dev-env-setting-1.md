---
layout: single
title: "[Python] VScode 개발 환경 설정(1)"
date: 2025-06-22 18:39:00 +0900
categories: development
---

JetBrains 계열의 IntelliJ나 WebStorm을 사용하던 사람이 VSCode로 `Python` 개발을 시작하면 불편함을 느끼는 경우가 많다.
VSCode는 기본적으로 단순한 텍스트 에디터에 가깝기 때문에, 제대로 활용하려면 여러 `Extension`을 설치하고 세부 설정을 직접 조정해야 한다.

하지만 필요한 설정만 잘 해두면 어떤 IDE보다도 효율적이고 쾌적한 개발 환경을 만들 수 있다.
여기서는 내가 `Python` 개발을 하며 실제로 유용하다고 느낀 VSCode 환경 설정을 정리해본다.

## 설치할 Extension 목록

이번 포스팅에서는 **Python 개발**과 **코드 포매팅 및 Lint**만을 다룰 예정이다.

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

## Python 기본 설정

jetbrain ide들을 사용하여 개발을 해보았다면, 각 변수의 타입이 자동으로 힌트가 생기는 걸 본 적이 있을 것이다. 또한, 귀찮게 import 문을 직접 적지않아도 Enter를 누르면 자동으로 import 되는 경험을 해본 적이 있을 것이다.

VSCode에는 이런 편의기능이 없을까? 당연히 존재한다. python extension을 설치했다면 자동으로 `pylance`가 설치되었을 것이다. 이 `pylance` 설정을 조금만 건드려줘도 사용성이 매우 올라간다.

### 타입 힌트 표시 - inlay hint

#### Step 1: 설정(`ctrl + ,`) > python inlay hint 검색

![검색 결과](/assets/images/2025/06/22/inlay-hint-search.png)

#### Step 2: 스크린샷과 같이 4개의 항목을 설정해준다.

![설정](/assets/images/2025/06/22/inlay-hint-setting.png)

#### 적용 결과

아래와 같이 어떤 함수 호출에 각 인자가 어떤 인자명인지, 변수가 어떤 타입인지 등이 힌트로 표시된다.

![결과](/assets/images/2025/06/22/inlay-hint-result.png)

### package 자동 import

#### Step 1: 설정(`ctrl + ,`) > python auto import 검색

![검색 결과](/assets/images/2025/06/22/auto-import-search.png)

#### Step 2: 스크린샷과 같이 Auto Import Completions을 설정한다.

![설정](/assets/images/2025/06/22/auto-import-setting.png)

#### 적용 결과

![결과1](/assets/images/2025/06/22/auto-import-result1.png)

필요한 패키지가 import되어 있지 않더라도 적절한 후보를 추천해준다. 이 상태에서 Enter를 치면 자동으로 import되며 아래와 같이 import 문이 생성된다.

![결과2](/assets/images/2025/06/22/auto-import-result2.png)

## 코드 포매팅 및 Lint 기본 설정

프로젝트를 할때 팀원간 formatting이 맞지 않으면 개발 도중 많은 충돌이 생기고 피로감이 증가한다. 이걸 막기위해 존재하는 도구가 바로 linter이다.

linter를 통해 코드 포맷을 강제하고, 더 나아가 자동으로 formatting을 맞출 수 있다. 대표적으로는 javascript 계열의 `prettier`와 `eslint`가 존재한다.

그렇다면 python에는 어떤 게 존재할까? 바로 `black formatter`와 `flake8`이다.

`black`은 `prettier`와 같이 단순 포매팅을 도와주는 도구이고, `flake8`은 함수 복잡도, 한줄 최대 길이 등 유지보수성을 고려하여 개발자가 직접 수정해야하는 규칙들을 지적해준다.

이에 더불어 import문을 자동 정렬해주는 `isort`까지 소개하고자 한다.

### VSCode 범용 formatter - `prettier`

VSCode에서는 다양한 언어를 다루다 보면 언어별로 포매팅 도구를 다르게 쓰는 게 일반적이다.

`prettier`는 `HTML`, `JS`, `CSS`, `Markdown`, `YAML`, `JSON` 등 프론트엔드 및 마크업 계열 파일에 가장 범용적으로 쓰이는 포매터로, 기본 포매터로 지정해두면 대부분의 파일에 적용된다.

> Python의 경우, `prettier`가 지원하지 않기 때문에 **Black Formatter**를 별도로 설정해줘야 한다.

#### Step 1: 설정(`ctrl + ,`) > `format on save` 검색 후 체크

해당 설정을 추가하면 저장(`Ctrl + S`)을 할때마다 자동으로 해당 파일에 대한 formatting이 적용된다.

![prettier](/assets/images/2025/06/22/prettier1.png)

#### Step 2: formatter 검색후 `prettier`로 변경

![prettier](/assets/images/2025/06/22/prettier2.png)

### Python - `black formatter`

다음은 python 전용 formatter인 `black formatter`이다. 언어별 설정을 통해 python에 대해서만 `black formatter`로 설정을 덮어씌울 수 있다.

#### Step 1: Python 전용 설정 진입

- `Ctrl + Shift + P` 후 `Preferences: Configure Language Specific Setting` 검색 후 Python 선택

![python config](/assets/images/2025/06/22/python-config1.png)

![python config](/assets/images/2025/06/22/python-config2.png)

#### Step 2: formatter 검색 후 `black formatter`로 변경

![black](/assets/images/2025/06/22/black1.png)

#### Step 3: `black args` 검색 후 한줄 최대 길이 120으로 변경

`black formatter`는 기본적으로 라인 길이를 88에 맞추려고한다. 하지만, 많은 프로젝트들이 그보다 훨씬 많은 길이로 코드가 작성되어 있고 이 설정을 변경하지 않으면 줄바꿈 지옥에 빠질 수도 있다.

![black](/assets/images/2025/06/22/black4.png)

- 항목 추가를 누른 뒤 `--line-length=120`을 작성하여 추가한다.

> 여기서는 python 전용 설정일 필요가 없다.

#### 결과

아래와 같이 못생긴 코드가 있다고 가정하자. 해당 코드는 엔터도 안되어 있고, 띄워쓰기도 통일성이 없다.

![black](/assets/images/2025/06/22/black2.png)

하지만 저장을 하는 순간 마법같이 아래처럼 이쁜 코드로 바뀐다.

![black](/assets/images/2025/06/22/black3.png)

### `isort`

이전 섹션에서 `auto import`를 설정했는데 import문이 아무렇게나 들어갈까 걱정이 되지 않는가? 하지만 `isort`를 사용하면 이런 걱정은 완전히 필요없다.

주요 특징은 알파벳별 정리, 유형별 정리(표준 라이브러리 / 서드파티 라이브러리 / 프로젝트 내부 라이브러리)가 있다. 사실상의 파이썬 오픈소스들의 표준 정렬 규칙으로 `auto import`와 함께 사용 시 import에 대해 더이상 신경쓸 필요가 없어진다.

#### Step 1: Python 전용 설정 진입

- `Ctrl + Shift + P` 후 `Preferences: Configure Language Specific Setting` 검색 후 Python 선택

![python config](/assets/images/2025/06/22/python-config1.png)

![python config](/assets/images/2025/06/22/python-config2.png)

#### Step 2: code action on save 검색 후 setting.json에서 편집 선택

![isort1](/assets/images/2025/06/22/isort1.png)

#### Step 3: setting.json에 설정 값 작성

![isort2](/assets/images/2025/06/22/isort2.png)

```json
"[python]": {
	"editor.defaultFormatter": "ms-python.black-formatter",
	"editor.codeActionsOnSave": {
		"source.organizeImports": "always"
	}
}
```

주피터 노트북에서도 `isort`가 동작하게 하고 싶다면 아래의 설정도 추가해야한다.

```json
"notebook.codeActionsOnSave": {
	"source.organizeImports": "explicit"
}
```

> 해당 설정법은 [microsoft-vscode/isort 공식 레포](https://github.com/microsoft/vscode-isort)에서도 안내하는 방법이다.

#### 결과

아래와 같이 전혀 정렬되어 있지 않은 import 문들이 존재한다고 하자.

![isort3](/assets/images/2025/06/22/isort3.png)

`Ctrl + S`를 누름과 동시에 아래와 같이 오픈소스들처럼 깔끔하게 정렬됨을 볼 수 있다.

![isort4](/assets/images/2025/06/22/isort4.png)

### `flake8`

파이썬 오픈소스들에서 ` .flake8`이라는 파일을 본적이 있는가? `flake8`은 많은 오픈소스들이 기본적으로 흔히 사용하는 python linter이다.

`flake8`은 자동 formatting 기능이 없기에 따로 설정할건 없지만, 위 `black`과 `isort`들에 충돌되는 규칙들을 처리해줄 필요가 있다.

#### line max length 처리

`flake8`은 `black formatter`에 비해 한술 더 떠, 79로 설정이 되어 있다.

이를 완화하여 편하게 쓸 수 있도록 위 `black formatter` 설정과 함께 120으로 맞춰줄 것이다.

![flake8](/assets/images/2025/06/22/flake8-1.png)

#### 예외 규칙 추가

`flake8`의 `E203`과 `W503` 규칙이 `black`과 충돌을 일으킨다. 이 규칙은 꼭 예외에 등록시켜놔야한다.

![flake8](/assets/images/2025/06/22/flake8-2.png)

#### 결과

저장을 누를때 마다 `flake8` 검사가 수행되며, 아래와 같이 오류사항을 안내해준다.

![flake8](/assets/images/2025/06/22/flake8-3.png)

불필요한 import가 존재하거나, 사용하지 않은 변수가 존재하거나 등등 코드 품질을 위한 경고를 띄워주므로 코드 관리가 한결 편해진다.
