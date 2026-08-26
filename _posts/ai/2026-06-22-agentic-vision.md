---
layout: single
title: "Agentic Vision이란 - 이미지를 확대하고 표시하며 다시 보는 시각 추론"
date: 2026-06-22 20:00:00 +0900
categories: ai
header:
  teaser: /assets/images/2026/06/22/agentic-vision-hero.png
---

멀티모달 모델은 이미지를 보고 답할 수 있지만, 작은 글자나 복잡한 그래프를 한 번에 정확히 읽지는 못한다. 한눈에 보이지 않는 부분을 확률적으로 메우면 그럴듯한 오답이 생긴다.

![카메라로 본 그래프를 확대하고 측정해 다시 검증하는 시각 추론 과정](/assets/images/2026/06/22/agentic-vision-hero.png){: .align-center }

Agentic Vision은 이미지 이해를 한 번의 관찰로 끝내지 않는다. 모델이 코드를 실행해 이미지를 확대하거나 표시하고, 그 결과를 다시 본 뒤 답을 고친다.

## Think, Act, Observe

```text
Think
어디를 확대하고 무엇을 계산할지 정한다
        ↓
Act
crop, rotate, annotate, OCR, 계산을 실행한다
        ↓
Observe
새로 만든 이미지를 다시 보고 검증한다
        ↺ 부족하면 반복
```

| 단계 | 하는 일 |
| --- | --- |
| Think | 질문과 원본 이미지를 보고 확인할 영역과 계산 순서를 계획 |
| Act | Python으로 이미지 조작, 객체 표시, 수치 계산 |
| Observe | 변환된 이미지를 시각 입력으로 다시 받아 누락과 오류 검토 |

핵심은 `Act`가 답을 정한 뒤 붙이는 장식이 아니라 **답을 정하기 전의 증거 생성 단계**라는 점이다. 결론을 먼저 내리고 그에 맞춰 박스를 그리면 검증이 아니라 사후 설명이다.

## 활용 1: 작은 영역을 확대해서 다시 보기

고해상도 도면, 먼 표지판, 작은 일련번호는 전체 이미지를 한 번에 볼 때 놓치기 쉽다.

```python
from PIL import Image

image = Image.open("input.png")
crop = image.crop((820, 240, 1320, 740))
crop.save("detail.png")
```

모델은 처음부터 정확한 좌표를 알 필요가 없다. 넓게 자른 뒤 grid overlay나 OCR 결과를 보고 다시 범위를 줄일 수 있다.

Google이 소개한 건축 도면 검증 사례도 지붕 모서리 같은 특정 patch를 잘라 건축 규정과 대조하는 방식이었다.

## 활용 2: 이미지 위에 표시하며 검산하기

여러 객체를 세거나 대응 관계를 찾을 때 원본 위에 번호와 박스를 그린다.

```python
from PIL import Image, ImageDraw

image = Image.open("input.png")
draw = ImageDraw.Draw(image)
draw.rectangle((100, 80, 220, 260), outline="red", width=4)
draw.text((105, 55), "1", fill="red")
image.save("annotated.png")
```

annotated 이미지를 다시 보면 이미 센 대상을 중복하거나 하나를 빠뜨린 오류를 찾기 쉽다. 사람이 종이에 표시하며 세는 것과 같은 visual scratchpad다.

## 활용 3: 그래프를 데이터로 바꿔 계산하기

막대그래프의 값과 비율을 눈대중으로 계산하지 않고, 픽셀 좌표를 값으로 환산해 Python으로 계산한다.

```text
y축 눈금 위치 확인
→ 막대 꼭대기 좌표 추출
→ 픽셀과 실제 값 사이의 비율 계산
→ 값 테이블 생성
→ 합계와 비율 계산
```

이미지를 예쁘게 다시 그리는 것이 목적이 아니다. 시각적 추정을 결정적인 계산으로 바꾸는 것이 목적이다.

## 프롬프트로 바로 유도하기

코드 실행과 이미지 읽기 도구가 있는 agent라면 다음처럼 요청할 수 있다.

```text
확신이 없는 영역은 crop해서 확대하고, 결과 이미지를 다시 확인한 뒤 답해줘.
```

```text
인식한 객체마다 번호와 박스를 표시한 이미지를 만든 뒤 누락과 중복을 검산해줘.
```

```text
그래프 수치는 눈대중으로 읽지 말고 좌표를 추출해 코드로 계산해줘.
```

## 코드 실행만으로는 부족하다

터미널이 `detail.png`를 만들었다는 로그를 반환해도 모델이 그 픽셀을 본 것은 아니다. 생성한 이미지를 다음 시각 입력으로 되돌리는 도구가 있어야 한다.

```text
exec("python crop.py")
→ detail.png 생성
→ view_image("detail.png")
→ 모델이 새 이미지를 관찰
```

필요한 최소 구성은 다음 두 가지다.

- 코드를 실행하고 파일을 만드는 도구
- 생성된 이미지를 모델의 visual input으로 되돌리는 도구

이 연결이 없으면 Think–Act까지는 있어도 Observe가 없어 루프가 닫히지 않는다.

## 언제 효과적인가

- 작은 텍스트와 표식 확인
- UI 스크린샷의 정렬·간격 측정
- 표와 그래프 수치 계산
- 객체 개수와 위치 검산
- 문서 스캔의 특정 영역 비교

반대로 원본 해상도가 이미 정보를 잃은 경우에는 확대해도 새로운 픽셀이 생기지 않는다. OCR과 보간이 읽기 가능성을 높일 수는 있어도 없는 정보를 복원했다고 단정하면 안 된다.

Agentic Vision의 가치는 더 오래 생각하는 데 있지 않다. **시각적 추측을 새 증거로 교체하고, 그 증거를 다시 관찰하는 데** 있다.

## 참고 자료

- [Google - Introducing Agentic Vision in Gemini](https://blog.google/innovation-and-ai/technology/developers-tools/agentic-vision-gemini-3-flash/)
- [Gemini API code execution](https://ai.google.dev/gemini-api/docs/code-execution)
