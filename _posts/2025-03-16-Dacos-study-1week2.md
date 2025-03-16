---
layout: post
title: "핸즈온 머신러닝 2.1~2.4"
date: 2025-03-16
categories: [research, dacos-study] # 또는 [research, nlp] 또는 [research, computer-vision]
use_math: true
---

## 2장- 머신러닝 프로젝트 처음부터 끝까지

- [2장- 머신러닝 프로젝트 처음부터 끝까지](#table-of-contents)
- [2.1 실제 데이터로 작업하기](#21-실제-데이터로-작업하기)
- [2.2 큰 그림 보기](#22-큰-그림-보기)
- [The end](#the-end)

## [2.1 실제 데이터로 작업하기](#21-실제-데이터로-작업하기)

• 유명한 공개 데이터 저장소

- OpenML(https://openml.org)
- 캐글(https://kaggle.com/datasets)
- PapersWithCode(https://paperswithcode.com/datasets)
- UC 어바인 머신러닝 저장소 (https://archive.ics.uci.edu/ml)
- 아마존 AWS 데이터셋(https://registry.opendata.aws)
- 텐서플로 데이터셋(https://tensorflow.org/datasets)
  메타 포털(공개 데이터 저장소가 나열되어 있는 페이지)
  데이터 포털(https://dataportals.org)
- 오픈 데이터 모니터(https://opendatamonitor.eu)
  인기 있는 공개 데이터 저장소가 나열되어 있는 다른 페이지
  위키백과 머신러닝 데이터셋 목록 (https://homl.info/9)
  Quora(https://homl.info/10)
  데이터셋 서브레딧 subreddit (https://www.reddit.com/r/datasets)

## [2.2 큰 그림 보기](#22-큰-그림-보기)

- 활용 데이터: 캘리포니아의 블록 그룹마다 인구, 중간 소득, 중간 주택 가격 등을 담고 있다.
- 이 데이터로 모델을 학습시켜서 다른 측정 데이터가 주어졌을 때 구역의 중간 주택 가격을 예측.

### 2.2.1 문제 정의

#### 머신러닝 프로젝트 접근 가이드

1.  비즈니스 목적 파악

    비즈니스의 목적이 정확히 무엇인지 이해하는 것이 첫 번째 단계이다.모델 자체가 아닌 모델을 통해 얻고자 하는 비즈니스 가치를 파악해야 한다. 목적 이해는 문제 구성, 알고리즘 선택, 성능 지표 선정, 모델 튜닝 노력 결정에 영향을 미친다.

    - 예시: 구역 중간 주택 가격 예측 모델은 투자 결정을 위한 입력 신호로 사용된다.

2.  데이터 파이프라인 이해

    데이터 파이프라인은 연속된 데이터 처리 컴포넌트들의 집합이다.컴포넌트들은 비동기적으로 작동하며 독립적이다.장점:시스템 이해도 향상,팀별 집중 가능,시스템 견고성 증가
    단점:모니터링 부재 시 고장 인지 지연,오래된 데이터 사용 시 전체 성능 저하

3.  현재 솔루션 분석

    기존 솔루션은 참고 성능 지표와 문제 해결 방향에 대한 통찰을 제공한다.

    - 예시: 현재는 전문가가 복잡한 규칙으로 수동 추정하여 비용과 시간이 많으며 정확도가 낮다(30% 이상 오차). 인구 조사 데이터셋은 중간 주택 가격 예측에 적합한 데이터로 판단된다.

4.  학습 방식 결정

    지도 학습: 레이블된 훈련 샘플(구역 데이터와 중간 주택 가격)이 있어 적합하다.

    회귀 문제: 값(중간 주택 가격)을 예측하는 회귀 문제이다.

    다중 회귀: 여러 특성(인구, 중간 소득 등)을 사용한다.

    단변량 회귀: 각 구역당 하나의 값만 예측한다.
    배치 학습: 데이터 크기가 적당하고 연속적 데이터 흐름이 없어 적합하다.

5.  **대규모 데이터 처리[^1]**

### 2.2.2 성능 지표 선택

머신러닝 모델의 성능을 평가하기 위해 **적절한 측정 지표(metric)** 를 선택하는 것이 중요하다. 특히 **회귀 문제(regression)** 에서는 **평균 제곱근 오차(RMSE, Root Mean Square Error)** 가 일반적으로 사용된다.
RMSE는 **오차가 커질수록 값이 더 커지므로**, 예측 성능이 얼마나 정확한지 직관적으로 이해 가능

RMSE는 다음과 같이 계산된다.

$
RMSE(X, h) = \sqrt{\frac{1}{m} \sum_{i=1}^{m} (h(x^{(i)}) - y^{(i)})^2}
$

- $m$: 데이터셋의 샘플 수
- $x^{(i)}$: $i$번째 샘플의 특성 벡터
- $y^{(i)}$: $i$번째 샘플의 실제 값 (타겟 값)
- $h(x^{(i)})$: 모델이 예측한 값 (가설 함수)

**RMSE는 오차의 제곱을 평균내고 제곱근을 취하므로, 오차의 크기가 클수록 더 큰 영향을 미친다.** 따라서 **큰 오차를 더 중요하게 다루고 싶을 때 RMSE를 사용하면 유용**하다.

#### 평균 절대 오차 (MAE)

이상치(outlier)가 많을 경우 **평균 절대 오차(MAE, Mean Absolute Error)** 를 사용하는 것이 더 적절할 수 있다.<br>
MAE는 다음과 같이 계산된다.

$
\[
MAE(X, h) = \frac{1}{m} \sum\_{i=1}^{m} | h(x^{(i)}) - y^{(i)} |
\]
$
<br>

**차이점: RMSE vs MAE**
RMSE는 이상치가 있는 경우 영향을 크게 받고, MAE는 이상치 영향을 상대적으로 덜 받는다.이러한 특성 때문에 RMSE는 종 모양 분포의 양 끝단처럼 이상치가 매우 드문 **일반적인 데이터 분포에서 자주 사용**되지만, 이상치가 많다면 MAE가 더 적절할 수 있다.

### 2.2.3 가정 검사

- 팀원들과 지금까지 만든 가정을 나열하고 검사해보기. 심각한 문제를 일찍 발견할 수 있기에 꼭 해보자.문제를 회귀or 분류 등 어떤 solution 으로 해결을 할지 확인을 꼭 해야 한다.

## [2.3 데이터 가져오기](#23-데이터-가져오기)

[02_end_to_end_machine_learning_project.ipynb](https://colab.research.google.com/github/rickiepark/handson-ml3/blob/main/02_end_to_end_machine_learning_project.ipynb)

[^2]

```python
# 데이터를 추출하고 로드하는 함수
from pathlib import Path
import pandas as pd
import tarfile
import urllib.request
def load_housing_data():
    tarball_path = Path("datasets/housing.tgz")
    if not tarball_path.is_file():
        Path("datasets").mkdir(parents=True, exist_ok=True)

        url = "https://github.com/ageron/data/raw/main/housing.tgz"
        urllib.request.urlretrieve(url, tarball_path)
        with tarfile.open(tarball_path) as housing_tarball:
            housing_tarball.extractall(path="datasets")
    return pd.read_csv(Path("datasets/housing/housing.csv"))
housing = load_housing_data()
```

load_housing_data() 함수를 호출하면 datasets/housing.tgz 파일을 찾는다.

### 2.3.6 데이터 구조 훑어보기

_head()_ --> 각 행은 하나의 구역을 나타냅니다. 특성은 longitude, latitude, housing\*median_age.
total_rooms. total_bedrooms, population. households, median_income. median_house_value, ocean_proximity 등 10개

_info()_ -> 전체 행 수, 각 특성의 데이터 타입과 널이 아닌 값의 개수 확인 . 20,640개의 샘플,ocean_proximity 필드만 빼고 모든 특성이 숫자형.

_describe()_ -> 숫자형 특성의 요약 정보, std 행은 값이 퍼져있는 정도를 측정하는 표준 편차를 나타낸다.

<img src="images/handsonmc/i3.png" width="500" height="300">

모든 숫자형 특성에 대한 히스토그램.중간 소득은 달러 단위가 아니라 숫자로 변환되어 있음을 확인 가능하다.또한 중간 주택 연도와 중간 주택 가격의 최댓값과 최솟값이 제한되어있다. 이를 해결하기 위해 한계값 데이터의 정확한 레이블을 확보하고, 훈련 세트에서 이런 구역을 제거해야 한다. 마지막으로 일부 특성 값이 다른 특성보다 훨씬 크거나 작은데, ml 에서는 스케일이 다르면 학습이 어려울 수 있다. 따라서 특성 스케일링을 통해 표준화 or 정규화를 사용해야 한다.

- _대부분의 histogram 에서 오른쪽 꼬리가 더 긴데, 일부 ml 알고리즘은 정규분포가 아닌 경우 패턴을 찾기 어렵다. 따라서 데이터 변환을 통해 더 정규분포에 가깝도록 조정해야 한다._ [^3]

### 2.3.7 테스트 세트 만들기

---

{: data-content="footnotes"}

[^1]: 데이터가 매우 클 경우 MapReduce로 배치 학습을 분산하거나 온라인 학습 기법을 고려할 수 있다.
[^2]: 데이터를 수동으로 내려받아 압축을 푸는 대신 이를 위한 함수를 작성하는 것이 일반적으로 낫다. 데이터를 내려받는 일을 자동화하면 여러 기기에 데이터셋을 설치해야 할 때도 편리하다.
[^3]: 데이터를 더 깊게 들여다보기 전에 테스트 세트를 따로 떼어놓아야 합니다. 그리고 테스트세트를 절대 들여다보면 안 됩니다.
[^4]:
