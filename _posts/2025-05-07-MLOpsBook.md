---
layout: post
title: "요즘 우아한 AI 개발발"
date: 2025-05-07
categories: [cv-nlp-pe] # 또는 [research, nlp] 또는 [research, computer-vision]
use_math: true
---

## 프롬프트 전략

1. 응답 최적화 : json 형식으로 응답을 설계
2. 생성 지식 프롬프트 -> LLM 이 자체적으로 생성한 지식을 포함하여 질문에 대한 응답을 강화하는 기법. like chain-of-thoughts
   - 일관성 향상 전략
3. 빠른 응답을 위한 Latency 최적화 : 작성된 프롬프트의 내용을 줄이고, 응답을 한 문장으로만 제한하는 등
4. 이미지를 텍스트 프롬프트보다 먼저 배치

> 생성형 AI 만으로 문제를 해결하기 어렵다면: 머신러닝 모델과의 하이브리드 접근 & 파인튜닝을 권장!

### 배민 선물하기 메세지 작성
