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
이 단계의 목표는 K-means, K-medoids, Hierarchical, Gaussian Mixture Clustering을 수행하여, 4개의 클러스터링 기법에서 두 번 이상 나온 지역을 선정하는 것입니다.

군집의 수를 정하기 위해 기본적으로 많이 사용하는 Elbow Method, Silhouette Method, Gap Statistic Method 사용했으며, 그럼에도 군집의 수를 결정하기 어려울 때 내부 평가 방법 중 하나인 Dunn Index를 확인하여 군집의 수를 결정하였습니다. 

예를 들어 K-medoids를 사용한 경우, 다음과 같이 Elbow 기법과 Silhouette 기법은 k = 3 또는 5를 생각할 수 있으나, Gap Statistic 기법의 결과는 Gap이 증가하기 직전인 k = 9가 최적으로 나왔습니다. 이런 경우는 군집의 수를 결정하기 어렵기 때문에 Dunn Index를 확인하였습니다.
```
#clValid
set.seed(123)
k_med_clvalid <- clValid(scaled_final,3:9, clMethods="pam",validation="internal", maxitems = nrow(scaled_final))
summary(k_med_clvalid)
```
다음 그림은 앞의 3가지 기법(Elbow Method, Silhouette Method, Gap Statistic Method)의 결과를 고려해 k = 3부터 9까지 Duun 값을 확인한 결과입니다.  
![image](https://github.com/user-attachments/assets/c66c1ee0-d896-4f16-b197-12371d6076ed)

클러스터 간의 분리도가 높고, 클러스터 내의 응집도가 높을수록 Dunn 지수는 더 큰 값을 가집니다. 그러므로 Dunn 값이 가장 높은 k = 5로 결정하였습니다.

그리고 k 값의 범위가 K-medoids와는 다르게 이산적으로 나오는 경우(예를 들어 k = 2 또는 k = 4), 다음과 같이 Between_SS / Total_SS 값을 비교했습니다.

예를 들어 Elbow Method, Silhouette Method, Gap Statistic Method를 사용했을 때, k-means의 군집 수는 다음과 같이 k = 2 또는 k = 4가 최적일 수 있습니다.
![image](https://github.com/user-attachments/assets/d8284e0a-7c70-428c-899a-aa734b9b56a9)

이런 경우에는 Between_SS / Total_SS 값을 비교합니다. 해당 값이 높을수록 잘 분류된 군집입니다. 클러스터 간의 거리를 나타내므로 값이 클수록 클러스터 간 구별이 잘 되는 것이기 때문입니다.

![image](https://github.com/user-attachments/assets/4792e5aa-b350-4c9f-ac10-4c54f1d430ea)

k = 4일 때, Between_SS / Total_SS 값이 높으므로 K-means의 k는 4를 설정합니다.

다음은 k = 4일 때, k-means 결과입니다.

![image](https://github.com/user-attachments/assets/07235cee-4229-4354-99c3-53b96b924656)

이렇게 군집을 얻었으면, 다음 단계는 프로파일링입니다. 
```
df_profile3 <- df6%>%
  select(인구수,택배함.개수,물류창고.개수,공공자전거.거치대수.LCD...QR.,친환경차.등록대수.전기.수소.,충전소.개수.전기.수소.,친환경차.한대당.충전소.개수)

# 프로파일링
mean_4<- df_profile3 %>%
  mutate(clst_k4 = km$cluster)%>%
  group_by(clst_k4)%>%
  summarise_all(mean)

mean_4$한명당.택배함 <- mean_4$택배함.개수/mean_4$인구수
mean_4$한명당.물류창고 <- mean_4$물류창고.개수/mean_4$인구수
mean_4$한명당.친환경충전소 <- mean_4$충전소.개수.전기.수소./mean_4$인구수
mean_4$한명당.자전거거치대 <- mean_4$공공자전거.거치대수.LCD...QR./mean_4$인구수

mean_4%>%
  select(clst_k4, 인구수, 한명당.택배함,한명당.물류창고,한명당.친환경충전소,한명당.자전거거치대,친환경차.한대당.충전소.개수)
```
![image](https://github.com/user-attachments/assets/b8ce011c-7947-4678-8e2a-93b0d659766a)

cluster 1이 인구수 대비 시설들(택배함, 충전소, 공공자전거 거치대)이 부족한 것을 확인할 수 있습니다. 그렇다면, cluster 1이 target cluster입니다.

그다음, cluster 1의 지역(법정동)을 저장합니다. 4개의 클러스터링 기법에서 두 번 이상 나온 지역을 선정하기 위해서입니다.
```
k_means_result <- k_means[km$cluster == 1, "법정동"]
```

위와 같은 방법으로 클러스터링을 진행하고, 결과를 프로파일링해서 target cluster를 찾습니다.

참고로 Hierarchical Clustering의 경우, hc_method를 다음과 같이 응집형 계수가 값이 1에 가까운 hc_method를 사용했습니다. 값이 1에 가까울수록 강력한 클러스터링 구조. 즉, 군집이 가장 안정적이기 때문입니다.
```
d=dist(scaled_final)
m <- c( "average", "single", "complete", "ward", "weighted","gaverage")
names(m) <- c( "average", "single", "complete", "ward","weighted","gaverage")
ac <- function(x) {
  agnes(scaled_final, method = x)$ac
}
choose_best_alg<- map_dbl(m, ac)
print(choose_best_alg)
```
![image](https://github.com/user-attachments/assets/b4376f01-e494-4a81-9956-2fdddf3586dd)

ward의 ac값이 가장 높은 것을 확인할 수 있습니다. 이는 ward의 방법이 가장 강력한 클러스터링 구조를 식별한다는 것을 의미합니다.

4개의 클러스터링 기법에서 target cluster를 찾았으면, 이제 두 번 이상 나온 지역을 추출합니다.
```
result1 <- as.vector(k_means_result)
result2 <- as.vector(k_med_result)
result3 <- as.vector(hcut_result)
result4 <- as.vector(gmm_result)
result_vector <- c(result1, result2, result3, result4)
value_counts <- table(result_vector)

result <- unique(result_vector[duplicated(result_vector) | duplicated(result_vector, fromLast = TRUE)]) # 중복된 값들만 추출한 후, 그 중에서도 고유한 값들만 반환(==두 번 이상 중복)
print(result)
```
![image](https://github.com/user-attachments/assets/6bf2c543-bd99-4376-b39c-b2fd0ec62268)

해당 지역들이 target으로 선정된 지역들입니다.

## 2. 회귀 분석 모델링
이제 해당 지역들이 있는 주유소에 추가적으로 급속 전기차 충전기를 설치를 고려합니다. 

다음과 같이 인구수 한 명당 급속 충전기, 친환경차 중 전기차 비율, 전기차 한 대당 급속 충전소 등의 파생 변수를 추가하여 급속 충전소 구축이 얼마나 잘 되어 있는지 비율로 확인하였습니다. (급속 충전기가 없는 지역은 제거)

![image](https://github.com/user-attachments/assets/b5ea6ee3-0708-4a0b-aed7-5fb3fb71342a)

## 2.1 다중공선성 제거
입지 기준을 설정하기 위해 종속 변수 Y = '급속충전기'일 때, 이 종속 변수를 가장 잘 설명하는 독립 변수를 확인합니다. 

그전에 다중공선성을 확인해야 합니다. 다중공선성은 회귀 분석 모델의 독립 변수들 간에 강한 상관관계가 나타나는 문제로서, 회귀 계수의 추정치가 불안정해지므로 어떤 변수의 회귀 계수가 실제로 종속 변수에 영향을 미쳤는지, 즉 독립 변수의 순수한 영향을 분리해서 확인하기 어렵게 만듭니다.

```
df_cor <- df%>%
  select(-법정동,-전기차.충전소.개수)
corr_df <- cor(df_cor)
corrplot(corr_df, method = 'number', order = 'hclust', type = 'lower', diag = FALSE)
```
![image](https://github.com/user-attachments/assets/1828fe27-58ad-4625-b28f-d442f277cc02)

```
set.seed(123)
model <- lm(급속충전기.대.~., df_cor)
vif(model)
```
![image](https://github.com/user-attachments/assets/6a5e6e28-0ef0-49c3-bbbe-50a7bff49573)

먼저 모든 변수로 회귀 분석 모델을 만든 다음, vif( ) 메서드를 통해 다중공선성을 확인했습니다. 그리고 VIF가 10 이상인 변수를 제거했습니다.

## 2.2 변수 선택
p개의 독립 변수들에 대해 만들어질 수 있는 가능한 모든 조합을 다 고려하는 방법인 best subset selection을 진행해서 Y = 급속충전기.대.~일 때, 가장 잘 설명할 수 있는 변수들의 조합을 찾앗습니다. 그 결과는 다음 그림과 같습니다.

![image](https://github.com/user-attachments/assets/6d419a77-8fca-41e4-a6cb-41e1f5660525)

변수들을 최대한 줄여서 PCA를 이용한 평가지표에 반영하는 것이 목표이기 때문에, 변수 개수에 민감하게 반응하는 BIC를 평가 지표로 선택했습니다.

```
#Performance measures
cbind( 
  RSS = summary(bestsub.model)$rss,
  Cp     = summary(bestsub.model)$cp,
  r2     = summary(bestsub.model)$rsq,
  Adj_r2 = summary(bestsub.model)$adjr2,
  BIC    =summary(bestsub.model)$bic
)
```
![image](https://github.com/user-attachments/assets/24d4a472-aa0b-4ba5-b8d7-a086cebbd066)

그러나 BIC 값의 차이가 유의미할 만큼 충분히 크지 않고, KMO 검정 결과 변수의 수가 늘어날수록 MSA 값이 높아졌습니다.
이런한 경우, 모델들 간의 관계가 Nested이므로 Partial F-Test를 통해 모델들 간에 통계적으로 유의미한 차이가 있는지를 확인했습니다.
```
lm2 <- lm(급속충전기.대.~인구수+완속충전기.대., data=df2)

lm3 <- lm(급속충전기.대.~인구수+완속충전기.대.
          +친환경차_전기차.비율, data=df2)

lm4 <- lm(급속충전기.대.~인구수+완속충전기.대.
          +전기차.등록.수
          +친환경차_전기차.비율, data=df2)

anova(lm2,lm3,lm4)
```
 ![image](https://github.com/user-attachments/assets/a2a7ac90-71ea-4f31-8b86-6054641883c1)

Model 2와 Model 3의 p 값이 유의하지 않은 것을 확인할 수 있습니다. 이는 Model 1에 변수를 추가적으로 더해도 모델 적합도가 향상되지 않는다는 것을 의미합니다. 그러므로 최종적으로 인구수와 완속충전기(대) 변수를 선택합니다.

## 3. Factor Analysis 
요인 분석은 변수들 중에서 영향력 있는 잠재 변수(요인)을 탐색하기 위해 사용하는 방법입니다. 이를 통해 변수들의 정보 손실을 최소화하면서 여러 개의 관련 있는 변수들을 소수의 요인(factor)으로 묶을 수 있습니다.
```
## 최종 model 선택
final_df <- df%>%
  select("법정동","인구수","완속충전기.대.")
```

```
fa.parallel(final_df[-1])
```
다음 그림은 fa.parallel( ) 메서드의 결과로, 병렬 분석을 통해 요인 또는 주성분의 개수를 나타낸 scree plot입니다.
이는 주성분 분석(PCA)에서 중요한 요소를 선택하기 위한 방법입니다.

![image](https://github.com/user-attachments/assets/40def90a-c17b-4db2-85d8-4e7879a0313c)

이 결과에 따라 요인 개수를 1개로 선택하였습니다.

요인 분석 결과를 시각화한 결과는 다음과 같습니다.
```
pa1 <- fa(final_df[-1], nfactors=1, fm="pa",max.iter = 100,rotate="oblimin")
fa.diagram(pa1,digits = 2)
```
![image](https://github.com/user-attachments/assets/e0d0bc18-d42c-4aa0-a4fb-1507302e6e2a)

'PA1'은 추출된 요인을 의미합니다. 변수 '인구수'와 '완속충전기.대'는 요인 PA1과 관련성이 있으며, 로딩 값이 0.48입니다. 로딩 값은 각 변수와 요인 간의 상관 관계를 나타냅니다. 

## 3. PCA를 이용한 평가지표 설정
### 3.1 PCA를 적용할 데이터 선정
요인 분석으로 '인구수 + 완속 충전기.대'를 통해 충전 환경이라는 잠재요인 변수를 생성하였습니다.
```
final_df$충전환경 <- final_df$인구수 * 0.48 + final_df$완속충전기.대. * 0.48
```

'충전환경'이라는 급속 충전기 잠재요인 변수 외에 급속 충전기,  공공 자전거 거치대, 택배함, 물류 창고 개수가 상대적으로 적은 곳으로 결과를 도출하기 위해 각 시설의 결핍률 변수를 추가했습니다. 각 결핍률 변수는 1 - (시설의 개수 - 인구 수)로 정의하였습니다.
```
final_df$급속충전기결핍률 <- 1-(df0$급속충전기.대./df0$인구수)
final_df$공공자전거거치대결핍률 <- 1-(df0$공공자전거.거치대수.LCD...QR./df0$인구수)
final_df$택배함결핍률 <- 1-(df0$택배함.개수/df0$인구수)
final_df$물류창고결핍률 <- 1-(df0$물류창고.개수/df0$인구수)
```

### 3.2 PCA를 이용한 평가 지표
PCA의 Loading을 평가 지표의 가중치로 사용하기 위해 PCA를 수행했습니다. PCA Loading은 주성분 혹은 요인들이 한 변수를 정의하는 선형 결합된 가중치 계수입니다.

```
set.seed(123)
final_pca2 <- prcomp(final[-1],scale=T)
summary(final_pca2)
```
![image](https://github.com/user-attachments/assets/5441555d-0145-4689-8f10-b0b07d372ddc)

주성분 결과는 위의 그림과 같습니다. 

주성분 선택은 통상적으로, 누적 분산 비율이 80% 이상이 되는 주성분 개수를 선택합니다. 90% 이상의 설명력을 원한다면 4개의 주성분을 선택해도 됩니다. 

다만, 주성분 3개일 때 전체 변동의 84.8% 정도를 설명하고 있고, 분산 비율도 PC4와 PC5를 합쳐야 15%이므로 주성분 3개를 선택합니다. 더 정확하게는 앞선 주성분들이 이미 대부분의 분산을 설명하고 있기 때문입니다.

이제 가중치로 사용할 PCA의 Loading 값을 확인합니다.
```
loadings <- final_pca2$rotation
round(loadings,2)
```
![image](https://github.com/user-attachments/assets/8417c83b-c031-4a62-921c-19b2a68d5d68)
Loading은 주성분에 대한 각 변수의 영향력을 의미하며 임의적인 부호를 갖습니다. 

주성분 3개를 선택했으므로 '충전환경' 변수에 대한 가중치는 0.08, '급속충전기결핍률' 변수에 대한 가중치는 0.72, '공공자전거거치대결핍률' 변수에 대한 가중치는 0.03, '택배함결핍률' 변수에 대한 가중치는 0.15, '물류창고결핍률' 변수에 대한 가중치는 0.68을 사용합니다.

Score = 0.08 x 충전환경 + 0.72 x 급속충전기결핍률 + 0.03 x 공공자전거거치대결핍률 + 0.15 x 택배함결핍률 + 0.68  x   물류창고결핍률

이 입지 점수를 이용해서 우선적으로 첨단 물류 복합 주유소가 필요한 '최종 입지'를 선정합니다.

```
final0$score <- final0$충전환경+final0$급속충전기+final0$공공자전거거치+final0$택배함+final0$물류창고
score <- final0%>%
  select("법정동","score")
score$입지점수 <- (score$score/max(score$score))*100

hist(score$입지점수)
```
![image](https://github.com/user-attachments/assets/7ed8b80f-77ca-498a-a0a9-b2970d8c1c02)

```
density_plot <- density(score$입지점수)
plot(density_plot)
```
![image](https://github.com/user-attachments/assets/51cfe4ed-a366-4a88-9b97-11c5fd2db889)

입지 점수의 분포를 고려해서 입지 점수가 80점 이상인 지역을 최종 입지로 선정하였습니다.

해당 지역들은 다음과 같습니다.
```
score <- score %>%
  filter(입지점수 >= 80) %>%
  arrange(desc(입지점수)) %>%
  select(법정동, 입지점수)
score
```
![image](https://github.com/user-attachments/assets/185e63e5-36dc-4925-a236-4be03044a1bb)

## 4. 최종 선택
이제 해당 지역들 중에서 다음 2가지를 고려해 첨단 물류 복합 주유소가 필요한 '주유소'를 선정합니다.
- (1) 가장 가까운 충전소끼리의 거리가 먼 주유소일수록 급속 충전기 설치가 필요
- (2) 첨단 물류 복합 주유소는 드론을 운영하므로, '비행 금지 구역'인지 확인 필요

첫 번째 사항은 Haversine Method를 이용해 각 주유소에서 해당 지역에 위치한 모든 충전소끼리의 거리를 계산합니다. 각 주유소에서 가장 가까운 충전소까지의 거리를 확인해서, 거리가 먼 주유소를 선정합니다.

두 번째 사항은 대통령 집무실이 용산으로 변경되면서 서울 비행 금지 구역이 변경되었습니다. 2023년 기준 비행 금지 구역 P73에 선정된 주유소가 있는지 확인합니다.


# 결과
![image](https://github.com/user-attachments/assets/02a5beb5-99ac-4a84-8be4-66df44f432c1)
