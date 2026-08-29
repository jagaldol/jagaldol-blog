---
layout: single
title: "T5 논문 리뷰 - 모든 NLP Task를 Text-to-Text로 통일하기"
date: 2026-09-04 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/09/04/t5-text-to-text-framework.webp
---

T5(Text-to-Text Transfer Transformer)는 분류, 질의응답, 요약, 번역처럼 입출력 구조가 다른 NLP 과제를 모두 `text → text` 문제로 바꾼다. 이 논문은 새로운 attention block을 제안하기보다 하나의 학습 interface 안에서 architecture, 사전학습 objective, data, transfer strategy와 scale을 같은 조건으로 비교한다.

## 모든 과제를 text-to-text로 바꾸기

![분류, 유사도, 요약과 번역을 모두 text-to-text 형식으로 표현한 T5](/assets/images/2026/09/04/t5-text-to-text-framework.webp)

입력 앞에 task를 구분하는 prefix를 붙이고, 정답은 항상 token sequence로 생성한다.

- 번역: `translate English to German: ...`를 입력하고 번역문을 생성한다.
- 분류: CoLA 문장을 입력하고 `acceptable`이나 `not acceptable`을 생성한다.
- 회귀: STS-B의 유사도 점수를 0.2 간격의 문자열로 바꿔 생성한다.
- 요약과 질의응답: 문서나 질문·context를 입력하고 요약문이나 answer span을 생성한다.

이 통일로 model architecture, maximum-likelihood loss, teacher forcing과 autoregressive decoding을 모든 task에서 재사용할 수 있다. 다만 task가 사라지는 것은 아니다. Prefix, label을 표현할 단어, 입력 직렬화와 평가 metric은 여전히 task별로 설계한다. T5의 prefix도 현대적인 자유형 instruction이라기보다 `summarize:`, `cola sentence:` 같은 task identifier에 가깝다.

## 최종 T5를 구성한 요소

### Encoder-decoder Transformer

![Encoder-decoder, language model과 prefix language model 구조 비교](/assets/images/2026/09/04/t5-transformer-architectures.webp)

T5는 [Attention Is All You Need](/papers/attention-is-all-you-need/)의 encoder-decoder 구조를 사용한다. Encoder는 입력 전체를 양방향으로 보고, decoder는 이전 출력만 보면서 encoder output에 cross-attention해 다음 token을 생성한다. 같은 text-to-text 조건에서 표준 encoder-decoder, decoder-only language model, prefix LM을 비교했을 때 표준 encoder-decoder가 가장 좋은 결과를 냈다.

Baseline은 encoder와 decoder에 각각 BERT-Base와 비슷한 12개 block을 둔 약 220M parameter 모델이다. 최종 실험에서는 Small 60M, Base 220M, Large 770M, 3B, 11B까지 같은 계열을 확장했다.

### Span corruption

![연속된 token span을 sentinel token으로 바꾸고 빠진 부분만 생성하는 T5 사전학습](/assets/images/2026/09/04/t5-span-corruption.webp)

사전학습에서는 원문 token의 15%를 평균 길이 3인 연속 span으로 고른다. 각 span을 서로 다른 sentinel token으로 치환하고, decoder는 sentinel과 빠진 span만 순서대로 생성한다.

예를 들어 입력이 `Thank you <X> me to your party <Y> week.`라면 target은 `<X> for inviting <Y> last <Z>`가 된다. [BERT](/papers/bert/)처럼 손상된 text를 복원하지만, 가려진 위치마다 token을 맞히는 encoder objective가 아니라 빠진 span을 하나의 target sequence로 생성한다. 복원할 부분만 target에 남기므로 원문 전체를 다시 생성하는 denoising보다 계산량도 줄어든다.

논문에서는 causal language modeling과 문장 순서 복원보다 denoising 계열이 강했다. Denoising 세부 변형 사이의 성능 차이는 작았고, 평균 span 길이 3이 대부분의 비번역 benchmark에서 token별 corruption보다 조금 나으면서 target도 짧았다. 따라서 span corruption의 선택은 큰 성능 도약보다 비슷한 성능을 더 짧은 target으로 얻는 효율 판단에 가깝다.

### C4

C4(Colossal Clean Crawled Corpus)는 2019년 4월 Common Crawl에서 추출한 약 745GB의 영어 text다. 문장부호로 끝나는 line, 최소 문장·단어 수, 영어 판정 확률 0.99, 중복된 3문장 span 제거 같은 heuristic으로 boilerplate, code와 중복을 줄였다.

정제하지 않은 6.1TB variant는 745GB C4보다 모든 평가 묶음에서 낮았다. 반면 뉴스, Wikipedia, 책처럼 downstream task와 domain이 맞는 작은 corpus는 특정 task에서 C4보다 나았지만, 같은 data를 수백·수천 번 반복하면 사전학습 loss는 낮아져도 downstream 성능은 떨어졌다. Data의 양만 늘리는 것으로는 부족했고, 정제, domain과 반복 횟수도 결과를 바꿨다.

## 비교 실험에서 남은 결론

- 모든 parameter를 갱신하는 fine-tuning이 이 실험의 adapter나 gradual unfreezing보다 강했다. Adapter는 저장·전환 비용을 줄이지만 당시 설정에서는 성능을 일부 포기했다.
- Supervised multi-task training만으로 끝내는 방식은 가장 좋은 결과를 내지 못했다. Unsupervised pre-training 뒤 task별 fine-tuning이 나았고, unsupervised와 supervised task를 섞어 pre-train한 뒤 다시 개별 fine-tuning하면 비슷한 수준에 도달했다.
- 같은 계산량을 더 긴 training, 더 큰 batch, 더 큰 model 또는 ensemble에 쓸 때 모두 성능이 올랐다. 여러 benchmark에서는 model size를 키우는 편이 작은 model을 오래 학습하는 것보다 더 큰 이득을 냈다.
- Domain이 맞는 data는 특정 task에 유리하지만 corpus가 작아 반복이 많아질 수 있다. 범용 전이에서는 넓고 큰 corpus와 domain 적합성 사이의 trade-off가 남는다.

## 최종 결과

최종 T5는 span corruption, C4, supervised task가 섞인 multi-task pre-training, task별 fine-tuning을 결합했다. 512-token sequence 2,048개를 batch로 삼아 100만 step, 약 1조 token을 사전학습했고 요약과 번역에는 beam search를 사용했다.

2019년 10월 24일 기준으로 T5-11B는 평가한 24개 task 중 18개에서 당시 최고 성능을 기록했다. 대표 결과는 GLUE 90.3, SuperGLUE 88.9, SQuAD validation EM 91.26·F1 96.22다. 그러나 WMT 번역 세 task에서는 최고 성능에 도달하지 못했다. 영어 중심 C4만 사용했고 backtranslation 같은 번역 전용 방법은 쓰지 않았다.

성능을 scale만으로 설명할 수도 없다. 같은 Base model에서 약 340억 token을 본 baseline의 GLUE는 83.28, 1조 token으로 늘린 baseline-1T는 84.80, span corruption과 최종 training recipe까지 적용한 T5-Base는 85.97이었다. Token 증가로 1.52점, 나머지 구성 변경으로 다시 1.17점이 올랐다.

## 전이학습 계보에서 T5의 자리

[GPT-1](/papers/gpt-1-pretraining-finetuning/)과 BERT는 대규모 사전학습 뒤 task별 fine-tuning하는 흐름을 굳혔다. T5는 여기에 “모든 task의 입출력을 text로 만들면 하나의 model·loss·decoding으로 비교할 수 있다”는 공통 규격을 제공했다. 새 Transformer 구조 하나를 제시한 논문이라기보다 2019년까지 흩어져 있던 전이학습 선택지를 통제된 실험으로 정리하고, 그 결론을 11B scale에서 검증한 논문에 가깝다.

이후 mT5는 같은 text-to-text 형식을 101개 언어로 넓혔고, FLAN-T5는 T5 checkpoint를 자연어 instruction으로 표현된 task mixture에 fine-tuning했다. Task prefix와 text label을 사용한 T5의 interface가 instruction tuning으로 이어진 것이다. 다만 원래 T5는 unseen instruction을 prompt만으로 처리하는 모델이 아니다. 각 downstream task의 labeled data로 parameter를 갱신하는 fine-tuning이 기본이며, 이 점에서 parameter를 고정하고 context로 적응하는 GPT-3와 구분된다.

오늘날 decoder-only chat model은 T5의 직접적인 architecture 후손이 아니다. T5의 영향은 encoder-decoder 구조 자체보다 서로 다른 task를 text 입출력과 자연어에 가까운 prefix로 표현하고, 여러 task를 하나의 training pipeline에 섞는 방식에 남았다.

## Text-to-text가 없애지 못한 것

Text-to-text는 task-specific head를 없앴지만 task 설계까지 없애지는 못했다. Classification label을 어떤 단어로 만들지, 회귀 값을 어떻게 문자열로 양자화할지, 입력을 어떤 순서로 붙일지는 여전히 사람이 정한다. 또한 C4의 영어 판정과 heuristic filtering은 넓은 web text를 다루는 대신 언어·표현의 편향을 만들 수 있다.

결과는 당시 benchmark와 training budget 안에서 얻은 경험적 결론이다. Denoising objective끼리 차이가 작았다는 결과나 encoder-decoder가 가장 좋았다는 결과를 모든 규모와 task에 일반화할 수는 없다. 11B의 최고 성능도 unified interface, training recipe와 대규모 계산이 묶인 결과이므로 한 요소의 공으로 돌리면 안 된다.

“모든 것을 text로 만들었다”는 구호보다 중요한 것은 입출력 규격을 먼저 통일했다는 점이다. 덕분에 architecture, objective, data와 scale을 같은 기준에서 비교할 수 있었고, 범용성이라는 주장도 개별 benchmark의 모음이 아니라 하나의 재사용 가능한 pipeline으로 시험할 수 있었다.

## 참고 자료

- [Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer](https://arxiv.org/abs/1910.10683)
