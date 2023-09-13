![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/308068f9-4d1d-43fb-8f77-67a4a04b1023)

<div align=center>
  <h1>
  <h1>DACON 2023 전력사용량 예측 AI 경진대회
    <h3>건물 정보와 시공간 정보를 활용하여 특정 시점의 전력 사용량을 예측하는 모델 개발
</div>


---


<h2>목차</h2>
[1. 배경](#배경)
[2. 주최 / 주관](#주최-/-주관)
[3. 참가 대상](#참가-대상)
[4. 데이터 소개]
[5. 평가방식]
[EDA (Exploratory Data Analysis)]
[Data Preprocessing]
[파생변수 생성]
[Feature Selection]
[모델링]
[후처리]
[최종 결과]


<br>

## 배경
안정적이고 효율적인 에너지 공급을 위해서는 전력 사용량에 대한 정확한 예측이 필요합니다.

따라서 한국에너지공단에서는 전력 사용량 예측 시뮬레이션을 통한 효율적인 인공지능 알고리즘 발굴을 목표로 본 대회를 개최합니다.


---


## 주최 / 주관
주최 : 한국에너지공단
주관 : 데이콘


---


## 데이터 소개
1. train.csv

  100개 건물들의 2022년 06월 01일부터 2022년 08월 24일까지의 데이터
일시별 기온, 강수량, 풍속, 습도, 일조, 일사 정보 포함
전력사용량(kWh) 포함

2. building_info.csv

  100개 건물 정보
건물 번호, 건물 유형, 연면적, 냉방 면적, 태양광 용량, ESS 저장 용량, PCS 용량

3. test.csv

  100개 건물들의 2022년 08월 25일부터 2022년 08월 31일까지의 데이터
일시별 기온, 강수량, 풍속, 습도의 예보 정보

4. sample_submission.csv

  제출을 위한 양식
  100개 건물들의 2022년 08월 25일부터 2022년 08월 31일까지의 전력사용량(kWh)을 예측
num_date_time은 건물번호와 시간으로 구성된 ID
해당 ID에 맞춰 전력사용량 예측값을 answer 컬럼에 기입해야 함


---


## 평가방식
- 심사 기준: SMAPE(Symmetric Mean Absolute Percentage Error)
- 대회 제공 데이터 이외의 외부 데이터 사용 금지
- 2022.08.25 00:00:00부터 알 수 있는 특성을 활용하는 경우 Data Leakage로 실격 (리더보드 기록 삭제)
- 2022.08.25 00:00:00부터 발생한 모든 사건은 전혀 알 수 없다고 가정하여 진행해야함
- 따라서 2022.08.25 00:00:00 이전에 제작된 예보 데이터인 test.csv는 활용 가능
- Pseudo Labeling은 사용 불가


```
def smape(true, pred):
  v = 2*abs(pred-true)/(abs(pred)+abs(true))
  output=np.mean(v)*100
  return output
```


---


## EDA (Exploratory Data Analysis)

<h3>년 기준 일별 평균 전력소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/8a9445ea-8bce-4811-be60-71b8a2ca31c0)



<h3>년 기준 시간 평균 전력소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/5dc1a33d-e84d-4038-8443-cfa5e9a35618)


<h3>일 기준 시간 평균 전력소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/a535e9f6-1d4c-450a-ac3d-9e17becc38c1)


<h3>월 기준 일 평균 전력소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/fbe1124b-283b-41a1-82e4-a2093103131b)


<h3>년 기준 월 평균 전력소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/6db66951-80fc-4965-ae97-593289bea42b)


<h3>요일별 평균 전력소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/179f5cc9-86d6-41d2-ac3c-01a648a5a798)


<h3>빌딩 타입 별 평균 전력 소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/c5a1a8bc-fa17-401c-96f2-3881b288dd9d)


<h3>빌딩 타입 별 일 기준 시간 평균 전력 소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/d1615bf9-dfc2-4ede-b9b8-07f00da8d6ae)


<h3>빌딩 타입 별 요일 평균 전력 소비량</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/7b52b113-866a-4e07-87c6-38db2b65da13)


<h3>feature 간의 상관관계 시각화</h3>

_전력소비량 기준, 대각 행렬 기준 한 쪽만 나타나게 설정_

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/da0dd964-13e9-4167-ae8c-7e6a66392ef8)


---


## Data Preprocessing

_test data도 마찬가지로 처리_

<h3>1. 풍속, 습도 결측치 행 ffill을 사용하여 처리</h3> 

**풍속이 0인 경우 결측치가 아닌 기상청 관측 기준 '무풍' 일경우 0으로 기입**


*무풍 : 평균 풍속이 1kts 미만인 경우*

    train_df['windspeed'].fillna(method='ffill', inplace=True)
    train_df['humidity'].fillna(method='ffill', inplace=True)


<h3>2. 강수량, 일사, 일조 feature 삭제</h3>
- test.csv에 일사, 일조 정보 없으며 강수량의 경우 습도로 대체 되며 중요도가 높지 않아 삭제 처리

    train_df = train_df.drop(['rainfall','sunshine', 'solar_radiation'], axis=1)

  
<h3>3. 태양광 용량, ESS 저장 용량, PCS 용량의 '-'를 0으로 대체 및 float 변환</h3>

    train_df = train_df.replace('-', '0')
    train_df['solar_power_capacity'] = train_df['solar_power_capacity'].astype('float64')
    train_df['ess_capacity'] = train_df['ess_capacity'].astype('float64')
    train_df['pcs_capacity'] = train_df['pcs_capacity'].astype('float64')
    
    
<h3>4. 왜도, 첨도가 높은 feature인 연면적, 냉방 면적, 태양광 용량, ESS 저장 용량, PCS 용량을 log 변환</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/55d2378e-915a-4a55-b6ea-e5a127f76fe7)

_태양광 용량, ESS 저장 용량, PCS 용량의 왜도와 첨도가 매우 크기에 log 변환 수행_


<h3>5. 시간, 일, 요일 feature sin, cos 변환</h3>

**sin cos 함수를 이용한 시간의 연속적 표현 (cyclical time encoding)을 위함**

    train['hour_sin'] = np.sin(2 * np.pi * (train['hour']+1)/24.0)
    train['hour_cos'] = np.cos(2 * np.pi * (train['hour']+1)/24.0)
    
    train_6 = train[train['month']==6]
    train_78 = train[train['month']!=6]
    train_6['day_sin'] = np.sin(2 * np.pi * train['day']/30)
    train_6['day_cos'] = np.cos(2 * np.pi * train['day']/30)
    
    train_78['day_sin'] = np.sin(2 * np.pi * train['day']/31)
    train_78['day_cos'] = np.cos(2 * np.pi * train['day']/31)
    train = pd.concat([train_6,train_78])
    
    train['weekday_sin'] = np.sin(2 * np.pi * (train['weekday']+1)/7)
    train['weekday_cos'] = np.cos(2 * np.pi * (train['weekday']+1)/7)


<h3>6. 빌딩 타입 one-hot encoding</h3>


<h3>7. 전력사용량 log 변환</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/9d3b081e-6a05-4870-8cd8-f1c0742f7f78)

**건물별 전력사용량의 왜도가 1.5 이상 또는 이하인 경우 RED color로 표시 한 결과 14개의 빌딩 발견**


_변환 후 4개로 줄어듦_

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/d638105f-592b-49d8-949e-40ab2e07052e)


**모델링 시에 target을 로그변환한 값으로 학습을 하고, 추론 시에는 모델 예측값에 exponential을 적용**


<h3>8. 연속형 변수 StandardScaling</h3>


---

  
## 파생변수 생성

- 불쾌지수 (기온, 습도 활용)
```
    train_df['discomfort'] = 0.81 * train_df['temperature'] + 0.01 * train_df['humidity'] * (0.99 * train_df['temperature'] - 14.3) + 46.3
```
  
- 체감온도 (기온, 풍속 활용)


- 기온, 풍속, 습도, 불쾌지수의 1,2,3 시간 변화
```
    train_df['temperature_1'] = train_df.groupby('building_number')['temperature'].shift()
    train_df['temperature_1'] = train_df[train_df['temperature_1'] != 0]['temperature'] - train_df[train_df['temperature_1'] != 0]['temperature_1']
    train_df['temperature_1'] = train_df['temperature_1'].fillna(0)
    train_df['temperature_2'] = train_df.groupby('building_number')['temperature'].shift(periods=2)
    train_df['temperature_2'] = train_df[train_df['temperature_2'] != 0]['temperature'] - train_df[train_df['temperature_2'] != 0]['temperature_2']
```


- 주말 및 공휴일


- 시간, 일, 요일, 달 (빌딩번호/일자정보(num_date_time) 데이터 활용)


- 태양광, ESS 설치 여부 (태양광 용량, ESS 용량 feature 여부)


- 불쾌지수, 기온의 3시간, 5시간 이동평균


- 냉방도일(CDH)

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/1e73ac6c-db0c-4c45-95c6-5705c52ca346)

  
- 빌딩별 일평균 기온/불쾌지수/냉방도일


- 빌딩별 시간, 요일별 전력사용량 평균


- 빌딩별 시간, 요일에 따라 평균 전력사용량이 높은 특정 시간대를 work_time, 낮은 특정 요일을 low_day, 특이한 경우의 particular로 feature 생성


_빌딩별 시간별 요일별 전력사용량 평균 시각화_

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/be28f7b0-ea41-4937-9472-b7916b3eafdc)


_군집별 빌딩별 시간별 전력샤용량 시각화_

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/09e958eb-d1bf-4e62-ab72-f6fc4c06ba20)


**두개의 시각화 자료를 통해 work_time, low_day, particular 생성**

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/d8ac3f61-ea08-455f-8eb0-96ae813735dc)


**특이한 이상치 데이터 삭제 및 특정 기간 이후 변동의 일정한 변화가 있는 빌딩의 기간 조정**

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/341286d2-cb21-47c3-afec-6b52e2ae77ff)


_etc.._

---


## Feature Selection (모델링 방식에 따라 다르지만 사용한 방식들을 나열함)
<h3>XGB feature importance</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/c0e242d9-e0e7-4bd8-bfa3-ba46cad7f367)

  
<h3>Shap value with XGB</h3>

![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/5362194f-0d57-4cdf-af24-f60f3dcbe95e)


<h3>RFECV with XGB</h3>

  ![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/af3e1f14-49bd-40a6-95c7-4471d8186957)


- 상관관계 높은 변수는 **다중공선성**의 문제를 야기하며 성능 저하를 일으키기 때문에 삭제


---


## 모델링
<div align=center>
  <h4> 1. 전체 데이터 XGB</h4>
  <hr>
<h4> 2. 빌딩 번호별 XGB</h4>
  <hr>
<h4> 3. 빌딩 번호, 시간, 요일에 따른 K-means clustering 후 군집별 pycaret 1등 모델</h4>
  <hr>
<h4> 4. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 기준 SMAPE 1등 모델</h4>
  <hr>
<h4> 5. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 기준 SMAPE 상위 모델 <b>스태킹</b></h4>
  <hr>
<h4> 6. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 기준 SMAPE 상위 모델 <b>앙상블</b></h4>
</div>


---


## 후처리
해당 대회의 평가 지표인 SMAPE 특성 상 과소추정의 경우 과대추정에 비해 현저히 낮은 퍼포먼스를 보이기에
예측값이 빌딩별 시간, 요일별 전력사용량의 최솟값보다 낮은 경우 최솟값으로 대체


---


## 최종 결과
<div align=center>
<h3> 4., 5., 6. 의 모델링 방식이 우수한 성능을 보임</h3>
  <hr>
<h2> 총 참가자 1880명 중 Private 7.80492 146등, Public 6.13548 157등 으로 마감</h2>
</div>


![image](https://github.com/2023dacon/Power-usage-Prediction/assets/90303745/5591a10a-4217-499b-b574-9475c8d47bd8)
