---
layout: single
title: "Gemini 창작 기능 확장 - Lyria 3 음악 생성 & Nano Banana 2 이미지 생성"
date: 2026-03-06 09:00:00 +0900
last_modified_at: 2026-03-07 20:37:00 +0900
categories: ai
---

2월에 Google은 Gemini의 창작 기능을 음악과 이미지 양쪽으로 대폭 확장했다.
Lyria 3로 음악 생성을, Nano Banana 2로 이미지 생성을 한 단계 끌어올렸고,
관련 내용을 이 글로 통합 정리했다.

---

## Gemini Lyria 3 - AI 음악 생성

Google이 Gemini 앱에 [Lyria 3](https://gemini.google/overview/music-generation/) 음악 생성 기능을 추가했다.

![Gemini Lyria 3 소개 이미지](/assets/images/2026/03/06/lyria-3-hero.png)

텍스트로 아이디어를 설명하거나 사진을 업로드하면, **30초 분량의 트랙**을 보컬, 가사, 커버 아트와 함께 생성해준다.
다양한 장르를 지원하며, 생성된 모든 트랙에는 **SynthID 워터마크**가 자동 삽입된다.

YouTube 크리에이터용 Dream Track 기능으로도 확장되었으며,
18세 이상 사용자를 대상으로 영어, 독일어, 스페인어, 프랑스어, 힌디어, 일본어, 한국어, 포르투갈어를 지원한다.

스터디에서는 실제로 Gemini Lyria로 노래를 만들어보며 체험해봤다.
텍스트 프롬프트만으로 보컬이 포함된 트랙이 생성되는 점이 인상적이었다.

### 스터디에서 만든 곡

사용한 프롬프트:

> AI로 인해 망가진 주니어 개발자 취준생들의 한을 노래해줘

[스터디 곡 들어보기 (Gemini Share)](https://gemini.google.com/share/10ddcb8061ef)

<a href="https://gemini.google.com/share/10ddcb8061ef" target="_blank" rel="noopener noreferrer"><img src="/assets/images/2026/03/06/lyria-study-song-cover.jpg" alt="스터디에서 만든 곡 커버" class="width-300px"></a>

> Weekly Tech Trend Talk 스터디(26.02.19)

---

## Nano Banana 2 - Gemini 이미지 생성

Google이 [Nano Banana 2](https://blog.google/innovation-and-ai/technology/ai/nano-banana-2/)를 2월 26일 출시했다.
**Gemini 3.1 Flash** 기반의 이미지 생성 모델로, Nano Banana Pro의 고품질과 Flash의 속도를 결합했다.

![Nano Banana 2 공식 이미지](/assets/images/2026/03/06/nanobanana2-hero.webp)

![Nano Banana 2 샘플 이미지](/assets/images/2026/03/06/nanobanana2-sample.webp)

**나노바나나 변천사:**

1. **Nano Banana** (초기) - Gemini 2.5 Flash Image 기반
2. **Nano Banana Pro** - Gemini 3.0 Pro 기반
3. **Nano Banana 2** (현재) - Gemini 3.1 Flash 기반

주요 특징:

- 512px ~ 4K 해상도, 다양한 비율 지원
- 웹 검색 기반 실시간 정보를 반영한 이미지 생성
- 최대 5명 캐릭터 유사성 유지, 10개 객체 충실도 보장
- Gemini 앱의 **기본 이미지 생성 모델**로 설정됨
- 개발자용 Gemini API, Gemini CLI, Vertex API에서도 프리뷰 제공

음악(Lyria 3)과 이미지(Nano Banana 2)가 하나의 Gemini 워크플로우로 묶이면서
아이디어 스케치부터 사운드트랙/비주얼 시안 제작까지 한 번에 이어서 작업할 수 있게 됐다.

> Weekly Tech Trend Talk 스터디(26.02.28)

---

## 후기

AI가 텍스트와 코드를 넘어 음악, 이미지 등 다양한 창작 영역으로 빠르게 확산되고 있다.
Google이 Gemini 플랫폼 하나에서 음악과 이미지 생성을 모두 제공하면서,
크리에이터 도구로서의 실용성이 한층 높아졌다.
