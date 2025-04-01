---
layout: post
title: "Zero-Shot Text-to-Image Generation"
date: 2025-04-01
categories: [cv-nlp-pe] # 또는 [research, nlp] 또는 [research, computer-vision]
use_math: true
---

> Abstract: T2I 는 전통적으로 고정된 데이터셋에서 더 나은 모델링 가정을 찾는 데 중점을 두었다. 이러한 가정은 복잡한 architecture, 보조 손실 또는 객체 부위 라벨이나 세분화 마스크와 같은 부가 정보를 훈련 중에 제공하는 것을 포함할 수 있다.우리는 텍스트와 이미지 토큰을 단일 데이터 흐름으로 자동회귀적으로 모델링하는 Transformer을 기반으로 한 간단한 접근 방식을 설명한다. 충분한 데이터와 규모를 제공하면, 이 접근 방식은 이전의 도메인 특화 모델들과 비교하여 제로샷 방식에서 경쟁력을 갖는다.

# Introduction

현대 머신러닝의 텍스트에서 이미지 합성 접근법은 Mansimov et al. (2015)의 연구에서 시작되었다. 이 연구는 **gen ai 를 image caption에 조건화시켜, 새로운 시각적 장면을 생성할 수 있음을 보여주었다.** 이후 Reed et al. (2016b)는 recurrent variational auto-encoder 대신 생성적 적대 신경망(GAN) 을 사용하면 이미지의 충실도가 향상된다는 것을 입증했다. 이 연구는 시스템이 인식 가능한 속성을 가진 객체를 생성할 뿐 만 아니라, 사전 학습되지 않은 범주에 대해서도 제로샷 일반화가 가능하다는 것을 보여주었다.

<details>
<summary><mark>what is GAN?</mark>
</summary>
<div markdown="3">
생성적 적대 신경망 GAN 은 딥러닝 모델 구조로 ,두 개의 신경망이 서로 경쟁하며 학습하는 방식.생성자는 랜덤 노이즈로부터 가짜 데이터를 생성 --> 판별자(Discriminator) 은 입력받은 데이터가 진짜인지 가짜인지 구분. 두 네트워큰느 서로 경쟁적으로 학습한다. 생성자는 판별자를 속이기 위해 더 진짜 같은 데이터를 만들려고 하고, 판별자는 진짜와 가짜를더 잘 구분하려고 한다. 이 과정을 통해 생성자는 점점 더 실제 데이터와 유사한 데이터를 생성할 수 있게 된다. 
</div>
</details>

<details>
<summary><mark>what is zero-shot?</mark>
</summary>
<div markdown="3">
AI 모델이 학습 과정에서 한 번도 보지 않은 클래스나 작업에 대해서도 추론할 수 있는 능력을 의미. 예를 들어, "파란 모자를 쓴 고양이"라는 텍스트가 주어졌을 때, 그 조합을 직접 학습하지 않았더라도 이미지를 생성할 수 있다.
</div>
</details>

2017~2020부터는 생성 모델 구조를 개선하였다. Zhang et al., 2017; 2018에서 다중 스케일 생성기를 사용하였고, _Attention_ 기법과 _Auxiliary loss_ 보조 손실을 적용하였다. 텍스트 외에 추가 정보를 활용하여 구조를 개선하였다.
또한 에너지 기반 모델을 제안하여 Nguyen et al. (2017) 당시 방법들보다 샘플 품질을 크게 개선했다. 사전학습된 분류 모델을 활용할 수 있고, MS-COCO 캡션 데이터로 텍스트-이미지 생성도 가능함을 보이기 시작했다.
Cho et al. (2020)은 사전학습된 크로스모델 마스킹 언어 모델에 입력을 최적화하는 방식을 제안했다.

_"Recent advances..~"_

최근 발전(transformer & 대규모 모델)에도 여전히 문제점이 존재하는데, **객체 왜곡, 이상한 위치 배치, 배경과 전경의 부자연스러운 혼합** 이다.
다만 새로운 전환점이 있었는데, 텍스트(GPT-2), 이미지(Image GPT), 오디오 등에서 대규모 트랜스포머 모델이 뛰어난 성능을 보여준 것이다.이를 기반으로 텍스트-이미지 생성에도 확장 가능하다는 가능성이 제기됐다.

**이 논문에서 주장하는 핵심 기여는, 인터넷에서 수집한 2억 5천만개의 이미지-텍스트 쌍을 이용해 매개변수 120억 개의 Autoregressive transformer 을 학습시켰고, MS-COCO 에서 레이블 없이도 제로샷 고품질 이미지 생서이 가능하다. 정성 평가에서도 우수하다고 평가됐고, 이미지들끼리의 변환 (image-image translation) 같은 복잡한 작업도 기본적으로 수행 가능하다.**

> 요약: 모델 규모와 데이터셋 규모를 충분히 키우고 , 트랜스포머 기반의 오토레그레시브 생성 방식을 적용하면 텍스트를 기반으로 한 고품질 이미지 생성이 가능하며, 일부 복잡한 기능들도 자연스럽게 모델 안에 통합될 수 있음.

# Method

목표는 transformer 을 학습시켜 텍스트와 이미지 토큰을 하나의 연속된 데이터 스트림으로 autoregressive 하게 모델링하는 것이다. 그러나 이미지 토큰으로 픽셀을 직접 사용하는 경우, 고해상도 이미지를 처리하기 위해 막대한 메모리가 필요하게 된다. 가능도(likelihood) 기반의 목적 함수는 픽셀 간의 단기 의존성(short-range dependencies)을 모델링하는 데 우선순위를 두는 경향이 있으므로, 모델의 용량 대부분이 객체를 시각적으로 인식 가능하게 만드는 저주파 구조보다는 고주파 세부사항을 포착하는 데 사용된다(Salimans et al., 2017).

이러한 문제를 해결하기 위해 2단계 학습 절차를 사용

## 2.1 Stage One: learning the visual Codebook

훈련의 첫 번째 단계에서는 φ와 θ에 관하여 ELB를 최대화
즉 이미지만을 사용하여 dVAE를 훈련시키는 것에 해당.

- 초기 사전 확률 pψ를 K = 8192개 코드북 벡터에 대한 균일 범주형 분포로 설정하고, qφ는 인코더에 의해 출력된 그리드의 동일한 공간 위치에 있는 범주형 분포로 설정.

- ψ가 이산 분포이기 때문에, (gradient)를 사용하여 이를 최대화할 수 없다. (straight-through estimator와 결합된 온라인 클러스터 할당 절차를 사용하여 이 문제를 해결할 수 있다)
- 논문에서는 대신 gumbel-softmax 완화(relaxation)(Jang 등, 2016; Maddison 등, 2016)를 사용하여 qφ에 대한 기대값을 q τ φ에 대한 기대값으로 대체. 여기서 온도 τ → 0으로 갈수록 완화가 더 정확해지고, 완화된 ELB는 지수적으로 가중치가 부여된 반복 평균화를 사용한 Adam을 통해 최대화.
- 다음이 안정적인 훈련에 특히 중요하다는 것을 확인:

  - 인코더 끝과 디코더 시작 부분에 1 × 1 합성곱을 사용하여여 완화 주변의 합성곱에 대한 수용 영역(receptive field) 크기를 줄이면 실제 ELB에 더 잘 일반화된다는 것을 발견.
  - KL 가중치를 β = 6.6으로 증가시키면 더 나은 코드북 사용이 촉진되고 궁극적으로 훈련 종료 시 더 작은 재구성 오류로 이어진다

<details>
<summary><mark>what is KL weigh?</mark>
</summary>
<div markdown="3">
KL 가중치(KL weight)는 변분 오토인코더(VAE)와 같은 생성 모델에서 중요한 하이퍼파라미터.

이는 손실 함수(loss function)에서 두 가지 주요 항목 사이의 균형을 조절:

재구성 손실(Reconstruction Loss): 모델이 얼마나 정확하게 원본 이미지를 재구성하는지 측정.
KL 발산(Kullback-Leibler Divergence): 인코더가 생성하는 잠재 분포와 사전 분포(보통 표준 정규 분포) 사이의 차이를 측정.

KL 가중치(β)는 이 두 항목 사이의 균형을 조절. 논문에서는 이 값을 6.6으로 설정했는데, 이는 KL 항에 더 큰 중요성을 부여한다는 의미입니다. 이렇게 함으로써:

코드북 벡터(codebook vectors)의 사용을 더 균등하게 만들고.잠재 공간이 더 규칙적이고 구조화된 형태를 갖게 하여 결과적으로 모델이 더 좋은 이미지 재구성 능력을 갖게 된다.

이러한 접근 방식은 β-VAE라고도 불리는 기법의 일종으로, 잠재 표현의 품질과 해석 가능성을 향상시키는 데 사용.

</div>
</details>

## 2.2 Stage Two: learning the Prior

**두 번째 단계에서는 φ와 θ를 고정하고, ψ에 관하여 ELB를 최대화함으로써 텍스트와 이미지 토큰에 대한 사전 분포를 학습함.**

여기서 pψ는 120억 파라미터를 가진 희소 트랜스포머로 표현

- 1. 텍스트-이미지 쌍이 주어지면, 최대 256개 토큰을 사용하여 소문자화된 **캡션**을 BPE-인코딩하고, 8,192 어휘 크기로 32 × 32 = 1,024개 토큰을 사용하여 이미지를 인코딩.(이미지 토큰은 gumbel 노이즈를 추가하지 않고 dVAE 인코더 로짓에서 argmax 샘플링을 통해 얻는다)
- 2. 텍스트와 이미지 토큰을 연결하여 단일 데이터 스트림으로 자기회귀적으로 모델링
     _"The transformer is a decoder-only model in which each image token can attend to all text tokens in any one of its 64
     self-attention layers"_

이 모델에서는 세 가지 종류의 self attention mask 가 쓰였는데,

1. 텍스트-텍스트 주의: 표준 인과 마스크(standard causal mask) 사용
2. 이미지-이미지 주의: 행(row), 열(column), 또는 합성곱(convolutional) 주의 마스크 사용
3. 이미지-텍스트 주의: 각 이미지 토큰이 모든 텍스트 토큰에 주의를 기울일 수 있음

- text caption: 256 token 으로 제한, 마지막 텍스트 토큰과 이미지 시작 토큰 사이 패딩을 위해 각 텍스트 위치마다 특별한 패딩 토큰을 학습, 분포 외 캡션에 대해 더 나은 성능을 보임
- 손실 함수: 텍스트 및 이미지 토큰에 대한 교차 엔트로피 손실을 배치 데이터에서 각 종류의 총 수로 정규화. 모델이 이미지 생성에 더 중점을 두도록 가중치 설정
- 최적화:
  - adam optimizer & 반복 평균화 사용
  - resblock 별 그래디언트 스케일링을 통해 안정적인 훈련 보장
  - 32비트 정밀도로 항등 경로의 활성화 + Gradient 저장
  - 비유한 값을 filtering -> Gradient underflow 방지

## 2.3 Data Collection

## 2.3. 데이터 수집

### 주요 내용:

- **초기 실험**:

  - **Conceptual Captions** 데이터셋 사용 (330만 텍스트-이미지 쌍)
  - MS-COCO (Lin 등, 2014)의 확장 버전
  - 최대 12억 파라미터 모델 실험용

- **대규모 데이터셋**:

  - **JFT-300M** 규모에 맞춰 새 데이터셋 구축
  - 인터넷에서 2억 5천만 텍스트-이미지 쌍 수집
  - 포함 내용:
    - Conceptual Captions
    - YFCC100M의 필터링된 부분집합
  - MS-COCO는 제외 (단 YFCC100M을 통해 일부 검증 이미지 포함)

- **주요 사항**:

  - MS-COCO 캡션은 포함되지 않음
  - 일부 MS-COCO 검증 이미지 포함이 결과에 미치는 영향은 미미함
  - 자세한 내용은 부록 C 참조

  - **파라미터 샤딩**:
  - 모델 파라미터를 머신당 8개 GPU에 분할
  - 순전파 시 **all-gather**로 다음 resblock 파라미터 미리 가져옴
  - 역전파 시 **reduce-scatter**로 그래디언트 계산

## 2.4 Mixed-Precision Training

_"Getting the model to train in 16-bit precision past one billion parameters, without diverging, was the most challenging part of this project."_

- 문제 원인: 16비트 그래디언트의 언더플로(underflow)가 불안정성의 주원인.

- 해결 방법: 각 resblock마다 별도의 그래디언트 스케일을 적용해 안정적인 학습 달성(표준 loss scaling으로는 부족). = us

- 결과: 약 85%의 압축률(compression rate)로 대규모 모델 학습 가능(Table 1 참조).

## 2.5 Distributed Optimization

또 다른 차별점 제시 _We were able to drastically reduce [the cost of all-reduce] by compressing the gradients using PowerSGD (Vogels et al., 2019)."_

대규모 분산 학습의 주요 병목 현상(inter-machine 통신 비용)을 해결한 방법을 강조하며, 이후 설명되는 PowerSGD 기반 그래디언트 압축 기술의 핵심 목적을 요약

1. **문제 진단**

클러스터 환경에서 머신 간 대역폭이 머신 내 GPU 대역폭보다 훨씬 낮아, all-reduce 연산이 학습 병목 현상이 됨.

2. **PowerSGD의 핵심 혁신**

   - 기존 uncompressed 그래디언트 통신 → 저랭크(low-rank) 인자 2회 통신으로 대체해 통신 부하 감소.
   - 약 85% 압축률 달성
     $\`\`\`math
\text{Compression Rate} = 1 - \frac{5r}{8d_{\text{model}}}
\`\`\`$

- `r`: 압축 랭크 (예: 512, 640, 896)

- `d_model`: 트랜스포머 활성화 차원 (예: 1920, 2688, 3968)
- `5/8`: GPU당 통신 비용 상수

3. **최적화 기법**
   - 그래디언트를 별도 버퍼가 아닌 에러 버퍼에 누적 저장.
   - Householder 직교화 + identity matrix 작은 배수 추가.
   - 에러 버퍼 및 통신에 커스텀 16비트 포맷 적용.
   - warm-start 대신 고정된 랜덤 가우시안 행렬 사용해도 동일 성능.

## 2.6 Sample Generation

### 주요 방법:

- **대조 학습 모델 기반 리랭킹**:
  - Razavi 등(2019)과 유사한 접근 방식 채택
  - 사전 훈련된 대조 학습 모델(Radford 등, 2021)을 활용해 샘플 평가
  - 주어진 캡션과 후보 이미지에 대해 **호환성 점수** 부여

### 핵심 프로세스:

1. **샘플 생성**:

   - 트랜스포머 모델에서 N개의 초기 샘플 생성 (기본값: N=512)
   - 온도 파라미터 기본값(t=1) 적용 (단, 그림 2 예외)

2. **상위 k개 선택**:
   - 대조 학습 모델이 부여한 점수 기준 상위 k개 이미지 선정
   - N 증가시 성능 향상 효과 확인

### 비교 가능한 기존 연구:

- 언어 유도 검색(Language-guided search) (Andreas 등, 2017)
- 보조 텍스트-이미지 매칭 손실(Xu 등, 2018)과 유사한 개념

### 적용 사항:

- 모든 정성적/정량적 결과에 동일 조건 적용
- 특별한 언급 없는 경우 기본 설정 유지:
  - 온도 조정 없음(t=1)
  - 리랭킹 샘플 크기 N=512

## 결론

이 논문에서는 autoregressive transformer 을 기반으로 한 Text-Image 생성에 대해, 규모를 확장했을 때 어떤 결과가 나오는지 조사했다. 그 결과, 규모를 확장함으로써 **일반화 성능**이 향상된다는 것을 발견하였다. 이 향상은 이전의 도메인 특화 방식들과 비교했을 때 **제로샷 성능**에서 나타나며, 또한 **하나의 generative model에서 나타나는 다양한 능력의 범위** 에서도 확인할 수 있다. 이러한 결과는 모델의 일반화 성능을 규모의 함수로 향상시키는 것이 이 text to image generation 작업에서 진보를 이끄는 유용한 동력이 될 수 있다는 것을 시사한다.
