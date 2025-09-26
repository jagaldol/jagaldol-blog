---
layout: single
title: "[Zotero] Cite Preview Resizer 플러그인을 만들며"
date: 2025-09-27 01:23:00 +0900
categories: development
---

Zotero 7 PDF 리더에서 인용이나 주석 위에 마우스를 올리면 작게 뜨는 미리보기 팝업 때문에 늘 고생했다.  
글자가 너무 작아서 참고문헌을 확인하려면 결국 팝업을 클릭해서 본문으로 이동하거나, 눈을 찌푸려가며 겨우 읽어야 했다. 놀랍게도 Zotero에는 이 팝업 크기를 조절하는 기본 기능이 없었고, 같은 문제를 해결해 주는 플러그인도 찾을 수 없었다. 결국 내 눈 건강을 위해서라도 직접 해결책을 만들기로 했고 성공적으로 플러그인을 제작하였다.

![확장된 인용 미리보기 팝업](/assets/images/2025/09/27/zotero-cite-preview-resizer.png){: .width-640px}

이미지 왼쪽은 Zotero 기본 미리보기 팝업(고정 크기)이고, 오른쪽은 플러그인을 적용해 넓힌 모습이다. 적용 후에는 드래그로 자유롭게 크기를 조절할 수 있고, 환경설정에서 기본 팝업 크기도 원하는 값으로 바꿀 수 있다.

{% linkpreview "https://github.com/jagaldol/zotero-cite-preview-resizer" %}

개발한 결과물은 프로젝트 페이지에서 정리해 두었다.  
👉 <https://jagaldol.com/projects/zotero-cite-preview-resizer>

## Zotero란?

Zotero는 오픈소스 서지 관리 프로그램으로, 특히 논문과 학술 자료를 수집하고 정리하는 데 특화되어 있다.  
브라우저 확장을 통해 arXiv, IEEE Xplore 같은 데이터베이스에서 메타데이터와 PDF를 한 번에 저장할 수 있고, 수집한 자료는 라이브러리로 자동 동기화된다.

- arXiv처럼 PDF를 제공하는 사이트에서는 "Save to Zotero" 버튼 한 번으로 논문과 서지 정보를 통째로 내려받을 수 있다.
- 내장 PDF 리더에서 하이라이트, 밑줄, 주석을 자유롭게 남길 수 있어 논문을 읽으면서 바로 필기하기 좋다.
- Word, LibreOffice, Google Docs 플러그인과 연동해 인용문과 참고문헌을 자동으로 생성해 준다.

Zotero에 대해 더 궁금하다면 회사 멘토님이신 [chanmuzi](https://github.com/chanmuzi)님의 블로그 글을 추천한다.  
👉 <https://chanmuzi.tistory.com/489>

## 구현 핵심

### 1. Reader 이벤트 훅킹

팝업 DOM에 접근하려면 Zotero Reader가 툴바를 그릴 때를 잡아야 했다.  
`renderToolbar` 이벤트에 리스너를 붙이고, 문서 루트에 CSS 변수를 심은 뒤 팝업에 스타일을 덮어씌웠다.

```ts
Zotero.Reader.registerEventListener(
  "renderToolbar",
  ({ doc }) => injectIntoReader(doc),
  addon.data.config.addonID
);
```

- 기본값은 `addon/prefs.js`에서 `popupWidth=800`, `popupHeight=500`으로 정의했다.
- 팝업 요소에는 `resize: both`를 넣고, 최대 폭/높이를 뷰포트 95%로 제한해 화면 밖으로 튀어나가지 않도록 했다.
- 큰 이미지가 들어있는 경우를 대비해 `.inner` 컨테이너에는 `overflow: auto`를 적용했다.

### 2. 환경설정과 기본값 유지

팝업을 키워도 새 창을 열면 다시 원래대로 돌아가는 문제가 있어서 **Preferences 패널을 따로 만들었다**.  
사용자가 원하는 너비/높이를 입력하면 `Zotero.Prefs`에 저장해 두고 다음 팝업부터 즉시 반영한다.

![Preferences 화면](/assets/images/2025/09/27/preferences.png){: .width-500px}

- 최소치는 240×120으로 제한했고, 입력값은 실시간으로 CSS 변수에 업데이트한다.
- Fluent 포맷으로 한글/영문 두 언어를 기본 지원해 UI 번역 작업을 단순화했다.

### 3. 설치 가이드

GitHub [Releases](https://github.com/jagaldol/zotero-cite-preview-resizer/releases)에서 최신 `.xpi` 패키지를 내려받은 뒤 아래 순서대로 설치한다.

먼저 Zotero 상단 메뉴에서 `Tools ▸ Plugins`를 눌러 플러그인 관리자 창을 연다.

![Tools ▸ Plugins 메뉴](/assets/images/2025/09/27/tool-plugins.png){: .width-500px}

오른쪽 상단의 기어 아이콘을 눌러 `Install Plugin From File…`을 선택하고 방금 내려받은 `.xpi` 파일을 지정한다.

![Install Plugin From File… 동작](/assets/images/2025/09/27/install-plugin-from-file.png){: .width-500px}

설치가 완료되면 Zotero를 한 번 재시작한 뒤 Reader 탭을 다시 열어 넓어진 미리보기 팝업을 바로 확인할 수 있다.

## 배포와 테스트

- 개발은 [`zotero-plugin-toolkit`](https://github.com/windingwind/zotero-plugin-toolkit)을 기반으로 진행했다.
- `npm run start`로 Zotero 7 개발 프로필을 띄워 놓고, 팝업을 열어 실시간으로 스타일을 확인했다.
- 릴리스 빌드는 `npm run build` (번들링 + 타입 검사)로 생성해 GitHub Release에 업로드했다.

소스 코드는 GitHub에서 확인할 수 있다.  
👉 <https://github.com/jagaldol/zotero-cite-preview-resizer>

## 만들면서 느낀 점

플러그인을 만들어 보면서 느낀 가장 큰 포인트는 "CSS를 어디에 어떻게 주입할 것인가"였다. Zotero 플러그인 환경을 세팅하고 개발자 도구로 팝업 DOM 구조를 살펴본 뒤, Codex와 VibeCoding을 활용해 필요한 수정 사항을 던져 주기만 해도 금세 결과물을 얻을 수 있었다. 덕분에 스타일 시트를 여러 번 손으로 붙여 넣으며 디버깅하는 시간을 크게 줄였다.

CSS가 적용되는 위치만 명확하게 찾으면 이후 작업은 놀랄 만큼 가볍다. 이벤트 훅을 연결하고, Codex에게 상황을 설명하며 코드를 다듬어 나가는 과정이 짧은 시간 안에 반복됐고, 그 사이에 나는 실제 동작을 확인하고 다음 아이디어를 정리하는 데 집중할 수 있었다.

## 마무리

네이버 부스트캠프에서 팀원에게 처음 Zotero를 추천받은 뒤로 지금까지 유용하게 활용하는 논문 리더 도구였다. 이번 플러그인을 만든 덕분에 예전보다 훨씬 편하게 논문을 읽을 수 있게 되었고, 함께 공부하던 팀원들에게 공유했더니 다들 사용해보고 좋은 기능이라고 만족해줬다. 특히 수료 후 대학원으로 진학한 형이 있는데, 생각보다 너무 좋아해줘서 열심히 만든 보람이 있었다.

오픈소스 생태계에 조금이나마 기여할 수 있다는 사실이 참 즐겁다. 플러그인이 마음에 들거나 구현 과정이 궁금하다면 GitHub 저장소에서 코드를 살펴보고, 도움이 되었다면 ⭐️를 눌러 힘을 보태 주시면 정말 감사하겠다. 작은 별 하나가 다음 업데이트를 준비하는 데 큰 동력이 된다.

👉 프로젝트 소개: <https://jagaldol.com/projects/zotero-cite-preview-resizer>  
👉 GitHub 저장소: <https://github.com/jagaldol/zotero-cite-preview-resizer>
