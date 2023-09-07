<div align=center>
  <h1>
  <h1>DACON 2023 전력사용량 예측 AI 경진대회
</div>


## 설명
건물 정보와 시공간 정보를 활용하여 특정 시점의 전력 사용량을 예측하는 모델 개발

## 데이터
1. train.csv

train 데이터 : 100개 건물들의 2022년 06월 01일부터 2022년 08월 24일까지의 데이터
일시별 기온, 강수량, 풍속, 습도, 일조, 일사 정보 포함
전력사용량(kWh) 포함

2. building_info.csv

100개 건물 정보
건물 번호, 건물 유형, 연면적, 냉방 면적, 태양광 용량, ESS 저장 용량, PCS 용량

3. test.csv

test 데이터 : 100개 건물들의 2022년 08월 25일부터 2022년 08월 31일까지의 데이터
일시별 기온, 강수량, 풍속, 습도의 예보 정보

4. sample_submission.csv

제출을 위한 양식
100개 건물들의 2022년 08월 25일부터 2022년 08월 31일까지의 전력사용량(kWh)을 예측
num_date_time은 건물번호와 시간으로 구성된 ID
해당 ID에 맞춰 전력사용량 예측값을 answer 컬럼에 기입해야 함

## 평가방식
1. 심사 기준: SMAPE(Symmetric Mean Absolute Percentage Error)
2. 대회 제공 데이터 이외의 외부 데이터 사용 금지
3. 2022.08.25 00:00:00부터 알 수 있는 특성을 활용하는 경우 Data Leakage로 실격 (리더보드 기록 삭제)
4. 2022.08.25 00:00:00부터 발생한 모든 사건은 전혀 알 수 없다고 가정하여 진행해야함
5. 따라서 2022.08.25 00:00:00 이전에 제작된 예보 데이터인 test.csv는 활용 가능
6. Pseudo Labeling은 사용 불가

## 접근법
<h3> 1. 전체 데이터 XGB 모델링</h3>
<h3> 2. 빌딩 번호별 XGB 모델링</h3>
<h3> 3. 빌딩 번호, 시간, 요일에 따른 K-means clustering 후 군집별 pycaret 1등 모델 모델링</h3>
<h3> 4. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 1등 모델 모델링</h3>
<h3> 5. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 상위 모델 스태킹</h3>
<h3> 6. 빌딩 타입, 시간, 요일에 따른 K-menans clustering 후 군집별 pycaret 상위 모델 앙상블</h3>
