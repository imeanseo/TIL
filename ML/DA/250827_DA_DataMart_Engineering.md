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
