---
layout: single
title: "GPT-2 논문 리뷰 - 언어 모델 안에서 Task를 학습한다는 가설"
date: 2026-09-02 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/09/02/gpt2-model-sizes.webp
---

GPT-2는 task별 supervised training 없이, 다양한 web text로 학습한 language model 하나가 여러 NLP task를 수행할 수 있는지 시험했다. [GPT-1](/papers/gpt-1-pretraining-finetuning/)이 pretrained representation을 task마다 fine-tuning했다면, GPT-2는 task 자체도 자연어 sequence 안에서 암묵적으로 학습될 수 있다는 가설로 나아간다.

## Language model 안에 task가 들어 있다는 가설

자연어에는 번역문, 질문과 답, 요약, 대화처럼 입력과 출력의 관계를 보여주는 사례가 이미 섞여 있다. 충분히 다양한 text에서 다음 token을 예측한 model이라면 $p(\text{output}\mid\text{input},\text{task})$도 함께 학습할 수 있고, 추론 때 알맞은 text 형식만 제공하면 parameter update 없이 task를 수행할 수 있다는 것이 논문의 출발점이다.

여기서 `unsupervised multitask learner`는 task label로 구성된 multi-task dataset을 직접 학습하지 않았다는 뜻이다. Web text 안의 형식과 관계가 사실상 암묵적인 supervision으로 작동하므로, 아무 구조도 없는 data에서 task가 저절로 생겼다는 의미는 아니다.

## WebText와 1.5B scaling

OpenAI는 2017년 12월 이전 Reddit에서 karma 3점 이상을 받은 외부 link를 따라가 약 800만 document, 40GB 규모의 WebText를 구성했다. 사람이 공유하고 지지한 page를 출발점으로 삼아 일반적인 web crawl보다 품질을 높이려 했고, benchmark contamination을 줄이기 위해 Wikipedia 문서는 제외했다.

Tokenization은 byte-level BPE를 사용한다. Unicode text를 byte 단위에서 빠짐없이 표현하면서 자주 등장하는 byte sequence는 하나의 token으로 합치며, vocabulary size는 50,257이다.

![GPT-2 네 가지 모델 크기의 layer, hidden dimension과 parameter 수](/assets/images/2026/09/02/gpt2-model-sizes.webp)

구조는 GPT-1과 같은 decoder-only Transformer를 키운 형태다.

- 네 가지 model size를 비교하며 가장 큰 model을 1.5B parameter까지 확장
- Context window 1,024 token
- LayerNorm을 각 sub-block 앞에 배치하고 마지막에 LayerNorm 추가
- Residual path가 깊어질 때 initialization scale을 $1/\sqrt{N}$로 조정

Model이 커질수록 WebText의 validation loss와 여러 downstream 성능이 함께 좋아졌고, 가장 큰 model도 WebText에 완전히 fit하지 못했다. 논문은 이를 architecture가 한계에 도달했다기보다 model size와 data를 더 키울 여지가 남았다는 신호로 해석했다.

## Zero-shot은 어떻게 작동했나

![GPT-2 모델 크기에 따른 reading comprehension, 번역, 요약과 질의응답 성능](/assets/images/2026/09/02/gpt2-zero-shot-tasks.webp)

평가할 때는 model parameter를 갱신하지 않지만, task를 model이 알아볼 수 있는 text 형식으로 바꾸는 작업은 필요하다. 번역은 `English sentence = French sentence` 형태의 예시를 context에 넣고, 요약은 article 끝에 `TL;DR:`을 붙인다. Natural Questions에서는 짧은 답변 형식을 알려주기 위해 question-answer example을 먼저 제공한다. 따라서 논문이 말하는 zero-shot은 주로 “task별 gradient update가 없다”는 뜻이며, input format이나 in-context example까지 없다는 뜻은 아니다.

![GPT-2가 별도 학습 없이 평가한 언어 모델링 데이터셋 결과](/assets/images/2026/09/02/gpt2-zero-shot-language-modeling.webp)

Language modeling에서는 평가한 8개 dataset 중 7개에서 별도 학습 없이 당시 최고 성능을 냈다. Children's Book Test, LAMBADA처럼 문맥에서 빠진 단어를 맞히는 task도 scale이 커질수록 좋아졌다. 이는 다음 token 예측과 평가 형식이 가까운 task에서 가장 강한 결과가 나왔다는 의미이기도 하다.

반면 reading comprehension, summarization, translation과 question answering은 무작위 수준을 넘었어도 supervised 최고 성능과는 거리가 있었다. Natural Questions의 전체 정확도는 4.1%였고, model confidence가 가장 높은 1%의 질문에서는 63.1%였다. Model 내부의 confidence가 아는 질문과 모르는 질문을 어느 정도 가르지만, 전체 coverage를 유지하면서 신뢰할 만한 QA를 수행한 결과는 아니다.

## 잘된 것과 아직 안 된 것

긴 text는 짧은 문장보다 훨씬 자연스러워졌지만, 같은 내용을 반복하거나 앞에서 만든 world state와 모순되고 topic이 갑자기 바뀌는 문제가 남았다. 문장 표면의 통계를 오래 이어 가는 능력과 일관된 세계 model을 유지하는 능력은 같지 않았다.

이 결과는 다음 질문도 남긴다.

- Children's Book Test의 정답률만으로 문맥 추론과 대규모 corpus에서 얻은 pattern matching을 구분할 수는 없다. 단어와 관계를 바꾼 counterfactual·adversarial 평가가 추가로 필요하다.
- Language-model perplexity가 낮아지는 흐름은 여러 task의 향상과 함께 나타났지만 일대일 관계는 아니다. Tokenization, domain 차이와 prompt format 때문에 perplexity가 좋아져도 특정 downstream 능력은 거의 오르지 않을 수 있다.
- Reddit의 karma를 품질 filter로 사용하면 특정 community에서 공유되고 영어권 이용자가 선호한 page가 과대표집된다. 낮은 품질을 거르는 동시에 popularity, 시기와 demographic bias를 data에 넣는다.

## GPT-1에서 한 걸음 더

GPT-1에서 중요했던 것은 “사전학습한 representation을 어디에 fine-tuning할 것인가”였다. GPT-2는 한 단계 더 나아가 “task도 language modeling data 안에 이미 들어 있지 않은가”라고 물었다. 여러 task의 성능은 아직 낮았지만, language modeling을 크게 하면 별도로 가르치지 않은 능력도 함께 나타날 수 있다는 이후 scaling 흐름의 전조가 보였다.

읽을 때는 task description이나 example이 들어간 text까지 zero-shot이라고 부르는 서술이 다소 혼란스러웠다. GPT-2의 기준은 parameter update의 유무에 있고, prompt 안에 demonstration이 몇 개 있는지는 엄격히 분리하지 않았다. 이후 GPT-3가 zero-shot, one-shot, few-shot을 명시적으로 나누면서 이 구분이 선명해졌다.

이 논문의 성과를 “GPT-2가 모든 task를 학습 없이 풀었다”로 요약하면 과하다. 더 정확한 의의는 하나의 autoregressive objective가 language modeling뿐 아니라 여러 conditional task의 공통 기반이 될 가능성을 scale과 zero-shot 평가로 드러냈다는 데 있다.

## 참고 자료

- [Language Models are Unsupervised Multitask Learners](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf)
