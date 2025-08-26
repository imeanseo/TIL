# 02 DA - **Data Readiness Check & Sampling**

---

# 1. Data Info Check
```shell
* Data를 Read하고 데이터의 형태, 타입, 기간 등 전반적인 기본 정보를 파악하는 단계
* 기본 사항을 확인하며 전처리(Pre-processing 단계)도 같이 수행함

  (1) Data shape(형태) 확인
  (2) Data type 확인 -> 문자, 숫자, 시간 형태의 type?
  (3) Null값 확인 (※ 빈 값의 Data) - NA, NULL
  (4) 중복 데이터 확인 - 똑같은 데이터가 여러 개 들어가있는가?
  (5) Outlier 확인 (※ 정상적인 범주를 벗어난 Data)  - 이상치 삭제할건지 그대로 갈건지
  ex) 0-100까지의 값만 설계된 제품에서 -9999 (제품 고장) 제거 바로 해주기,
  매출 데이터의 경우 100만원 고객 구매가 1억을 구매함
	-> 실제 발생한 매출이기 때문에 고객을 빼고 분석을 해야하는 건지는 선언하기 어려움
	-> 현업 전문가와 상의를 통해 넣을지 뺄지 결정
```

```python
# Warning 메세지 제거해주기
# 경고 메시지가 코드 실행을 막는 건 아님
# 너무 많이 뜨면 코드 실행 결과를 보기 불편하기 때문

import warnings
warnings.filterwarinings('ignore')

import pandas as pd
import numpy as np

# 출력시 최대 행, 열 100개까지만 보여주기 
pd.set_option('display.max_columns', 100)
pd.set_option('display.max_rows', 100)

url = https://github.com/imeanseo/TIL/blob/main/ML/Part3_Data.csv
df = pd.read_csv(url, encoding = 'ISO-8859-1')

# data check
df.head()

# data 형태 확인
df.shape

# data type
df.info()

# Null 값 확인
print(df.isnull().sum())

# Null 값 필요없으면 드랍 필요
# Custom ID를 기준으로 Null 값 드롭
df.dropna(subset = ['CustomerID'], how = 'all', inplace = True)
df.isnull().sum()

# subset=['CustomerID'] → CustomerID 컬럼 기준으로만 결측치(NaN) 확인
# how='all' → 지정한 컬럼들이 전부 NaN일 때만 행을 삭제
# inplace=True → 새로운 데이터프레임을 반환하지 않고, df 자체를 바로 수정

# Outlier 확인, 도메인 지식 없이 이상치 그냥 지우면 안됨, 의미있는 값일 수 있음
df.describe()

# distplot 활용해 음수 데이터의 분포를 확인
import seaborn as sns
import metaplotlib.pyplot as plt
%matplotlib inline
plt.style.use(['dark_background'])
sns.displot(df['Quantity']);

# 음수 데이터값 확인 
df[df['Quantity'] < 0]

# 지식 물어보고 음수 값 제거해달라고 요청받았다고 가정하에 음수값 제거
df = df[df['Qunatity'] > 0]
df.describe()

# 중복 데이터가 몇 개 존재하는지 확인 
pd.Series(df.duplicated()).value_counts()

# 중복 데이터가 뭔지 raw data로 확인
df[df.duplicated(keep = False)].sort_values(by = list(df.columns)).head(10)
# 중복을 볼건데 데이터에서 열을 기준으로 상위 10개 정도만 가져와줘
# keep='first' (기본) → 처음 나온 행만 남기고 이후 중복은 True
# keep='last' → 마지막 것만 남기고 그 전 중복들은 True
# keep=False → 중복된 값은 전부 True

# 중복 데이터 삭제 후 중복 재확인
df = df.drop_duplicated(keep='first')
pd.Series(df.duplicated()).value_counts()
# 출력 결과가 False의 개수가 나온다는 건 중복된게 없는 개수를 의미함

# 최종 데이터 형태 확인
df.shape
```

# 2. Data Readiness Check

---

> 데이터 수준 사전 점검
>

```
* 현재 가지고 있는 데이터의 수준으로 문제를 해결할 수 있는지 점검
(1) Target label 생성
(2) Target Ratio 확인
(3) 분석 방향성 결정

```

> ① Target label 생성
>

```
* 당월 구매 고객이 다음달 재구매 시 해당 고객을 재구매 고객으로 정의 (※ Target 정의)
* ex) 2011.01월 구매 고객이 다시 2011.2월에 구매 한 고객은 미래에 재구매 고객임
```

> ② Target Ratio 확인
>

```
* 1번 단계에서 생성한 Target Ratio의 수준을 확인하고, 문제해결이 가능한 수준인지 판단
```

> ③ 분석 방향성 결정
```
* 1,2번 단계를 종합하여 분석 방향성을 결정
 ex) Target ratio가 낮다면 전체고객이 아닌, 특정 Segmentaion 그룹으로 진행 
 - 전체 연령에서 20-30대를 기준으로
 ex) +1M 구매 타겟은 +2M/+3M까지 확대하여 Target Ratio를 상승
```

## Target Label 만들기

```python
df.head()

from datetime import datetime

# ▶ 기준년월 추출 (bsym)
df['date'] = df['InvoiceDate'].str.split(' ').str[0]
# 12/1/2010 8:26 이니까 시간 빼기 위해 0번째 인덱스만 추출

df['date'] = pd.to_datetime(df['date'])
df['bsym'] = df['date'].apply(lambda x: datetime.strftime(x, '%Y-%m'))

# ▶ CustomerID는 str Type으로 변경
df['CustomerID'] = df['CustomerID'].astype(str)
df.head()

# 기준년월 추가한 원본 데이터를 복사
df_origin = df.copy()

# 데이터 적재 기간 확인
df['date'].min(), df['date'].max()
# date 기간에서 데이터가 언제부터 언제까지 쌓였는지 알 수 있음

# 기준년월 기준으로 Unique한 CustomerID 추출
# 매 월 구매한 고객을 파악하기 위함
# 타겟 레이블링 위한 중복 제거, 여러개 구매한 사람 한명으로만 남겨두기
df = df[['bsym', 'CustomerID']].drop_duplicates(keep='first')
df.head()

# csv 파일이 2010년 1월부터 시작하니까 다음 년도에도 구매한 고객 2011년 1월을 타겟으로 설정
i = '2011-01'

# 구하고자하는 기준월 데이터 추출
df_left = df[df['bsym'] == i]

# 기준년월에 + 1달
bsym_1m = (pd.to_datetime(i) + pd.DateOffset(months=1)).strftime('%Y-%m')
# Offset 시간 더해주는 함수

# 기준년월 +1m, CustomerID는 1월의 구매한 고객이 다음달에도 구매한 고객인지 찾기
df_right = pd.DataFrame(df[df['bsym'] == bsym_1m]['CustomerID'].unique())
df_right['target'] = 'Y'
df_right.columns = ['CustomerID', 'target']

# Series.unique()는 중복을 제거한 정렬하지 않고 나온 순서대로 고유값만 남김
# 2011-02에 구매한 고객ID의 목록(중복 제거)을 만드는 용도예요.
# 고객ID들에 target='Y' 라벨을 붙여줌

# 기준년월 Data에 기준년월 +1M 고객을 Join하면 Target Data 생성 완료
df_merge = pd.merge(df_left, df_right, how='left')
df_merge['target'] = df_merge['target'].fillna('N')
# 다음 달에 산 고객이면 target='Y'가 붙고, 없으면 NaN → fillna('N')로 바꿔 미구매 = 'N'.

df_merge.head()

# ▶ for문 활용 전체 Data Set 대상 Target label 생성
loop_list = list(df['bsym'].unique())
df_all = []
for i in loop_list: 
	df_left = df[df['bsym'] == i] 
	bsym_1m = (pd.to_datetime(i) + pd.DateOffset(months=1)).strftime('%Y-%m') 
	df_right = pd.DataFrame(df[df['bsym'] == bsym_1m]['CustomerID'].unique()) 
	df_right['target'] = 1 
	df_right.columns = ['CustomerID', 'target'] 
	df_merge = pd.merge(df_left, df_right, how='left') 
	df_merge['target'] = df_merge['target'].fillna(0)

  df_all.append(df_merge)
df_all = pd.concat(df_all)

df.shape, df_all.shape

df_all.head()
```

## Target Ratio 확인

```python
# 기준년월(bsym) 기준 Target ratio 확인
df_target = df_all.groupby('bsym')['target'].agg(['sum', 'count']).reset_index()
df_target.columns = ['bsym', 'Y', 'Total']
df_target['ratio'] = df_target['Y']/df_target['Total']
df_target

# groupby('bsym') → 기준년월(bsym)별로 그룹 나눔
# 각 월별로 target 컬럼에 대해 sum: Y 개수 합 (즉, 다음달에도 구매한 고객 수)
# count: 전체 고객 수
# groupby 결과는 index가 bsym이 되는데, 그걸 reset_index 일반 컬럼으로 풀어줌

# 현재 가지고 있는 데이터는 2011-12월이 마지막이기 때문에 다음달(2012-01) Target Data를 생성할 수 없음
df_target = df_target.iloc[0:12, :]
df_target

# 전체 Data Set에 Target Ratio 확인
f_target['Y'].sum()/df_target['Total'].sum()

# 12월 데이터 분석 대상 제외
df_all = df_all[df_all['bsym'] != '2011-12']

df_all['target'].sum()/len(df_all)
# 37% 정도면 go해도됨
```
