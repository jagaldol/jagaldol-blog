---
layout: single
title: "GPT-3 논문 리뷰 - Fine-tuning 없이 In-context Learning하기"
date: 2026-09-06 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/09/06/gpt3-training-data.webp
---

GPT-3는 175B parameter의 autoregressive language model을 task별로 fine-tuning하지 않고, instruction과 example을 context에 넣는 것만으로 여러 task에 적응시킨다. 핵심은 [GPT-2](/papers/gpt-2-unsupervised-multitask-learning/)보다 약 100배 큰 model이라는 점만이 아니라, model parameter를 고정한 채 context를 임시 학습 공간처럼 사용하는 in-context learning을 대규모로 시험했다는 데 있다.

## Model을 다시 학습하지 않고 task를 바꾸다

[BERT](/papers/bert/)와 [T5](/papers/t5-text-to-text-transfer-learning/)는 pretrained model을 downstream task의 labeled data로 다시 학습한다. GPT-3는 gradient update 없이 prompt 안의 자연어 설명과 demonstration만 바꿔 task를 전환한다.

- Zero-shot: task description만 제공
- One-shot: task description과 example 하나 제공
- Few-shot: task description과 context가 허용하는 여러 example 제공

이때 model이 example을 통해 새로운 parameter를 얻는 것은 아니다. 같은 fixed parameter가 앞선 token을 조건으로 다음 token 분포를 바꾸는 것이므로, “학습”은 context 안에서만 유지되고 prompt가 끝나면 사라진다. 논문은 사전학습 중 반복해서 본 pattern과 task 구조가 이 동작을 가능하게 하는 일종의 meta-learning이라는 가설을 제시한다.

GPT-2는 parameter update가 없으면 context에 example이 있어도 넓게 zero-shot으로 불렀다. GPT-3가 zero-, one-, few-shot을 example 개수로 나누면서 이후 prompt 기반 평가의 용어가 훨씬 분명해졌다.

## 175B까지 키운 조건

![GPT-3가 사용한 Common Crawl, WebText, Books와 Wikipedia의 학습 비율](/assets/images/2026/09/06/gpt3-training-data.webp)

Filtered Common Crawl을 중심으로 WebText2, Books1, Books2와 Wikipedia를 섞어 약 300B token을 학습했다. 품질이 높지만 작은 corpus는 여러 번 반복하고, Common Crawl은 상대적으로 낮은 비율로 sampling했다. Near-duplicate 제거와 benchmark contamination 검사를 적용했지만 모든 test overlap을 제거하지는 못했다.

가장 큰 model은 96 layer, hidden size 12,288, attention head 96개이며 context window는 2,048 token이다. GPT-2와 같은 decoder-only Transformer 계열에 dense attention과 locally banded sparse attention을 번갈아 사용했다.

논문은 125M부터 175B까지 여덟 개 model size를 비교한다. Scaling law에 따라 language-model loss가 매끄럽게 낮아지는지뿐 아니라, 같은 증가가 zero-, one-, few-shot task 성능으로 이어지는지를 함께 본다.

## Scale이 잘 통하는 곳과 아닌 곳

Language modeling, closed-book question answering, translation, cloze, commonsense reasoning과 SuperGLUE 계열에서 model size가 커질수록 few-shot 성능이 대체로 좋아졌다. 일부 task에서는 fine-tuned model의 당시 최고 성능에 근접하거나 넘어섰지만, 여러 task에서는 여전히 큰 격차가 남았다. “하나의 model이 모든 benchmark를 이겼다”가 아니라, parameter update 없는 한 model이 서로 다른 task에서 경쟁력 있는 결과를 내기 시작했다는 쪽이 정확하다.

![GPT-3의 zero-shot, one-shot과 few-shot 산술 성능](/assets/images/2026/09/06/gpt3-arithmetic-table.webp)

![GPT-3 모델 크기에 따른 산술 문제 정확도](/assets/images/2026/09/06/gpt3-arithmetic-scaling.webp)

산술에서는 두 자리 덧셈과 뺄셈처럼 training text에 비슷한 pattern이 많고 절차가 짧은 문제는 scale과 함께 크게 좋아졌다. 자릿수가 늘어난 곱셈이나 여러 연산을 조합한 문제는 훨씬 약했다. 일부 결과는 작은 model에서 조금씩 오르기보다 13B와 175B 사이에서 큰 폭으로 뛰었다.

이 결과는 scale이 새로운 행동을 드러낼 수 있음을 보여주지만, 안정적인 계산 algorithm을 습득했다는 증거와는 다르다. 숫자 자릿수나 표현을 바꿨을 때 일반화가 무너지면 memorized pattern과 실제 연산 절차를 분리하기 어렵다.

## In-context learning이 남긴 문제

- Example 선택, 순서, label 표현과 prompt wording에 결과가 민감하다. 같은 task라도 context 구성에 따라 성능 차이가 크다.
- Few-shot example은 parameter를 갱신하지 않으므로 training data에 없던 지식을 지속적으로 습득하거나 다음 session에 보존하지 못한다.
- Benchmark test example과 유사한 text가 거대한 pretraining corpus에 섞였을 수 있다. 논문도 contamination을 조사했지만 175B model을 data 정리 후 다시 학습해 영향을 완전히 분리하지는 못했다.
- 긴 생성에서는 반복과 자기모순이 나타나고, 상식적인 물리와 world state를 안정적으로 추적하지 못한다.
- Web data의 사회적 편향과 유해한 pattern을 함께 학습한다. 더 큰 model이 자동으로 더 안전하거나 truthfulness가 높아지는 것은 아니다.
- 175B 학습과 추론은 큰 compute·energy·serving cost를 요구한다. Few-shot은 example까지 context에 넣으므로 inference 비용도 커진다.

## 이 논문 이후

GPT-3의 중요한 변화는 모든 task에서 최고 점수를 얻었다는 데 있지 않다. 하나의 pretrained language model이 자연어와 example을 interface로 삼아, parameter를 바꾸지 않고 여러 task에 재사용될 수 있음을 규모 있게 보여줬다. Task마다 model artifact를 따로 만드는 방식에서 prompt로 behavior를 지정하는 방식으로 중심이 이동했다.

다만 원래 GPT-3는 instruction을 안정적으로 따르도록 별도 학습한 assistant가 아니다. Zero-shot instruction만으로는 task를 오해하거나 형식을 어기기도 한다. 이후 instruction tuning과 human feedback을 이용한 alignment는 base model의 잠재 능력을 사용자가 요구한 행동으로 끌어내는 별도 단계로 발전했다.

읽고 남은 결론도 두 갈래다. Scale은 단순한 loss 감소를 넘어 in-context learning 같은 사용 방식을 실제로 강하게 만들었다. 동시에 산술, 장기 일관성과 상식 추론의 실패는 language pattern을 잘 이어 가는 것과 안정적인 계산·세계 model을 갖는 것이 다르다는 점을 보여준다. GPT-3는 scale만으로 문제가 끝났다는 논문이 아니라, scale을 올렸을 때 무엇이 나타나고 무엇이 그대로 남는지를 드러낸 논문에 가깝다.

## 참고 자료

- [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165)
