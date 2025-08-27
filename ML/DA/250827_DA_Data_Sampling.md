# Sampling 수행 여부 결정

Sampling을 진행하면 효과적으로 데이터 분석 및 모델링을 진행할 수 있음

- 데이터 Mart 생성시간을 단축 → 변수를 이용하기 위한 Mart 만들기
- 모델 학습 시간 단축

현재 데이터 상황에 맞는 샘플링 기법을 선정 후 수행하기

1. Time Series data 시간의 흐름에 따라 기록된 데이터
    1. up-sampling : 주기를 변경할 때, 결측값을 채우기
    2. down-sampling : time window 기준 mean(), max(), min()
2. Non-time Series data 비시계열 데이터, 시간의 흐름이 존재하지 않는 데이터
    1. target data가 너무 적다면 (class - imbalanced), oversampling (뻥튀기로 타겟률을 올리기)
    2. Data가 너무 많다면 Under-sampling 
    - under-sampling을 할 때는 Stratified Sampling (층화추출) 진행 → 각 특성 별로 데이터를 층을 내서 추출한다 → 모수의 특성을 잘 추출할 수 있음

## Over-Sampling

대표적인 기법: SMOTE (Synthetic Minority Over-sampling Technique) → 정답과 오답의 비율이 50:50

낮은 비율로 존재는 클래스의 데이터를 최급적 이웃 (K-NN) 알고리즘을 활용하여 새롭게 생성하는 방법

예를 들어, 데이터셋에 A 클래스는 많고 B 클래스는 너무 적으면 데이터 불균형 문제로, 머신러닝 모델이 A만 학습하고 B는 무시하게 되는 현상이 생길 수 있다

→ 스모트 방식은 B 데이터 중 하나를 고르고 그 데이터와 근접한 데이터를 찾은 뒤 그 둘 사이에 중간 지점을 골라서 가짜 데이터를 새로 만들어 준다. 

원래 B 클래스 데이터가 점 **(2, 2)**, **(4, 2)** 에 있다고 하면, SMOTE는 그 사이 어딘가, 예를 들어 **(3, 2)** 같은 새로운 점을 만들어냄

## Under Sampling & Stratified Sampling

층화추출: 모집단을 동질적인 소집단 (성별, 연령대 등)들로 층화를 시키고, 그 집단의 크기에 따라 단순 무작위 표본 추출 방법(random sampling) → 내가 프로젝트에서 할 샘플링의 집단은 월별로 데이터를 나눌 예정

### 장점

- 단순 임의 추출보다 신뢰성이 높은 추정치를 구할 수 있음
- 비용 절감 및 자료 분석이 용이함
- 각 층에 대한 추정치를 부수적으로 구할 수 있음

### 단점

- 방대한 모집단의 경우 목록 작성의 어려움
- 적절하게 층을 나누기 위해서는 모집단 각 층에 대한 정확한 정보가 필요

모수를 기준으로 샘플링한 데이터 셋도 동일한가에 대한 비교

[image link](https://blog.kakaocdn.net/dna/nEv3E/btsiSQ5L9wK/AAAAAAAAAAAAAAAAAAAAAK9RlpkfEuh2vY84Go-rOGcffHsILNiQNOkvzvPhmH0V/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=W2JftAm6Ldrp1UPrxNj53f4B8vM%3D)

---

## Time Series

## Up-sampling

```python
# 타임 시리즈의 데이터 프레임 만들기

import pandas as pd
time_index = pd.date_range("2023-01-01", periods = 3, freq = "10s")
df_time = pd.DateFrame({'value' : [1.2.3]}, index = time_index)
df_time
```

|  | **value** |
| --- | --- |
| **2023-01-01 00:00:00** | 1 |
| **2023-01-01 00:00:10** | 2 |
| **2023-01-01 00:00:20** | 3 |

```python
# 타임 시리즈 실습 (업 샘플링)

df_upsample = df_time.resample('5S').mean()
df_upsample
```

2023-01-01 00:00:00	1.0
2023-01-01 00:00:05	NaN
2023-01-01 00:00:10	2.0
2023-01-01 00:00:15	NaN
2023-01-01 00:00:20	3.0

→ 빈 값이 생기면, 어떻게 처리하냐에 따라 데이터가 달라지기 때문에 주의

### 빈 값을 채우는 여러가지 방법

```python
# 1. ffill - 앞에 있는 정보를 넣어주기
df_upsample.fillna(method = "ffill")

# 2. bfill - 뒤에 있는 정보를 넣어주기
df_upsample.fillna(method = "bfill")

# 3. fill(0) - 0으로 대신 넣어주기
df_upsample.fillna(0)

#4. mean() - 평균값 넣어주기
df_upsample.fillna(df_upsample.mean())

# linear interpolation - 가장 일반적인 방법, 선형 보관 -> 중간에 오는 애를 넣어주기
df_upsample.interpolate()
```

interpolate 출력 결과

|  | value |
| --- | --- |
| **2023-01-01 00:00:00** | 1.0 |
| **2023-01-01 00:00:05** | 1.5 |
| **2023-01-01 00:00:10** | 2.0 |
| **2023-01-01 00:00:15** | 2.5 |
| **2023-01-01 00:00:20** | 3.0 |

## Down-sampling

```python
df_upsample.resample('10S').mean()
```

|  | value |
| --- | --- |
| **2023-01-01 00:00:00** | 1.0 |
| **2023-01-01 00:00:10** | 2.0 |
| **2023-01-01 00:00:20** | 3.0 |

---

## Non-time Series

## SMOTE (over-sampling)

```python
df_all.head()
```

|  | **bsym** | **CustomerID** | **target** |
| --- | --- | --- | --- |
| **0** | 2010-12 | 17850.0 | 0.0 |
| **1** | 2010-12 | 13047.0 | 0.0 |
| **2** | 2010-12 | 12583.0 | 1.0 |
| **3** | 2010-12 | 13748.0 | 0.0 |
| **4** | 2010-12 | 15100.0 | 1.0 |

```python
# target ratio 확인
df_all['target'].sum()/len(df_all)

# `np.float64(0.3712218649517685)`
```

```python
df_all['target'].value_counts
```

|  |  |
| --- | --- |
| **target** | count |
| **0.0** | 7822 |
| **1.0** |  4618 |

```python
# 샘플링을 하기 위한 X, Y data split
# X - Customer ID, Y = Target
X = df_all.drop(['bsym', 'target'], axis = 1)
Y = df_all['target']

# df_all['customerID']로 뽑으면 Series가 됨
# 모델 학습 시 보통 DataFrame 형태가 후처리가 더 안정적이라 굳이 drop을 쓰는 것

# SMOTE 활용 Over-samplingfrom imblearn.over_sampling import SMOTE
smote = SMOTE(random_state=42)X_over, y_over = smote.fit_resample(X, Y)
print("SMOTE 적용 전 학습용 피처/레이블 데이터 세트 : ", X.shape, Y.shape)
print('SMOTE 적용 후 학습용 피처/레이블 데이터 세트 :', X_over.shape, y_over.shape)
print('SMOTE 적용 후 값의 분포 :\n',pd.Series(y_over).value_counts() )

# random_state: 난수(random number), 결과를 항상 똑같이 재현하고 싶을 때 설정하는 난수 시드
# 보통 관례적으로 42를 많이 씀

# target과 non-target data가 5:5
pd.Series(y_over).value_counts(normalize=True)
```

|  |  |
| --- | --- |
| **target** | proportion |
| **0.0** | 0.5 |
| **1.0** | 0.5 |

## Stratified Sampling

```python
df_all.head()
df_target

# 층화 추출의 기준을 각 월로 잡음

# 각 월 별 차지하는 비중을 계산
# target_ratio / total_ratio col 생성
df_target['total_ratio'] = df_target['Total'] / df_target['Total'].sum()d
f_target.columns = ['bsym', 'y', 'total_cnt', 'target_ratio', 'total_ratio']
df_target

# df_target의 열 이름을 통째로 바꿔버림
# 네 번째 열 ratio 이름을 target_ratio로 바꾸고 거기다가 total_ratio까지 추가해서 출력

# Total(sum)행 추가
df_target = pd.concat([df_target, pd.DataFrame({'bsym': ['total']
          , 'y': [df_target['y'].sum()]
          , 'total_cnt': [df_target['total_cnt'].sum()]
          , 'target_ratio' : [df_target['y'].sum() / df_target['total_cnt'].sum()]
          , 'total_ratio' : [1]})], ignore_index=True)

df_target

# 'bsym': ['total'] 종목(symbol) 대신 "total"이라는 문자열
# 전체 y / total_cnt 비율 (즉 평균적인 target 비율)
# 'total_ratio': [1] 전체 비중이니까 항상 1
# ignore_index=True 기존 인덱스는 무시하고 0,1,2,... 새 인덱스

# 전체 Data set에서 30% 비례 층화 추출
# 30% 로 정하지 않아도 컴퓨터가 돌릴 수 있을만한 수준으로만 추출하면 됨
len(df_all) * 0.3
```

| **bsym** | **y** | **total_cnt** | **target_ratio** | **total_ratio** |
| --- | --- | --- | --- | --- |
| **0** | 2010-12 | 324.0 | 885 | 0.366102 |
| **1** | 2011-01 | 262.0 | 741 | 0.353576 |
| **2** | 2011-02 | 290.0 | 758 | 0.382586 |
| **3** | 2011-03 | 304.0 | 974 | 0.312115 |
| **4** | 2011-04 | 368.0 | 856 | 0.429907 |
| **5** | 2011-05 | 410.0 | 1056 | 0.388258 |
| **6** | 2011-06 | 365.0 | 991 | 0.368315 |
| **7** | 2011-07 | 388.0 | 949 | 0.408851 |
| **8** | 2011-08 | 425.0 | 935 | 0.454545 |
| **9** | 2011-09 | 489.0 | 1266 | 0.386256 |
| **10** | 2011-10 | 622.0 | 1364 | 0.456012 |
| **11** | 2011-11 | 371.0 | 1665 | 0.222823 |
| **12** | total | 4618.0 | 12440 | 0.371222 |

```python
# 30% sampling을 한다고 가정했을 때, 각 월 별 비중과 동일하게 Random sampling 하는 것이 중요
# sample_n : 비례층화추출을 위한 월 별 추출 개수
df_target['sample_n_total'] = len(df_all) * 0.3
df_target['sample_n'] = df_target['sample_n_total'] * df_target['total_ratio']
df_target['sample_n'] = df_target['sample_n'].astype(int)df_target
# 추출한 총 개수 * 전체 데이터 비율 = 샘플(추출) 개수

# ▶ 월 별 비중을 유지하면서 Random sampling 진행
rnd_sate = 1234
# 랜덤 시드(seed) 고정 → 재실행해도 똑같은 샘플이 나오도록 설정

df_all_00 = df_all[df_all['bsym'] == '2010-12'].sample(n=265, random_state = rnd_sate)
df_all_01 = df_all[df_all['bsym'] == '2011-01'].sample(n=222, random_state = rnd_sate)
df_all_02 = df_all[df_all['bsym'] == '2011-02'].sample(n=227, random_state = rnd_sate)
df_all_03 = df_all[df_all['bsym'] == '2011-03'].sample(n=292, random_state = rnd_sate)
df_all_04 = df_all[df_all['bsym'] == '2011-04'].sample(n=256, random_state = rnd_sate)
df_all_05 = df_all[df_all['bsym'] == '2011-05'].sample(n=316, random_state = rnd_sate)
df_all_06 = df_all[df_all['bsym'] == '2011-06'].sample(n=297, random_state = rnd_sate)
df_all_07 = df_all[df_all['bsym'] == '2011-07'].sample(n=284, random_state = rnd_sate)
df_all_08 = df_all[df_all['bsym'] == '2011-08'].sample(n=280, random_state = rnd_sate)
df_all_09 = df_all[df_all['bsym'] == '2011-09'].sample(n=379, random_state = rnd_sate)
df_all_10 = df_all[df_all['bsym'] == '2011-10'].sample(n=409, random_state = rnd_sate)
df_all_11 = df_all[df_all['bsym'] == '2011-11'].sample(n=499, random_state = rnd_sate)
df_all_sample = pd.concat([df_all_00, df_all_01, df_all_02, df_all_03,
df_all_04, df_all_05,df_all_06, df_all_07, df_all_08, df_all_09, df_all_10, df_all_11],
 axis=0)
df_all_sample
```

| **bsym** | **y** | **total_cnt** | **target_ratio** | **total_ratio** | **sample_n_total** | **sample_n** |
| --- | --- | --- | --- | --- | --- | --- |
| **0** | 2010-12 | 324.0 | 885 | 0.366102 | 0.071141 | 3732.0 |
| **1** | 2011-01 | 262.0 | 741 | 0.353576 | 0.059566 | 3732.0 |
| **2** | 2011-02 | 290.0 | 758 | 0.382586 | 0.060932 | 3732.0 |
| **3** | 2011-03 | 304.0 | 974 | 0.312115 | 0.078296 | 3732.0 |
| **4** | 2011-04 | 368.0 | 856 | 0.429907 | 0.068810 | 3732.0 |
| **5** | 2011-05 | 410.0 | 1056 | 0.388258 | 0.084887 | 3732.0 |
| **6** | 2011-06 | 365.0 | 991 | 0.368315 | 0.079662 | 3732.0 |
| **7** | 2011-07 | 388.0 | 949 | 0.408851 | 0.076286 | 3732.0 |
| **8** | 2011-08 | 425.0 | 935 | 0.454545 | 0.075161 | 3732.0 |
| **9** | 2011-09 | 489.0 | 1266 | 0.386256 | 0.101768 | 3732.0 |
| **10** | 2011-10 | 622.0 | 1364 | 0.456012 | 0.109646 | 3732.0 |
| **11** | 2011-11 | 371.0 | 1665 | 0.222823 | 0.133842 | 3732.0 |
| **12** | total | 4618.0 | 12440 | 0.371222 | 1.000000 | 3732.0 |

```python
# 층화추출이 정상적으로 되었는지, 모수의 target ratio와 비교
df_all_sample_temp = df_all_sample.groupby(['bsym'])['target'].agg(['sum', 'count']).reset_index()
#df_all_sample : 층화추출로 뽑아온 표본 데이터
#월(bsym)별로 target값의 1(성공) 0(실패) 합(sum), **개수(count)**를 집계

df_all_sample_temp['target_ratio'] = df_all_sample_temp['sum'] / df_all_sample_temp['count']
df_all_sample_temp['target_ratio_mosu'] = df_target['target_ratio']
df_all_sample_temp.style.bar(subset=['target_ratio','target_ratio_mosu'], align='zero', vmin=0, vmax=0.8)
# 기준점 0, 0~0.8 범위에서 색깔/길이 표현

# sample 전체 Target ratio (※ mosu 37%)
df_all_sample_temp['sum'].sum() / df_all_sample_temp['count'].sum()
# 모수와 유사한 결과값 38% 나옴

	df_all_sample.shape
```
