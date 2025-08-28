# 04 Data Mart, Engineering

# 01. 모델링을 위한 Data Mart 기획, 설계

![image.png](attachment:feb61690-7e95-4f82-9e7f-1ae305596100:image.png)

[image link](https://drive.google.com/uc?id=1XB6sIcr1m1_5b6mvrFqun8iaMbOv4pkH)

- Sampling 완료 후 월 별 기준이 되는 CustomerID를 추출한 상태
- Mart를 구성하기 위한 준비가 모두 완료된 상태
- 가설을 수립하고, 가설에 해당하는 다양한 변수 Category와 List를 생성
- 풍부한 Data Mart를 기획해야 좋은 모델의 성능과 다양한 해석이 가능, 데이터 가져오기도 편함

![image.png](attachment:010fb807-af9d-45c6-89a9-f4984c8e68d5:image.png)

[image_link2](https://drive.google.com/uc?id=1BbiKwpqO20wrAx_yCZJ9AKXyYQrL7Tig)

---

# 02. **Data 추출 및 Mart 개발**

- 상위 단계에서 정의한 Data Mart를 기반으로 변수를 추출하는 과정
- 변수 추출은 Sampling한 Data로 진행\

## 개발 준비

```python
# 최초 Data에서 bsym(기준년월)을 추가한 origin data
df_origin.head()

# Sampling한 CustomerID 대상 Mart 개발 수행
df_all_sample.head()

# 원본(Origin) data에서 Sampling 하기 위한 key 생성 (1)
df_origin['sample_key'] = df_origin['bsym'].astype(str) + df_origin['CustomerID'].astype(str)
df_origin.head()

# 원본(Origin)에서 Sampling 하기 위한 key 생성 (2)
df_all_sample['sample_key'] = df_all_sample['bsym'].astype(str) + df_all_sample['CustomerID'].astype(str)
df_all_sample.head()

# sampling 된 CustomerID만 추출
df_origin_sample = df_origin[df_origin['sample_key'].isin(df_all_sample['sample_key'])]
# amt col 생성
df_origin_sample['amt'] = df_origin_sample['UnitPrice'] * df_origin_sample['Quantity']
df_origin_sample.head()

df_origin.shape, df_origin_sample.shape
```

## 추출 시작

```python
# 총 구매 금액 (total_amt)
# reset_index() 기존에 만든 인덱스 초기화 시켜줌
df_mart = df_origin_sample.groupby(['bsym', 'CustomerID'])['amt'].agg(total_amt = ('sum')).reset_index()
df_mart

# 변수 생성 시작 (max_amt, min_amt)
# pd.DataFrame(df_origin_sample.groupby(['bsym', 'CustomerID', 'InvoiceNo'])['amt'].agg(amt = ('sum')).reset_index()).groupby(['bsym', 'CustomerID'])['amt'].agg(max_amt = ('max')).reset_index()
df_mart[['max_amt', 'min_amt']] = pd.DataFrame(df_origin_sample.groupby(['bsym', 'CustomerID', 'InvoiceNo'])['amt'].agg(amt = ('sum')).reset_index())
df_mart.groupby(['bsym', 'CustomerID'])['amt'].agg(max_amt = ('max'), min_amt = ('min')).reset_index()[['max_amt', 'min_amt']]
df_mart

# 변수 생성 시작 (total_cnt)
df_mart['total_cnt'] = pd.DataFrame(df_origin_sample.groupby(['bsym', 'CustomerID'])['InvoiceNo'].agg(total_cnt = ('nunique'))).reset_index()['total_cnt']
df_mart
# bsym, CustomerID별로 묶음
# InvoiceNo의 중복을 제거한 개수를 센 것 즉, 고객별(월별) 고유한 거래 건수

#  변수 생성 시작 (max_cnt, min_cnt)
df_mart[['max_cnt', 'min_cnt']] = pd.DataFrame(df_origin_sample.groupby(['bsym', 'CustomerID', 'InvoiceNo'])['StockCode'].agg(stock_cnt = 'nunique').reset_index())
df_mart.groupby(['bsym', 'CustomerID'])['stock_cnt'].agg(max_cnt = ('max'), min_cnt = ('min')).reset_index()[['max_cnt', 'min_cnt']]
df_mart

# 변수 생성 시작 (total_qty, max_qty, min_qty)
df_mart[['total_qty', 'max_qty', 'min_qty']] = df_origin_sample.groupby(['bsym', 'CustomerID'])['Quantity'].agg(total_qty = 'sum', max_qty = 'max', min_qty = 'min').reset_index()[['total_qty', 'max_qty', 'min_qty']]
df_mart

# 변수 생성 시작 (country)
df_mart['Country'] = df_origin_sample.groupby(['bsym', 'CustomerID'])['Country'].agg(country = 'first').reset_index()['country']
df_mart
# groupby 집계할 때, 그룹 안의 첫 번째 값을 가져와라

# ▶ target join
df_mart = pd.merge(df_mart, df_all_sample[['bsym', 'CustomerID', 'target']], how='left', on=['bsym', 'CustomerID'])
df_mart

# target은 원래 df_mart에 없었음
# df_mart는 금액(amt) 관련 집계만 있지, target은 들어있지 않음

# 키(bsym, CustomerID)는 “공통 기준"
# 따라서 이 두 컬럼으로 merge해야 동일 고객·동일 월 데이터끼리 매칭 가능
# 모든 고객-월 조합을 보존하면서, 있는 경우에만 target 값을 채워줌

# 최종 Data set
df_mart.shape
```

| **bsym** | **CustomerID** | **total_amt** | **max_amt** | **min_amt** | **total_cnt** | **max_cnt** | **min_cnt** | **total_qty** | **max_qty** | **min_qty** | **Country** | **target** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **0** | 2010-12 | 12370.0 | 1864.27 | 1587.07 | 277.20 | 2 | 82 | 8 | 967 | 96 | 1 | Cyprus |
| **1** | 2010-12 | 12383.0 | 600.72 | 600.72 | 600.72 | 1 | 37 | 37 | 754 | 100 | 3 | Belgium |
| **2** | 2010-12 | 12395.0 | 679.92 | 346.10 | 333.82 | 2 | 19 | 12 | 753 | 120 | 1 | Belgium |
| **3** | 2010-12 | 12417.0 | 291.34 | 291.34 | 291.34 | 1 | 11 | 11 | 60 | 16 | 2 | Belgium |
| **4** | 2010-12 | 12481.0 | 791.34 | 631.96 | 159.38 | 2 | 12 | 10 | 319 | 48 | 1 | Germany |
| **...** | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| **3721** | 2011-11 | 18188.0 | 392.76 | 392.76 | 392.76 | 1 | 13 | 13 | 268 | 60 | 4 | United Kingdom |
| **3722** | 2011-11 | 18205.0 | 291.60 | 291.60 | 291.60 | 1 | 17 | 17 | 116 | 24 | 2 | United Kingdom |
| **3723** | 2011-11 | 18209.0 | 139.10 | 139.10 | 139.10 | 1 | 8 | 8 | 58 | 12 | 3 | United Kingdom |
| **3724** | 2011-11 | 18230.0 | 663.84 | 343.80 | 320.04 | 2 | 11 | 6 | 286 | 48 | 4 | United Kingdom |
| **3725** | 2011-11 | 18274.0 | 175.92 | 175.92 | 175.92 | 1 | 11 | 11 | 88 | 18 | 1 | United Kingdom |

3726 rows × 13 columns

---

## Integer Feature Engineering

모델링의 목적인 성능 향상을 위해  Feature을 생성, 선택, 가공하는 모든 활동

```
Target 변수와 의미있는 변수를 선택하는 방법

1. Regression 회귀: 상관계수 분석을 통한 유의미한 변수 선택
2. Classification 분류: bin(통)으로 구분 후 Target 변수와의 관계 파악
```

```
Information Value (IV)
Feature 하나가 Good(Target)과 Bad(Non-Target)을 잘 구분해줄 수 있는지에 대한 정보량을
표현하는 방법론

IV 수치가 크면 클수록 타겟과 타겟이 아닌 것을 잘 구분하는 정보량이 많은 Feature이고,
IV 수치가 작을수록 정보량이 적은 Feature

(Target data 구성비 - Non-target data 구성비) * WoE

WoE (Weight of Evidence) = ln(Target data 구성비/Non-target data 구성비)
ln(자연로그)를 취하는 이유 - WoE의 최대, 최소값의 범위를 맞춰주기 위해서 (1~∞ , -∞~1)
```

## Regression (corr)

```python
df_mart.head()

# heatmap plot
import seaborn as sns

df_pair = df_mart[['total_amt', 'max_amt', 'min_amt', 'total_cnt','max_cnt', 'min_cnt', 'total_qty', 'max_qty', 'min_qty']]
mask = np.triu(np.ones_like(df_pair.corr(), dtype=np.bool)
sns.heapmap(df_pair.corr(), vmin = -1, vmax = +1, annot = True, cmap = "coolwarm", mask = mask);
plt.gcf().set_size_inches(10,10)

```

![image.png](attachment:9993a284-11fc-482b-8733-3b12033cb635:image.png)

## Classification (IV)

```python
# Binning
total_amt_quantile  = df_mart['total_amt'].quantile(q=list(np.arange(0.05,1,0.05)), interpolation = 'nearest')
# [0.05, 0.10, 0.15, …, 0.95] - 분위수로 나누고, nearest 정수/실제 데이터에 맞춰지도록 설정
# total_amt_quantile

# 그룹 초기화
df_mart['total_amt_grp'] = 0

# 40%, 75% 기준으로 총 3개 구간화 진행
for i in  range(len(df_mart['total_amt'])) :
  if df_mart.loc[i, "total_amt"] <= total_amt_quantile.iloc[7] : # 40%
    df_mart.loc[i, 'total_amt_grp'] = 1
  elif df_mart.loc[i, "total_amt"] <= total_amt_quantile.iloc[14] : # 75%
    df_mart.loc[i, 'total_amt_grp'] = 2
  else : df_mart.loc[i, 'total_amt_grp'] = 3

# 1번 그룹 : 40% 분위수 이하 (소비금액이 낮은 고객)
# 2번 그룹 : 40%~75% 사이 (중간 수준 고객)
# 3번 그룹 : 75% 초과 (상위 고객, VIP 느낌)

df_mart.groupby('total_amt_grp')['target'].agg(['sum', 'count'])

```

|  | sum | count |
| --- | --- | --- |
| total_amt_grp |  |  |
| 1 | 422.0 | 1491 |
| 2 | 522.0 | 1304 |
| 3 | 492.0 | 931 |

```python
# non-target labeling
df_mart['n_target'] = np.where(df_mart['target']==0, 1, 0)
# target이 0이면 n_target=1, 아니면 0

# target , non-target count
iv = df_mart.groupby('total_amt_grp')[['target', 'n_target']].sum().reset_index()
# 각 구간별 Good / Bad 분포 테이블 생성

# good_pct / bad_pct 계산
iv['good_pct'] = iv['target'] / iv['target'].sum()
iv['bad_pct'] = iv['n_target'] / iv['n_target'].sum()

# t_ratio
iv['t_ratio'] = iv['target'] / (iv['target'] + iv['n_target'])

iv
```

| total_amt_grp | target | n_target | good_pct | bad_pct | t_ratio |
| --- | --- | --- | --- | --- | --- |
| 0 | 1 | 422.0 | 1069 | 0.293872 | 0.466812 |
| 1 | 2 | 522.0 | 782 | 0.363510 | 0.341485 |
| 2 | 3 | 492.0 | 439 | 0.342618 | 0.191703 |

```python
# Group별 IV 계산import math
for i in range(len(iv['total_amt_grp'])) : 
    iv.loc[i, 'iv'] = (iv.loc[i,'good_pct'] - iv.loc[i,'bad_pct']) * math.log(iv.loc[i,'good_pct']/iv.loc[i,'bad_pct'])
iv

# (Target data 구성비 - Non-target data 구성비) * WoE
# WoE (Weight of Evidence) = ln(Target data 구성비/Non-target data 구성비)
```

|  | total_amt_grp | target | n_target | good_pct | bad_pct | t_ratio | iv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1 | 422.0 | 1069 | 0.293872 | 0.466812 | 0.283032 | 0.080034 |
| 1 | 2 | 522.0 | 782 | 0.363510 | 0.341485 | 0.400307 | 0.001377 |
| 2 | 3 | 492.0 | 439 | 0.342618 | 0.191703 | 0.528464 | 0.087632 |

## IV 구하기 자동화

```python
# for문 활용 자동화
integer_list = ['total_amt', 'max_amt', 'min_amt', 'total_cnt','max_cnt', 'min_cnt', 'total_qty', 'max_qty', 'min_qty']
iv_df = []

for val in integer_list :
    #  Binning
    quantile  = df_mart[val].quantile(q=list(np.arange(0.05,1,0.05)), interpolation = 'nearest')

    #  그룹 초기화
    df_mart['grp'] = 0

    #  40%, 75% 기준으로 총 3개 구간화 진행
    for i in  range(len(df_mart[val])) :
      if df_mart.loc[i, val] <= quantile.iloc[7] :
        df_mart.loc[i, 'grp'] = 1
      elif df_mart.loc[i, val] <= quantile.iloc[14] :
        df_mart.loc[i, 'grp'] = 2
      else : df_mart.loc[i, 'grp'] = 3

    #  non target labeling
    df_mart['n_target'] = np.where(df_mart['target']==0, 1, 0)

    #  target , non-target count
    iv = df_mart.groupby('grp')[['target', 'n_target']].sum().reset_index()

    #  good_pct / bad_pct 계산
    iv['good_pct'] = iv['target'] / iv['target'].sum()
    iv['bad_pct'] = iv['n_target'] / iv['n_target'].sum()

    #  t_ratio
    iv['t_ratio'] = iv['target'] / (iv['target'] + iv['n_target'])

    iv['val'] = val

    for i in range(len(iv['grp'])) :
      iv.loc[i, 'iv'] = (iv.loc[i,'good_pct'] - iv.loc[i,'bad_pct']) * math.log(iv.loc[i,'good_pct']/iv.loc[i,'bad_pct'])

    iv_df.append(iv)

iv_df=pd.concat(iv_df)
iv_df
```

|  | grp | target | n_target | good_pct | bad_pct | t_ratio | val | iv |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1 | 422.0 | 1069 | 0.293872 | 0.466812 | 0.283032 | total_amt | 0.080034 |
| 1 | 2 | 522.0 | 782 | 0.363510 | 0.341485 | 0.400307 | total_amt | 0.001377 |
| 2 | 3 | 492.0 | 439 | 0.342618 | 0.191703 | 0.528464 | total_amt | 0.087632 |
| 0 | 1 | 441.0 | 1050 | 0.307103 | 0.458515 | 0.295775 | max_amt | 0.060688 |
| 1 | 2 | 547.0 | 757 | 0.380919 | 0.330568 | 0.419479 | max_amt | 0.007139 |
| 2 | 3 | 448.0 | 483 | 0.311978 | 0.210917 | 0.481203 | max_amt | 0.039562 |
| 0 | 1 | 536.0 | 955 | 0.373259 | 0.417031 | 0.359490 | min_amt | 0.004854 |
| 1 | 2 | 528.0 | 776 | 0.367688 | 0.338865 | 0.404908 | min_amt | 0.002353 |
| 2 | 3 | 372.0 | 559 | 0.259053 | 0.244105 | 0.399570 | min_amt | 0.000888 |
| 0 | 1 | 907.0 | 1924 | 0.631616 | 0.840175 | 0.320381 | total_cnt | 0.059508 |
| 1 | 3 | 529.0 | 366 | 0.368384 | 0.159825 | 0.591061 | total_cnt | 0.174156 |
| 0 | 1 | 522.0 | 977 | 0.363510 | 0.426638 | 0.348232 | max_cnt | 0.010109 |
| 1 | 2 | 551.0 | 767 | 0.383705 | 0.334934 | 0.418058 | max_cnt | 0.006630 |
| 2 | 3 | 363.0 | 546 | 0.252786 | 0.238428 | 0.399340 | max_cnt | 0.000840 |
| 0 | 1 | 632.0 | 959 | 0.440111 | 0.418777 | 0.397234 | min_cnt | 0.001060 |
| 1 | 2 | 489.0 | 733 | 0.340529 | 0.320087 | 0.400164 | min_cnt | 0.001266 |
| 2 | 3 | 315.0 | 598 | 0.219359 | 0.261135 | 0.345016 | min_cnt | 0.007283 |
| 0 | 1 | 449.0 | 1043 | 0.312674 | 0.455459 | 0.300938 | total_qty | 0.053707 |
| 1 | 2 | 517.0 | 787 | 0.360028 | 0.343668 | 0.396472 | total_qty | 0.000761 |
| 2 | 3 | 470.0 | 460 | 0.327298 | 0.200873 | 0.505376 | total_qty | 0.061720 |
| 0 | 1 | 569.0 | 1123 | 0.396240 | 0.490393 | 0.336288 | max_qty | 0.020072 |
| 1 | 2 | 464.0 | 697 | 0.323120 | 0.304367 | 0.399655 | max_qty | 0.001121 |
| 2 | 3 | 403.0 | 470 | 0.280641 | 0.205240 | 0.461627 | max_qty | 0.023592 |
| 0 | 1 | 598.0 | 1009 | 0.416435 | 0.440611 | 0.372122 | min_qty | 0.001364 |
| 1 | 2 | 567.0 | 868 | 0.394847 | 0.379039 | 0.395122 | min_qty | 0.000646 |
| 2 | 3 | 271.0 | 413 | 0.188719 | 0.180349 | 0.396199 | min_qty | 0.000380 |

```python
# ▶ Feature별 전체 IV 값을 sum해서 비교
iv_df.groupby('val')['iv'].sum().sort_values(ascending=False)
```

|  | iv |
| --- | --- |
| val |  |
| total_cnt | 0.233664 |
| total_amt | 0.169042 |
| total_qty | 0.116188 |
| max_amt | 0.107388 |
| max_qty | 0.044786 |
| max_cnt | 0.017578 |
| min_cnt | 0.009608 |
| min_amt | 0.008095 |
| min_qty | 0.002390 |

**dtype:** float64

## Categorical Feature Engineering

```python
Target 변수와 의미있는 변수를 선택하는 방법
-> 차원이 너무 많은 경우에는 차원을 도메인 지식을 활용하여 축소할 것

Catplot
Categorical Plot: 색상(hue)과 행을 동시에 사용하여 3개 이상의 카테고리 값에 의한 변화를 그림
카테고리형 데이터 파악에 용이
```

```python
import seaborn as sns
import metplotlib.pyplot as plt
%matplotlib inlineplt.style.use(['dark_background'])

# target 기준(hue,색상)으로 구분하여 Plot 만들기
sns.catplot(x="Country", hue="target", kind="count",palette="pastel", edgecolor=".6",data=df_mart);plt.xticks(rotation=-20)
plt.gcf().set_size_inches(25, 3)
```

![image.png](attachment:7eb72c54-3042-42ef-bec2-ef5cc330f7f5:image.png)

```python
# UK(영국) 이외는 기타국가로 처리
df_mart['Country'] = np.where(df_mart['Country'] == 'United Kingdom', 'UK', 'ETC')
df_mart['Country'].value_counts()
```

|  | count |
| --- | --- |
| Country |  |
| UK | 3372 |
| ETC | 354 |

**dtype:** int64

```python
# target , non-target count
iv = df_mart.groupby('Country')[['target', 'n_target']].sum().reset_index()

# good_pct /bad_pct 계산
iv['good_pct'] = iv['target'] / iv['target'].sum()
iv['bad_pct'] = iv['n_target'] / iv['n_target'].sum()

# t_ratio
iv['t_ratio'] = iv['target'] / (iv['target'] + iv['n_target'])

iv['val'] = 'Country'

for i in range(len(iv['Country'])) :
  iv.loc[i, 'iv'] = (iv.loc[i,'good_pct'] - iv.loc[i,'bad_pct']) * math.log(iv.loc[i,'good_pct']/iv.loc[i,'bad_pct'])

iv
```

     Country 	target 	n_target 	good_pct 	bad_pct 	t_ratio 	val 	iv
0 	ETC 	133.0 	221 	0.092618 	0.096507 	0.375706 	Country 	0.000160
1 	UK 	1303.0 	2069 	0.907382 	0.903493 	0.386418 	Country 	0.000017

```python
# 많은 정보량을 가지고 있지 않음
iv['iv'].sum()

# np.float64(0.00017659066476091975)

# 최종 Mart 저장
df_mart = pd.DataFrame(df_mart)
df_mart.to_csv('df_mart.csv', index=False)
```

### IV(Information Value) 기준

일반적으로 크레딧 스코어링, 머신러닝 피처 엔지니어링에서 아래 기준을 많이 씀

| IV 값 범위 | 변수 설명력 (정보량) | 해석 |
| --- | --- | --- |
| **< 0.02** | 거의 없음 | 타겟과 거의 무관 |
| **0.02 ~ 0.1** | 약함 | 조금 관련 있지만 약함 |
| **0.1 ~ 0.3** | 중간 정도 | 유용한 변수 |
| **0.3 ~ 0.5** | 강함 | 타겟과 높은 관련 |
| **> 0.5** | 과적합 위험 | 너무 강함, 실제로는 드뭄 |

수동으로 quantile 기준(40%, 75%)으로 binning 하면, **타겟 분포를 잘 못 잡아서 IV 값이 작게 나올 수 있어** **Optimal Binning**을 써보자

### Optimal Binning

- 변수 값을 **타겟(0/1) 분포에 따라 자동으로 구간화**하는 방법
- 목표: 각 bin 안에서 **타겟과 비타겟의 비율 차이(WoE 차이)**를 극대화 → IV 값 상승

```python
!pip install optbinning

from optbinning import OptimalBinning

integer_list = ['total_amt', 'max_amt', 'min_amt', 'total_cnt','max_cnt', 'min_cnt', 'total_qty', 'max_qty', 'min_qty']
iv_df = []

for i in integer_list :
  variable = i
  x = df_mart[variable].values
  y = df_mart.target
  # max_n_prebins
  optb = OptimalBinning(name=variable, dtype="numerical", solver="cp", max_n_prebins=3)
  optb.fit(x, y)
  # print("split points : ", optb.splits)

  binning_table = optb.binning_table
  v1 = binning_table.build()

  df = pd.DataFrame({'val' : variable,
                     'IV' : [v1.loc['Totals','IV']]})
  iv_df.append(df)

iv_df = pd.concat(iv_df).reset_index(drop=True)
iv_df.sort_values(by=['IV'], ascending = False)

# binning_table.plot(metric="event_rate")
```

|  | val | IV |
| --- | --- | --- |
| 3 | total_cnt | 0.283918 |
| 0 | total_amt | 0.203491 |
| 6 | total_qty | 0.160114 |
| 1 | max_amt | 0.128935 |
| 7 | max_qty | 0.079832 |
| 4 | max_cnt | 0.032054 |
| 2 | min_amt | 0.024855 |
| 5 | min_cnt | 0.017754 |
| 8 | min_qty | 0.009636 |

아까보다 iv의 값이 나아진 것을 볼 수 있음

그러나 여전히 totoal_cnt를 제외하고는 별로 유용한 변수가 없어보임
