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

```python
binning_table.plot(metric="event_rate") # 타겟률 - event rate
```

![image.png](attachment:d1e622ea-10f1-4ac0-91ae-891937673e56:image.png)

```python
# Categorical
variable_cat = "Country"
x_cat = df_mart[variable_cat].values
y_cat = df_mart.target

optb = OptimalBinning(name=variable_cat, dtype="categorical", solver="mip",
                      cat_cutoff=0.1)
optb.fit(x_cat, y_cat)
optb.splits

binning_table = optb.binning_table
display(binning_table.build())
binning_table.plot(metric="event_rate")
```

[](data:image/png;base64,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fLlGWloabty4Ib34QxUdHR3ZxTc9PT0MHTpUmlWmsr3U607effdd6ZJqnTp1IITAyJEj0a1bN1k/GxubSiuQiIiISB3y8/MRGhoKBwcHHD58GACgUCjg4OAgPZNRGQwMDNCqVSvs3Lmz0sasqJcKiC4uLiUePpkyZYrKvkKI8ldFREREpAG8vLzg4+ODS5cuISQkBNOnT4eBgQG8vb0BAD4+PoiLi8N3330H4OmDLR06dADw9Cpf48aN0bVrV2RmZuL27dsAgJ9++gl//PEH7t69C0tLS3h6eqKwsBB79uxRz0mqUOaA2K9fv6qsg4iIiEjj7N+/H2ZmZli4cCEsLCwQFhYGJycnJCUlAQCaNWuGoqIiqb+lpaX0wg8AmDlzJmbOnIkzZ85IWapJkybYs2cPGjRogOTkZFy4cAG9evVCSkpKtZ7b85Q5IJ47d64q6yAiIiLSSOvWrcO6detUbvv3BbS7d+8+96UfAPDJJ59UWm1V5aU+YiYiIiIi9ajOZ0EYEImIiIhqgOp8FoQBkYiIiEjDVfezIAyIRERERBquup8FYUDUQL6HvlV3Ca+tDuougIiISAPwTSpEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJMOASEREREQyDIhEREREJKNRAfGdd97BkSNHEBcXByEEhg0bVqKPp6cn4uPj8eTJE5w4cQKtW7eWba9Xrx527dqFtLQ0pKamYuvWrTAwMJD16dy5M86dO4fs7Gzcu3cPM2fOrNLzIiIiIqpJNCogGhgY4OrVq/j8889Vbp81axamTZsGNzc32NraIisrC35+ftDX15f6/O9//0PHjh0xYMAADB06FH369MHmzZul7UZGRvD398fdu3dhY2ODmTNnYsGCBZg8eXKVnx8RERHVPO7u7oiNjUV2djaCgoLQo0ePUvt26NABv/32G2JjYyGEwFdffVXhMdVBowLi8ePHMW/ePBw6dEjl9unTp2PRokU4cuQIwsPDMW7cOFhaWuKDDz4AALRr1w6DBw/GpEmTEBISgr/++gtffvklPv74YzRq1AgAMHr0aOjp6WHixImIiIjAvn378PPPP2PGjBnVdJZERERUUzg7O8PLywuenp7o3r07rl69Cj8/P5iZmansX6dOHcTExGDOnDlISEiolDHVQaMC4vO0aNECjRo1QkBAgLQuPT0dwcHBsLOzAwDY2dkhNTUVoaGhUp+AgAAUFRXB1tZW6nPu3Dnk5+dLffz8/NCuXTuYmJioPLaenh6MjIykZmhoWAVnSERERNXB0NBQ9nddT0+v1L4zZszAli1bsH37dkRGRsLNzQ1PnjzBxIkTVfa/dOkSZs2ahX379iE3N7dSxlSHGhMQLSwsAABKpVK2XqlUStssLCyQlJQk215YWIhHjx7J+qga49lj/JuHhwfS09OlFh0dXfETIiIiIrWIjo6W/V338PBQ2U9XVxc2Njayi1NCCAQEBEgXp15WVYxZFXTUXUBNsHTpUnh5eUnLjRo1YkgkIiKqoaytrWUf/5Z2pc/U1BQ6OjoqLyy1a9euXMeuijGrQo0JiImJiQAAc3Nz6evi5bCwMKlPw4YNZftpa2ujfv360j6JiYkwNzeX9SlefnbcZ+Xl5SEvL09aNjIyqtjJEBERkdpkZmYiIyND3WVotBrzEXNsbCwSEhLg4OAgrTMyMoKtrS0CAwMBAIGBgahXrx66d+8u9enfvz+0tLQQHBws9enTpw90dP5/Nh4wYACioqLw+PHj6jkZIiIi0ngpKSkoKChQeWGptItK6hizKmhUQDQwMEDXrl3RtWtXAE8fTOnatSuaNm0KAFi1ahXmzp2L9957D506dcKOHTsQHx8vPfUcFRUFX19fbNmyBT169EDv3r2xdu1a7N27V7qUvHv3buTl5WHbtm3o0KEDnJ2d8dVXX8k+QiYiIiLKz89HaGio7OKUQqGAg4ODdHFKE8asChr1EfObb76JM2fOSMsrV64EAGzfvh0TJkzA8uXLYWBggM2bN8PExAQXLlyAk5OT7N6B0aNHY+3atTh58iSKiopw4MABTJs2Tdqenp6OgQMHYt26dQgNDUVKSgoWLlyILVu2VNt5EhERUc3g5eUFHx8fXLp0CSEhIZg+fToMDAzg7e0NAPDx8UFcXBy+++47AE8fQunQoQOAp7OgNG7cGF27dkVmZiZu375dpjE1gQKAUHcRNY2lpSXi4uLQuHFjxMfHV/r4Edaac5Pq66ZDdJS6SyAioipS3r/fn3/+OWbOnAkLCwuEhYVh2rRpCAkJAQCcPn0ad+7cwYQJEwAAVlZWuHPnTokxzpw5g379+pVpTE3AgFgODIivLgZEIqJXV1X//X6VaNQ9iERERESkfgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhEREREMgyIRERERCTDgEhERET0HO7u7oiNjUV2djaCgoLQo0eP5/YfMWIEIiMjkZ2djWvXrmHw4MGy7d7e3hBCyJqvr29VnsJLY0AkIiIiKoWzszO8vLzg6emJ7t274+rVq/Dz84OZmZnK/nZ2dtizZw+2bduGN954A4cOHcKhQ4fQsWNHWT9fX19YWFhI7ZNPPqmO0ykzBkQiIiKiUsyYMQNbtmzB9u3bERkZCTc3Nzx58gQTJ05U2f+rr77C8ePH8d///hdRUVH4/vvvcfnyZXzxxReyfrm5uVAqlVJ7/PhxNZxN2TEgEhER0WvF0NAQRkZGUtPT01PZT1dXFzY2NggICJDWCSEQEBAAOzs7lfvY2dnJ+gOAn59fif59+/aFUqlEVFQU1q9fj/r161fwrCoXAyIRERG9VqKjo5Geni41Dw8Plf1MTU2ho6MDpVIpW69UKmFhYaFyHwsLixf2P378OMaNGwcHBwfMnj0b9vb28PX1hZaW5sQyHXUXQERERFSdrK2tkZCQIC3n5uZW6/H37dsnfX39+nVcu3YNMTEx6Nu3L06dOlWttZRGc6IqERERUTXIzMxERkaG1PLy8lT2S0lJQUFBAczNzWXrzc3NkZiYqHKfxMTEl+oPALGxsUhOTkbr1q1f8kyqDgMiERERkQr5+fkIDQ2Fg4ODtE6hUMDBwQGBgYEq9wkMDJT1B4ABAwaU2h8AGjdujAYN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OAOAOAOAOAOAOAOA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OAOA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)

![image.png](attachment:c73433ff-336f-4dec-a0bd-37f76ac388fe:image.png)

![image.png](attachment:b10dff35-f152-47cd-b9a4-89fba2e08f3e:image.png)
