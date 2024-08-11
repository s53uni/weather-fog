<a name="top"></a>
  
# SMOTE 기법을 활용한 XGBoost 안개 예측 모델
- 구분: 팀 프로젝트
- 기간: 2024년 5월 21일 ~ 2024년 6월 28일

<details>
  <summary>Table of Contents</summary>
  
  1. [프로젝트 개요](#프로젝트-개요)
  2. [데이터 정의](#데이터-정의)
  3. [프로젝트 과정](#프로젝트-과정)
      + [데이터 전처리](#데이터-전처리)
      + [파생 변수 생성](#파생-변수-생성)
      + [오버 샘플링](#오버-샘플링)
      + [모델링 및 결과](#모델링-및-결과)
  5. [결론](#결론)

</details>
<br>

## 프로젝트 개요
안개는 시야를 방해하여 교통사고와 항공·해양 운송의 어려움을 초래하고, 농작물의 광합성을 저해하여 경제적 손실을 유발한다. 
따라서 안개 발생을 정확히 예측하는 것이 중요하며, 이를 위해 내륙 및 해안 지역에서 수집된 기상 관측 데이터를 활용한 예측 모델을 개발하고자 한다. 
본 연구는 각 지역의 시정 구간을 정확히 예측하여 교통안전 확보와 농업 피해 최소화를 목표로 한다.

<br>

## 데이터 정의
기상청 빅데이터 콘테스트의 과제 2에서 제공된 안개 데이터(fog_train, fog_test)를 활용하였다.<br> 
훈련 데이터는 I년부터 K년까지의 기간을, 검증 데이터는 L년의 기간을 포함한다.

<p align="right"><a href="#top">⬆️TOP</a></p>

## 프로젝트 과정
### 데이터 전처리
1. 년도(year) 변수는 I부터 L까지의 문자형 데이터를 1부터 4까지의 숫자로 변환하여 숫자형 변수로 재정의함
2. 지점 번호(std_id) 변수는 AA부터 EC까지의 지점을 포함하며, 각 지점 번호의 첫 글자만 추출하여 원-핫 인코딩 진행
3. 독립변수에 존재하는 결측값 -99.9는 NaN 값으로 변환한 후, 선형 보간법을 적용하였음
4. 종속변수인 시정 구간(class)에 존재하는 결측값 –99는 임의로 대체하면 모델의 예측력이 저하될 수 있으므로 결측값을 삭제함

<br>

### 파생 변수 생성
#### 기상 파생 변수
1. 풍향의 사인 및 코사인 값
- 풍향을 단순히 각도로 표현할 경우 주기성을 제대로 반영하지 못하므로, 사인 및 코사인 값으로 변환하여 사용함
- 특정 풍향은 바다나 강에서 육지로 습한 공기를 이동시킬 수 있으며, 이를 통해 안개 발생을 효과적으로 예측할 수 있음

2. 풍속의 제곱
- 일정 수준 이상의 풍속은 혼합 작용을 통해 안개를 흩뜨리거나 형성을 방해할 수 있음
- 풍속과 안개 발생 간의 비선형적인 관계를 반영하기 위해, 풍속의 제곱 값을 사용하여 풍속이 클 때의 영향을 모델에 반영함

3. 이슬점 온도
- 이슬점 온도는 기온과 상대습도를 바탕으로 계산되며, 공기가 포화 상태에 도달하여 수증기가 응결하는 온도를 나타냄
- 높은 이슬점은 대기가 더 많은 수증기를 포함하는 것으로, 이슬점이 기온에 가까울수록 안개 발생 가능성이 높아짐

4. 절대 습도
- 절대 습도는 기온과 상대습도를 이용해 계산되며, 특정 온도에서 공기 중 포함된 수증기의 양을 나타냄
- 높은 절대 습도는 안개 발생 가능성을 높이며, 이를 예측 모델에 반영하여 안개 발생을 더 정확히 예측할 수 있음

#### 시간 파생 변수
1. 야간 여부
- 안개는 주로 야간에 기온이 낮아지면서 발생하는 경우가 많아 이를 고려하여 야간 여부를 파생 변수로 추가함
- 오후 6시부터 다음 날 오전 6시까지를 야간(1)으로, 그 외의 시간대는 주간(0)으로 정의함

2. 계절 변수
- 계절에 따라 기상 조건이 변화하며, 이는 안개 발생 빈도와 강도에 영향을 미칠 수 있으므로 계절 변수를 추가함
- 봄(3, 4, 5월), 여름(6, 7, 8월), 가을(9, 10, 11월), 겨울(12, 1, 2월)을 각각 1, 2, 3, 4로 정의함

<p align="right"><a href="#top">⬆️TOP</a></p>

### 오버 샘플링

| 시정 구간 | SMOTE 적용 전 개수 | SMOTE 적용 후 개수 |
| ------ | ------ | ------ |
| 1 | 7,866 | 3,101,809 |
| 2 | 12,088 | 3,101,809 |
| 3 | 12,180 | 3,101,809 |
| 4 | 3,101,809 | 3,101,809 |

- 데이터의 불균형을 해결하기 위해 SMOTE 기법을 적용하여 각 구간의 데이터 개수를 각각 3,101,809개로 증가시킴
- 새로 생성된 데이터는 기존 데이터와 유사하면서도 약간의 변형을 가지고 있어 소수 클래스에 대해 더 잘 일반화할 수 있음

<br>

### 모델링 및 결과
- 사용한 모델: RandomForest, XGBoost, LightGBM, AdaBoost
- 데이터 분할: 데이터를 8:2 비율로 학습용과 검증용으로 분할함

<br>

#### 앙상블 모델 예측 결과

| model | train_accuracy | test_accuracy | f1-score | precision | recall | auc | csi |
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| RF | 1.000000 | 0.998567 | 0.998567 | 0.998567 | 0.998567 | 0.999988 | 0.998092 |
| XGBoost | 0.782575 | 0.781619 | 0.781357 | 0.781987 | 0.781619 | 0.941890 | 0.711473 |
| LightGBM | 0.733364 | 0.732875 | 0.732350 | 0.732764 | 0.732875 | 0.918170 | 0.648459 |
| AdaBoost | 0.571195 | 0.570512 | 0.568084 | 0.567130 | 0.570512 | 0.779857 | 0.453610 |

- RandomForest 모델의 훈련 정확도가 1.0으로 나타나, 훈련 데이터에 과적합된 것으로 의심됨
- XGBoost 모델은 f1-score, precision, recall 모든 평가 지표에서 0.78을 기록하며 우수한 성능을 보임
- 이는 XGBoost가 새로운 데이터에서도 높은 예측 성능을 보일 가능성이 크다는 것을 의미하며, 최종 모델로 XGBoost를 선정함

<br>

#### 앙상블 모델 혼동 행렬 결과

<table>
    <tr align="center">
        <td><b>RF</b></td>
        <td><b>XGBoost</b></td>
        <td><b>LightGBM</b></td>
        <td><b>AdaBoost</b></td>
    </tr>
    <tr align="center">
        <td><img src="https://github.com/user-attachments/assets/3f1938b6-3246-49fb-93e6-32462f300857"></td>
        <td><img src="https://github.com/user-attachments/assets/c64e2e3b-4278-4d6e-82a3-bcd266374cd1"></td>
        <td><img src="https://github.com/user-attachments/assets/3b0b7305-c7b9-4a7b-a936-2a702180abc9"></td>
        <td><img src="https://github.com/user-attachments/assets/a05ffa56-c030-4128-8eaa-f2eb81ce42b3"></td>
    </tr>
</table>

- RandomForest 모델은 훈련 데이터에 과적합되어 대각선이 매우 진하게 나타남
- AdaBoost 모델은 정확도가 비교적 낮아 혼동행렬에 더 많은 분산이 있는 것으로 나타남
- 이러한 혼동행렬 분석 결과, 마찬가지로 XGBoost 모델이 과적합 없이 가장 적합한 모델로 선정됨

<br>

#### 최종 검증 CSI
최종 XGBoost 모델로 검증 데이터를 예측하고 계산한 CSI는 0.122이다.

<p align="right"><a href="#top">⬆️TOP</a></p>

## 결론
본 연구에서는 결측된 기상 데이터를 선형 보간하고, 안개 발생 예측에 영향을 주는 파생 변수를 추가하여 효과적인 예측 모델을 개발하고자 했다. 
안개 발생이 적은 기상 데이터의 불균형 문제는 SMOTE 오버 샘플링 기법을 사용하여 해결하고, 이를 통해 모델의 성능을 향상시켰다. 
4개의 앙상블 모델을 학습하고 비교 분석한 결과, 가장 일반화된 모델인 XGBoost 모델을 최종 모델로 선정했다. 선정된 XGBoost 모델의 최종 검증 CSI 값은 0.122이다.
<br><br>
이 모델은 운전자에게 사고 가능성을 줄이기 위한 경고를 제공할 수 있으며, 안개 예측을 통해 농업 생산성을 유지하고 향상시키는 데 기여할 수 있다. 
또한, 안개 예측 모델은 대기 중 미세먼지를 제거하는 공기 품질 개선과 환경 보호에도 도움을 줄 수 있다. 
따라서 이 모델을 통해 환경의 지속 가능성을 높이는 데 중요한 역할을 할 것으로 기대된다.

<p align="right"><a href="#top">⬆️TOP</a></p>
