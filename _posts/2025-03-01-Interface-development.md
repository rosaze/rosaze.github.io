---
layout: post
title: "Interface development"
date: 2025-03-01 09:00:00 +0900
categories: interface
project: webtoonizer
---

## 인터페이스

#### 사용자에게 선택권을 주고, 목적에 맞는 도구를 제공

- 텍스트-웹툰 변환 인터페이스는 사용자 친화적으로 설계되었다.
- Streamlit을 활용해서 직관적이고 깔끔한 웹 인터페이스를 구현하였다.

### main page

- **"Visualize Story Text" (스토리 텍스트 시각화)** - 소설, 스크립트, 이야기 등을 웹툰 형식으로 변환하는 옵션. 이 옵션은 주로 서사적인 텍스트를 다룰 때 사용하며, 캐릭터와 스토리 흐름을 시각화하는 데 최적화
- **"Visualize Educational/Scientific Text"** (교육/과학 텍스트 시각화) - 교육 자료, 과학적 개념, 프로세스 등을 시각적으로 설명하는 옵션. 복잡한 개념이나 원리를 이해하기 쉬운 그림으로 바꿔주는 기능.

  <img src="images/webtoon/int4.png" width="550" height="340">

메인 화면에서는 크게 두 가지 입력 방식을 제공:

1. 직접 입력 - 사용자가 텍스트를 바로 입력할 수 있는 텍스트 영역
2. 파일 업로드 - TXT, PDF, DOCX 등 다양한 형식의 파일을 업로드할 수 있는 기능

### 시각화 방법 선택

<img src="images/webtoon/int5.png" width="600" height="320">

### 텍스트 입력 후에는 생성할 웹툰의 스타일과 분위기를 선택할 수 있다:

- 스타일 옵션: Minimalist(미니멀), Pictogram(픽토그램), Cartoon(카툰), Webtoon(웹툰), Artistic(예술적)
- 분위기 옵션: Casual(일상적), Tense(긴장감), Serious(진지함), Warm(따뜻함), Joyful(즐거움)
- 구도 옵션: 배경과 캐릭터 강조, 클로즈업 샷, 인터랙티브 구도, 풍경 중심
- (+) 캐릭터 설명을 추가할 수 있는 선택적 입력란과 생성할 컷 수(1~4컷), 이미지 비율(1:1, 16:9, 9:16) 등을 설정
  <br><br>
  <img src="images/webtoon/int7.png" width="600" height="320">

#### 교육/과학 텍스트를 위한 특화된 모드

  <img src="images/webtoon/int6.png" width="600" height="320">

- 일반 스토리가 아닌 '설명하기', '비교하기', '과정 보여주기', '원리 설명하기' 같은 시각화 방식을 선택할 수 있다.

  > => 복잡한 개념을 쉽게 시각화하는 데 유용

- **Explain(설명하기)**: 개념을 명확하게 전달하는 데 초점을 맞춤
  - Simple and intuitive illustrations(간단하고 직관적인 일러스트)
  - Highlight key elements(핵심 요소 강조)
- **Compare(비교하기)**: 차이점이나 특징을 대조하는 방식
  - Present in parallel structure(병렬 구조로 표현)
  - Emphasize differences(차이점 강조)
- **Process Demonstration(과정 보여주기)**: 단계별 변화를 설명
  - Sequential flow representation(순차적 흐름 표현)
  - Clearly visualize relationships(관계를 명확하게 시각화)
- **Principle Explanation(원리 설명하기)**: 사물이 어떻게 작동하는지 시각화
  - Depict structures and relationships(구조와 관계 묘사)
  - Diagram mechanisms(메커니즘 다이어그램)

## 이미지 생성 결과 실험 모음

<div style="display: flex; gap: 10px;">
      <img src="images/webtoon/int1.png" width="215" height="350">
      <img src="images/webtoon/int2.png" width="215" height="350">
      <img src="images/webtoon/int3.png" width="215" height="350">
       
  </div>
  <br>
  <div style="display: flex; gap: 10px;">
      <img src="images/webtoon/int8.png" width="305" height="180">
      <img src="images/webtoon/int9.png" width="305" height="180">
   
  </div>

### CLIP 점수 예시

<div style="display: flex; gap: 10px;">
 <img src="images/webtoon/clip.png" width="280" height="330">
  <img src="images/webtoon/int10.png" width="280" height="330">
</div>
- 첫번째 이미지는 우리 프로젝트에서 "폭풍의 언덕" 텍스트를 기반으로 생성된 웹툰/이미지에 대한 CLIP 분석 결과를 보여준다.

- 품질 점수는 1.00으로, 이는 가능한 가장 높은 CLIP 유사도 점수를 나타낸다. 시스템은 이 점수를 기반으로 콘텐츠를 "고품질"로 분류했으며, 이는 코드에서 설정한 임계값(아마도 0.7 이상)을 충족했기 때문이다.

- 두 번째 이미지:
  - Device: cpu - 현재 CLIP 모델이 CPU에서 실행되고 있다는 걸 나타냄. 일반적으로 GPU가 있다면 더 빠른 처리가 가능하지만, 현재는 CPU로 동작 중
  - Model: openai/clip-vit-base-patch32 - 사용 중인 CLIP 모델의 정확한 버전. 이 모델은 OpenAI에서 개발한 ViT(Vision Transformer) 기반 모델로, 텍스트와 이미지 간의 유사도를 평가하는 데 사용.
  - Average CLIP Score: 1.00 - 생성된 모든 이미지의 평균 CLIP 점수. 1.00은 최대 점수로, 텍스트 프롬프트와 생성된 이미지 간의 의미적 일치도가 완벽하다는 걸 의미.
  - Total Generation Time: 72.5s - 모든 이미지를 생성하는 데 걸린 총 시간.

#### 위 패널은 사용자에게 시스템의 성능과 품질을 한눈에 파악할 수 있게 해주는 역할이다
