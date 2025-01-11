# 2023년 숭실대학교 캡스톤 디자인 프로젝트
## 팀 프로젝트
### 주제 선정 이유
(1) 탄소중립과 온실가스 감축
- 서울시는 2026년까지 온실가스를 30% 감축할 계획이며, 온실가스 배출의 88%가 교통 및 건물 부문에서 발생.
- 주유소 감소: 전국 및 서울시 주유소 수가 지속적으로 감소(2013년 대비 2022년 서울시 주유소 약 25% 감소).
- 친환경 차량 증가: 전기차, 수소차 등 친환경 차량의 신차 비중이 2025년 50.6%, 2030년 83.3%에 이를 것으로 전망.
- 수익성 감소: 기름값 상승과 출혈 경쟁, 폐업 부담 증가로 주유소 생존 전략 필요.
  
(2) 감소 원인
- 친환경 차량 확대: 누적 판매량이 2013년 244,158대에서 2022년 1,477,756대로 급증.
- 수익성 감소: 경쟁 심화와 고비용으로 인한 폐업 사례 증가.
- 지역적 특성: 수도권에서 친환경 차량이 높은 비율을 차지, 서울은 친환경 차량 인프라의 중심지.
  
(3) 주유소 현황
- 물류 및 유통 허브화: 주유소를 택배 집하장이나 지역 물류 거점으로 활용.
- 첨단 물류 복합 주유소 필요성: 친환경 차량 충전소, 드론, 로봇 물류 기술 등 미래 물류 거점으로의 전환 필요.
  
(4) 첨단 물류 복합 주유소의 필요성
- 탄소중립 정책과 친환경 차량 증가에 따른 수익성 감소를 해결하기 위해, 물류와 친환경 모빌리티의 복합적인 공간으로 전환이 요구됨.

## Stack
```
R
Python
```

# 코드 실행
## 1. Clustering
아래 Rmd 파일에 기술되어 있음
https://github.com/HyunJae0/capstone-project/blob/main/%EC%A0%84%EC%A2%85%EC%84%A41.Rmd
### 1.1 클러스터링 변수 선정
클러스터링을 진행할 데이터 셋은 다음과 같습니다.
![image](https://github.com/user-attachments/assets/a2fad287-9ceb-4572-bd57-6ec333367f26)
이때, 물류창고의 수는 대부분 0에 치우쳐져 있어 클러스터링 변수로 사용하기엔 적합하지 않다고 판단하였습니다.
```
df_cor <- df6%>%
  select(-법정동)
corr_df <- cor(df_cor)
corrplot(corr_df, method = 'number', order = 'hclust', type = 'lower', diag = FALSE)
```
인구수와 시설 수 간의 양의 상관관계를 확인하고, 인구수 대비 시설이 부족한 지역을 ‘우선 설치 후보군’으로 선정하였습니다. 인구수가 택배함, 공공자전거 거차대, 친환경차 충전소와 양의 상관관계를 보이는 것을 확인할 수 있습니다.
![image](https://github.com/user-attachments/assets/ff20044b-e230-44ea-b7ea-7372806a1a8b)
일반적으로 인구가 많을수록 생활 편의/교통 관련 시설이 더 많이 필요하고 실제로도 많이 들어서는 경향이 있기 때문에, 인구가 많은 지역에 해당 시설들이 더 많이 필요한 것으로 해석했습니다.

클러스터링 변수를 '인구수', '택배함 개수', '공공자전거 거치대수', '충전소 개수(전기, 수소)'로 선정했습니다.
![image](https://github.com/user-attachments/assets/310701df-de05-4cda-8034-8776202a5e07)

### 1.2 스케일링
min-max 스케일링을 적용해 모든 특성의 값이 [0, 1] 구간에 위치하도록 하였습니다.
```
min_max_scaling <- function(x) {
  (x - min(x)) / (max(x) - min(x))
}

# 데이터프레임의 모든 열에 Min-Max 스케일링 적용
scaled_final <- as.data.frame(lapply(df5, min_max_scaling))
row.names(scaled_final) <- row.names(df5)
str(scaled_final)
```
![image](https://github.com/user-attachments/assets/19f5116b-d1b9-4888-9cd4-f48c96bb11cd)
클러스터링에서 '인구수'처럼 스케일(값의 크기)이 큰 변수가 있으면, 두 변수 간 전체 거리 계산을 왜곡시킬 수 있기 때문에 스케일 조정이 필수입니다.
'인구수'처럼 한 변수가 월등히 큰 수치를 갖고 있으면, 거리 기반 클러스터링에서 그 변수 하나가 전체 거리를 대부분 지배하게 됩니다. 결과적으로 클러스터링 결과가 '인구수'기준으로만 군집이 나뉘어버리고 '시설 수'같은 다른 변수들은 제대로 반영되지 못하는 문제가 발생합니다.
### 1.3 클러스터링 평가

## 2. 회귀분석 모델링

## 3. PCA를 이용한 평가지표 설정

## 4. 최종 선택

# 결과
![image](https://github.com/user-attachments/assets/02a5beb5-99ac-4a84-8be4-66df44f432c1)
