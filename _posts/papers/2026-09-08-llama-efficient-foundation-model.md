---
layout: single
title: "LLaMA 논문 리뷰 - 작은 모델을 더 오래 학습하는 이유"
date: 2026-09-08 20:00:00 +0900
categories: papers
header:
  teaser: /assets/images/2026/09/08/llama-pretraining-data.webp
---

LLaMA는 7B, 13B, 33B, 65B 규모의 decoder-only foundation language model이다. 공개적으로 구할 수 있는 data만 사용하면서 작은 model을 기존의 compute-optimal 기준보다 훨씬 오래 학습해, inference cost가 낮은 model도 훨씬 큰 model과 경쟁할 수 있음을 보였다.

## 더 작게 만들고 더 오래 학습하기

Chinchilla scaling law는 정해진 training compute 안에서 parameter 수와 token 수를 함께 늘리는 compute-optimal 지점을 찾는다. LLaMA가 최적화한 조건은 조금 다르다. 실제 배포에서는 training을 한 번 하고 inference를 반복하므로, training compute를 더 쓰더라도 작은 model의 성능을 충분히 올리면 이후 serving cost를 줄일 수 있다.

이 관점에서 7B·13B model은 1T token, 33B·65B model은 1.4T token을 학습했다. LLaMA-7B의 validation loss는 1T token을 지난 뒤에도 계속 낮아졌다. “작은 model은 적은 data만 봐야 효율적이다”라는 당시 compute-optimal 해석과 달리, inference 효율을 목표로 하면 작은 model을 훨씬 오래 학습할 가치가 있다는 결과다.

논문은 LLaMA-13B가 대부분의 평가에서 [GPT-3](/papers/gpt-3-in-context-learning/) 175B를 능가하고, LLaMA-65B는 Chinchilla 70B와 PaLM 540B에 견줄 만한 결과를 냈다고 보고한다. 이 비교의 핵심은 13B가 언제나 175B보다 낫다는 일반 법칙이 아니라, parameter 수만으로 성능과 inference 효율을 판단할 수 없다는 데 있다.

## 공개 data로 만든 학습 recipe

![LLaMA가 사용한 CommonCrawl, C4, GitHub, Wikipedia, Books, arXiv와 Stack Exchange 비율](/assets/images/2026/09/08/llama-pretraining-data.webp)

CommonCrawl 67%와 C4 15%를 중심으로 GitHub, Wikipedia, Books, arXiv와 Stack Exchange를 섞었다. Source마다 deduplication, language filter, heuristic quality filter와 formatting 정제를 다르게 적용했다. `공개 data`는 접근 가능한 source로 재현 가능성을 높인다는 뜻이지, 자동으로 편향이나 저작권 문제가 없는 data라는 뜻은 아니다.

Tokenizer는 SentencePiece의 BPE를 사용한다. 숫자를 개별 digit로 분리하고, vocabulary에 없는 문자는 UTF-8 byte로 fallback한다.

Architecture는 [Attention Is All You Need](/papers/attention-is-all-you-need/)의 Transformer decoder를 바탕으로 당시 효과가 검증된 구성요소를 결합했다.

- Pre-normalization: 각 sub-layer 입력에 RMSNorm 적용
- ReLU 대신 SwiGLU activation 사용
- Absolute positional embedding 대신 Rotary Positional Embedding(RoPE) 사용
- Causal multi-head self-attention과 2,048-token context
- FlashAttention과 activation recomputation으로 memory 사용량 절감

65B model은 A100 80GB GPU 2,048개에서 약 21일 동안 1.4T token을 학습했다. 작은 inference model을 얻기 위해 training cost 자체를 줄인 것이 아니라, 큰 일회성 training compute를 감수하고 반복되는 inference 비용을 낮추는 선택이다.

## 결과를 읽는 범위

평가는 common-sense reasoning, closed-book question answering, reading comprehension, 수학 추론, code generation과 MMLU를 zero-shot과 few-shot 조건에서 다룬다. LLaMA-13B와 65B는 여러 평가에서 훨씬 큰 model과 경쟁했지만 모든 영역에서 같은 우위가 나온 것은 아니다.

MMLU에서는 LLaMA-65B도 Chinchilla와 PaLM보다 낮았다. 논문은 pretraining mixture에서 book과 academic text의 비중이 상대적으로 낮다는 점을 한 원인으로 본다. 수학·code·지식 task의 결과도 model size뿐 아니라 corpus 구성과 중복, prompt format, 비교 model의 학습 조건에 영향을 받는다.

Instruction fine-tuning을 적용한 LLaMA-I는 MMLU 성능이 크게 올랐지만 당시 instruction-tuned 최고 결과에는 미치지 못했다. 원래 LLaMA는 사용자의 지시를 안정적으로 따르도록 학습된 chat model이 아니라 next-token prediction으로 사전학습한 base model이라는 구분이 필요하다.

## 책임 있는 AI 평가와 한계

- Toxicity: model size가 커질수록 일부 toxicity 지표도 함께 증가했다.
- Bias: CrowS-Pairs에서 religion, age, gender 등 여러 범주의 사회적 bias가 남았다.
- Truthfulness: TruthfulQA에서 규모에 따른 개선이 제한적이었고, 잘못된 전제를 그대로 따르거나 그럴듯한 오류를 만드는 문제가 남았다.
- Data contamination: 공개 benchmark와 유사한 text가 pretraining corpus에 섞일 수 있어 일부 결과를 순수한 일반화로만 해석하기 어렵다.
- Context와 language: 2,048-token context로 긴 문서를 직접 처리하기 어렵고, corpus에서 영어 비중이 높아 비영어권 language coverage가 제한적이다.

## Open model 생태계로 이어진 부분

공개 data로 학습했다는 사실과 model 자체가 완전한 open source라는 말은 같지 않다. Original LLaMA weight는 연구자에게 신청 기반으로 제공됐고, 비상업적 연구 중심의 별도 license가 적용됐다. 이후 weight가 비공식적으로 퍼지고 Alpaca·Vicuna 같은 instruction-tuned derivative, LoRA·QLoRA 같은 parameter-efficient fine-tuning이 결합되면서 local·community model 생태계가 빠르게 커졌다.

LLaMA 2와 3는 data 규모, context, tokenizer, grouped-query attention, post-training과 배포 license를 계속 바꿨다. 따라서 이후 Llama 계열의 개방성과 성능을 2023년 original LLaMA 논문에 그대로 소급하면 안 된다.

## 결론

읽고 가장 크게 남은 것은 “parameter가 더 큰 model이 항상 더 효율적이다”라는 단순한 구도가 깨졌다는 점이다. 학습량, data quality와 inference budget을 함께 보면 작은 model을 오래 학습하는 편이 실제 사용 비용에 유리할 수 있다.

LLaMA의 의의도 완전히 새로운 block 하나보다 이 조합에 있다. 공개 data, 오래 학습한 비교적 작은 decoder model, 검증된 architecture recipe가 강한 base model을 만들었고, 이 model이 이후 fine-tuning 도구와 community 연구가 빠르게 실험되는 공통 기반이 됐다.

## 참고 자료

- [LLaMA: Open and Efficient Foundation Language Models](https://arxiv.org/abs/2302.13971)
