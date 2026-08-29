---
layout: single
title: "GPT-1 논문 리뷰 - 사전학습과 Downstream Fine-tuning의 시작"
date: 2026-08-31 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/08/31/gpt1-task-input-transformations.webp
---

GPT-1은 대규모 unlabeled text로 Transformer language model을 먼저 학습한 뒤, labeled data가 있는 downstream task에 맞게 model 전체를 fine-tuning하는 두 단계 framework를 제시했다. 지금은 익숙한 방식이지만, task마다 model을 처음부터 학습하던 흐름에서 하나의 pretrained representation을 여러 task의 출발점으로 재사용했다는 데 의미가 있다.

## 사전학습한 언어 모델을 task로 옮기기

### Unsupervised pre-training

Corpus $U=\{u_1,\ldots,u_n\}$에서 이전 token들을 보고 다음 token의 likelihood를 최대화한다.

$$
L_1(U)=\sum_i \log P(u_i\mid u_{i-k},\ldots,u_{i-1};\Theta)
$$

모델은 causal mask가 있는 12-layer Transformer decoder다. Hidden size 768, attention head 12개로 약 117M parameter를 가지며, [원래 Transformer](/papers/attention-is-all-you-need/)의 decoder stack에서 encoder-decoder attention을 제거한 형태다. 약 7,000권의 책으로 구성된 BooksCorpus에서 다음 token을 예측하며 일반적인 언어 표현을 먼저 학습했다.

### Supervised fine-tuning

각 labeled example의 입력 token sequence $x^1,\ldots,x^m$을 pretrained model에 넣고, 마지막 위치의 final hidden state $h_l^m$으로 label을 예측한다.

$$
P(y\mid x^1,\ldots,x^m)=\operatorname{softmax}(h_l^m W_y)
$$

$$
L_2(C)=\sum_{(x,y)}\log P(y\mid x^1,\ldots,x^m)
$$

Fine-tuning에서는 task loss에 language modeling objective를 보조 loss로 더한다.

$$
L_3(C)=L_2(C)+\lambda L_1(C)
$$

이 auxiliary objective는 labeled task를 학습하는 동안 pretrained representation을 유지하고 수렴과 일반화를 돕는다. 다만 GPT-1은 하나의 model이 prompt만 보고 곧바로 여러 task를 수행하는 방식이 아니다. Task마다 labeled dataset과 별도의 fine-tuning이 필요하다.

## 서로 다른 task를 한 sequence로 만드는 법

![GPT-1이 분류, 함의, 유사도와 객관식 입력을 하나의 token sequence로 바꾸는 방법](/assets/images/2026/08/31/gpt1-task-input-transformations.webp)

GPT-1은 model architecture를 task마다 바꾸는 대신 Start, Delimiter, Extract token으로 입력 구조를 하나의 token sequence 안에 표현한다.

- Classification: text 뒤에 Extract token을 붙이고 그 위치의 representation으로 label을 예측한다.
- Entailment: premise와 hypothesis를 delimiter로 구분하고 마지막 Extract representation을 분류한다.
- Similarity: 두 문장의 순서를 바꾼 두 sequence를 각각 처리한 뒤 representation을 합친다.
- Multiple choice: context와 각 answer candidate를 연결해 candidate별 score를 비교한다.

논문에서 question answering은 BERT의 SQuAD처럼 answer span의 start와 end를 찾는 문제가 아니다. RACE 같은 multiple-choice QA에서 document, question과 각 answer candidate를 하나의 sequence로 만들고 후보별 score를 계산한다.

이 방식은 task-specific network를 크게 줄였지만 input serialization까지 없앤 것은 아니다. 어떤 field를 어떤 순서로 놓고 어디에 delimiter를 둘지는 사람이 task마다 정한다. 이후 T5가 모든 입출력을 text-to-text로 통일하고, GPT-2와 GPT-3가 task description과 example까지 context 안으로 옮기는 흐름의 초기 형태로 볼 수 있다.

## 무엇이 실제로 효과였나

![사전학습 없이 학습한 Transformer와 GPT-1 전이학습 결과 비교](/assets/images/2026/08/31/gpt1-pretraining-transfer.webp)

같은 Transformer를 사전학습 없이 downstream task에서 바로 학습하면 LSTM baseline보다도 낮은 결과가 나왔다. 반면 language model pre-training 뒤 fine-tuning한 model은 평가한 12개 dataset 중 9개에서 당시 최고 성능을 냈다. 성능의 핵심은 Transformer라는 구조만 가져온 것이 아니라, 대규모 unlabeled corpus에서 학습한 parameter를 task 학습의 초기값으로 사용한 데 있었다.

큰 downstream dataset에서는 auxiliary language-model loss가 도움이 됐지만, 작은 dataset에서는 오히려 성능을 낮출 수도 있었다. 한편 pretraining update가 진행될수록 downstream task의 zero-shot heuristic 성능도 대체로 함께 올랐다. 이는 next-token prediction이 단순한 표면 통계만이 아니라 sentiment, 문장 유사도, entailment 등에 재사용할 수 있는 feature를 내부에 만든다는 근거였다. 다만 이 분석만으로 각 layer가 어떤 linguistic feature를 인과적으로 담당한다고 확정할 수는 없다.

## 이 논문이 연 흐름

전체 framework는 unlabeled pre-training과 labeled fine-tuning을 이어 붙이므로 넓게 보면 semi-supervised learning에 속한다. 논문이 `unsupervised pre-training`이라고 부르는 것은 첫 단계의 data에 사람이 붙인 task label이 없다는 뜻이다.

GPT-1의 learned positional embedding과 context window는 학습한 최대 길이에 묶인다. 원래 model은 512-token context를 사용하므로 더 긴 sequence를 그대로 외삽하는 구조가 아니다. 이후 GPT 계열이 context length와 positional method를 계속 바꾼 이유 중 하나다.

[BERT](/papers/bert/)는 이 전이학습 recipe를 양방향 encoder와 masked language modeling으로 가져갔다. GPT-2는 반대로 task별 fine-tuning 없이 language modeling만으로 여러 task가 나타나는지를 시험했고, GPT-3는 parameter를 고정한 채 context의 instruction과 example로 task에 적응하는 in-context learning까지 밀어붙였다.

논문 계보를 따라 읽으면서 model architecture의 변화만큼, 사전학습한 지식을 downstream task로 옮기는 방식이 달라지는 과정이 재미있었다. GPT-1의 중요한 지점은 오늘날의 chat model을 이미 완성했다는 데 있지 않다. 대규모 language model pre-training을 공통 기반으로 두고 task 학습을 그 위에 얹는 순서를 분명히 만든 출발점에 가깝다.

## 참고 자료

- [Improving Language Understanding by Generative Pre-Training](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf)
