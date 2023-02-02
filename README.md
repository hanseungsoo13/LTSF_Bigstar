# LTSF_Bigstar

## Introduction

### 과제이해

천연가스란?

LNG, CNG, PNG 등으로 분류되며 도시가스, 난방, 천연가스 버스 등에 이용되는 자원

- 우리나라의 천연가스 수입량은 세계 3위에 해당
- 미래의 천연가스 수요를 미리 예측하여 적절한 수급량을 미리 확보하는 것이 중요
- 과제 소개
    - 과거로부터 기록된 민수용/산업용 천연가스 소비량을 참고하여 다양한 입력데이터를 활용해 미래의 천연가스 소비량을 예측하는 것
    - 타겟 데이터는 시계열 데이터 이고, 도메인 조사 결과, 보다 유의미한 입력데이터를 활용하는것이 효과적일 것이라 예측
    - 다양한 외부지표를 이용하여 타겟 시계열 데이터를 예측하는 것이 중요

## 데이터이해

###Data Structure

![화면 캡처 2023-01-18 094411.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/97888f07-b940-4b96-b3cf-ebf56975d5af/%ED%99%94%EB%A9%B4_%EC%BA%A1%EC%B2%98_2023-01-18_094411.png)

위와 같이 제공된 X와 Y를 효과적으로 활용하기 위해 다음 3가지를 강조한다.

1. 외부데이터 수집
2. 외부데이터의 시계열 변환
3. 외부데이터와 타겟의 관계를 분석

본 데이터들을 분석에 활용한 중점은 외부데이의 미래값을 예측한 후 회귀 모형을 이용하여 타겟을 예측하는 것이다.

### EDA, 데이터 선별

타겟값이 시계열 데이터인 만큼 Trend, Seasonal의 대표성을 가진 데이터들을 위주로 수집

# Modeling

### 방법론

![화면 캡처 2023-01-18 095503.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4b8d0288-1a0e-4dbe-a365-971a1f3174ea/%ED%99%94%EB%A9%B4_%EC%BA%A1%EC%B2%98_2023-01-18_095503.png)

- 개략적인 방법론은 위의 그림을 통해 설명 가능하다.
1. 외부데이터에 D-Linear모델을 적용하여 미래의 외부데이터 X값을 생성한다.
2. 기존의 데이터와 생성한 미래 외부데이터 X를 이용하여 회귀모델을 생성한다.
3. Time Series모델과 회귀모델의 예측 결과를 앙상블하여 최종 결과를 생성한다.

### Feature Engineering

Target의 효과적인 예측을 위해 설명변수들에 Feature Engineering을 적용해주었다.

1. PCA
2. Polynomial 피쳐 추가

### Feature Selection

회귀모형의 적합성을 이용하여 유의미한 Feature들만을 선별

- P-value가 0.3이상인 Feature들을 선정
- 다중공선성을 고려해 대표성이 있는 Feature들을 선정
- 도메인 지식에 따라 유의미한 Feature들을 선정

### D-Linear

- 시계열 분해를 이용하여 LTSF(Long Time Series Forecasting)과제에 SOTA를 달성한 모델
    
    ![그림3.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d1bf40af-d744-4e63-ba33-0f89217c119f/%EA%B7%B8%EB%A6%BC3.png)
    
    - 시계열 데이터를 Seasonal data와 Trend data로 변환하여 각각을 예측한 후 최종 결과를 예측하는 모델
    
- 이 모델을 본 데이터에 적용하기 위해 수정한 부분은 다음과 같다.
    - 시계열의 단위를 hour에서 month로 바꿈
    - History timesteps와 Future timestep을 지속적으로 바꿔가며 각 모델마다 최적의 t를 찾아냄
    - Future timestep보다 더 긴 time step을 예측해야하는 과제이기에 예측값을 다시 input으로 활용하는 순환적 구조로 활용

### Ensemble

앙상블 이전, Target값을 예측하기 위해 활용한 모델들은 다음과 같다.

1. 단순 회귀 모형
    - Sample의 수는 적고 Column의 수가 더 많아 단순한 회귀식이 더 효과적이었음
2. Extra Tree 회귀 모델
    - 많은 Tree들을 이용하여 예측하기에 일반화의 성능에 향상을 보임
3. D-Linear
    - 외부데이터를 활용하지 않고 단순 Time Series 데이터만을 이용하여 예측하여 Target값 고유의 Season과 Trend를 반영하고자 하였음

## Conclusion

![그림5.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e6ea1a81-9f92-4278-8d65-8e057b2fe8d4/%EA%B7%B8%EB%A6%BC5.png)

위의 그림이 최종적으로 예측한 예측값

- 파란색: 민수용 천연가스 수요량, 주황색: 산업용 천연가스 수요량

본 연구를 통해 얻은 결론 및 제언은 아래와 같습니다.

- 기존의 단순히 target만을 이용한 시계열 모델에서 벗어나 유의미한 외부변수를 활용하면 더욱 효과적인 예측이 가능함
- 필요한 외부변수를 선정할 때 다항회귀를 통해 통계적으로 유의미한 변수들만을 선택함으로써 효율성을 극대화함
- 변수의 미래를 예측하는 시계열 모델에서 변수별로 미세조정을 해준 것이 효과적이었음
- 단 하나의 방법론으로 예측하기 보다 여러가지 모델들을 동시에 활용하고 앙상블 하여 과적합을 방지할 수 있었음
