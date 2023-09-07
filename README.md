<div align=center>
  <h1>
  <h1>DACON 2023 전력사용량 예측 AI 경진대회
    <h3>건물 정보와 시공간 정보를 활용하여 특정 시점의 전력 사용량을 예측하는 모델 개발</h3>
</div>

    
<div align=right>
  <h5>EDA 및 코드는 각 팀원의 개별 파일에 포함되어 있음</h5>
</div>

---


<br>


## 데이터
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


---


## Data Preprocessing
- 풍속, 습도 결측치 행 ffill을 사용하여 처리
- 강수량, 일사, 일조 feature 삭제
- 태양광 용량, ESS 저장 용량, PCS 용량의 '-'를 0으로 대체 및 float 변환
- 왜도, 첨도가 높은 feature인 연면적, 냉방 면적, 태양광 용량, ESS 저장 용량, PCS 용량을 log 변환
- 시간, 일, 요일 feature sin, cos 변환
- 빌딩 타입 one-hot encoding
- 전력사용량 log 변환
- 연속형 변수 StandardScaling


  <hr>

  
<h5>파생변수 생성</h5>

- 불쾌지수 (기온, 습도 활용)
- 체감온도 (기온, 풍속 활용)
- 기온, 풍속, 습도, 불쾌지수의 1,2,3 시간 변화
- 주말 및 공휴일
- 시간, 일, 요일, 달 (빌딩번호/일자정보(num_date_time) 데이터 활용)
- 태양광, ESS 설치 여부 (태양광 용량, ESS 용량 feature 여부)
- 불쾌지수, 기온의 3시간, 5시간 이동평균
- 냉방도일(CDH)
- 빌딩별 일평균 기온/불쾌지수/냉방도일
- 빌딩별 시간, 요일별 전력사용량 평균
- 빌딩별 시간, 요일에 따라 평균 전력사용량이 높은 특정 시간대를 work_time, 낮은 특정 요일을 low_day, 특이한 경우의 particular로 feature 생성


---

## Feature selection 방식 (모델링 방식에 따라 다르지만 사용한 방식들을 나열함)
- XGB feature importance
- Shap value with XGB
- RFECV with XGB
- 상관관계 높은 변수는 다중공선성의 문제를 야기하며 오히려 성능 저하를 일으켜 삭제

---


## 사용한 모델링 방식
<div align=center>
  <h5> 1. 전체 데이터 XGB</h5>
<h5> 2. 빌딩 번호별 XGB</h5>
<h5> 3. 빌딩 번호, 시간, 요일에 따른 K-means clustering 후 군집별 pycaret 1등 모델</h5>
<h5> 4. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 기준 SMAPE 1등 모델</h5>
<h5> 5. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 기준 SMAPE 상위 모델 스태킹</h5>
<h5> 6. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 기준 SMAPE 상위 모델 앙상블</h5>
</div>


---


## 후처리
해당 대회의 평가 지표인 SMAPE 특성 상 과소추정의 경우 과대추정에 비해 현저히 낮은 퍼포먼스를 보이기에
예측값이 빌딩별 시간, 요일별 전력사용량의 평균보다 낮은 경우 평균으로 대체


---


## 최종 결과
<div align=center>
<h3> 3., 5., 6. 의 모델링 방식이 우수한 성능을 보임</h3>
  <hr>
<h2> 총 참가자 1880명 중 Private 7.80492 146등, Public 6.13548 157등 으로 마감</h2>
</div>
