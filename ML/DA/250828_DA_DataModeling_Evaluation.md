# 05 DA - Data Modeling, Evaluation

# 01. 모델링을 위한 데이터 사전 준비

> 모델이 이해할 수 있는 데이터의 형태로 가공
> 

```
모델은 숫자로 이루어진 형태의 Data만 인식 가능 (※ 문자형 변수 인코딩 필요)
모델링을 수행하기 위해 Feature와 예측하고자하는 값인 Y로 데이터를 나눔
학습과 예측을 위한 Train / Test set 분할
```

> 대표적인 Encoding의 방법
> 

```
(1) One-hot encoding : 범주형(Categorical) 변수의 유형 만큼 새로운 Feature를 생성하고, 1과 0으로 표현
(2) Label encoding : 범주형(Categorical) 변수의 unique한 값만큼 증가하는 숫자를 부여
```

> ① One-hot encoding
> 

```
* 범주형 데이터의 unique한 값이 증가하면 증가할 수록 변수의 차원이 늘어나는 효과
* 예측 성능의 저하
* 범주형 변수의 Unique한 수준을 축소하고 사용하기를 권장
```

> ② Label encoding
> 

```
* Unique하고 증가하는 숫자값으로 변환되어 숫자의 ordianl한 특성이 반영되어 의도하지 않은 관계성이 생김
* 숫자값을 가중치로 잘못 인식하여 값에 왜곡이 생김
* 예측 성능의 저하
* 선형회귀와 같은 ML 알고리즘에는 적용하지 않음
* Tree 계열의 알고리즘은 숫자의 ordinal한 특성을 반영하지 않으므로 진행 가능
```

[table](https://drive.google.com/uc?id=1IBuDFmgCO0tySi5TVNcUBzCOsmpTy-oA)

![image.png](attachment:1197a673-e73d-4080-b556-e997985260cc:image.png)

```python
# 최종 추출된 Mart read
import pandas as pd
df_mart = pd.read_csv('df_mart.csv')
df_mart.head()

# 학습에 필요없는 Column 제거
df_mart = df_mart.drop(['bsym', 'CustomerID', 'total_amt_grp', 'n_target', 'grp'], axis=1)
df_mart.head()
```

| total_amt | max_amt | min_amt | total_cnt | max_cnt | min_cnt | total_qty | max_qty | min_qty | Country | target |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

| 0 | 1864.27 | 1587.07 | 277.20 | 2 | 82 | 8 | 967 | 96 | 1 | ETC | 0.0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 600.72 | 600.72 | 600.72 | 1 | 37 | 37 | 754 | 100 | 3 | ETC | 1.0 |
| 2 | 679.92 | 346.10 | 333.82 | 2 | 19 | 12 | 753 | 120 | 1 | ETC | 0.0 |
| 3 | 291.34 | 291.34 | 291.34 | 1 | 11 | 11 | 60 | 16 | 2 | ETC | 1.0 |
| 4 | 791.34 | 631.96 | 159.38 | 2 | 12 | 10 | 319 | 48 | 1 | ETC | 1.0 |

```python
# 모델링을 학습하기 위한 Feature(X)와 Target(Y)데이터를 구분하는 단계
from sklearn.model_selection import train_test_split

X = df_mart.drop(['target'], axis=1)
Y = df_mart['target']
# X → 모델이 학습할 입력 값(독립 변수) target 컬럼만 제거하고 나머지 컬럼들 사용
# Y → 모델이 예측해야 하는 정답(종속 변수) 여기서는 target (0 or 1)
x_train, x_test, y_train, y_test = train_test_split(X, Y, test_size=0.3, stratify=Y, random_state=1234)
# train_test_split → 데이터를 학습/평가 용도로 나눔
# test_size=0.3 → 전체 데이터 중 30%를 테스트용, 70%를 train용으로 사용
# stratify=Y → Y(target)의 비율을 학습/테스트 데이터셋에서 동일하게 유지
# (예: target=1이 20%라면, train과 test 둘 다 20%로 분포 유지)
# random_state=1234 → 랜덤 분할을 할 때 결과를 고정하기 위한 시드값

print(x_train.shape)
print(y_train.shape)

print(x_test.shape)
print(y_test.shape)
```

```python
# 숫자형(Integer), 범주형(Categorical) 변수 분할
numerical_list=[]
categorical_list=[]

for i in df_mart.columns :
  if df_mart[i].dtypes == 'O' :
    categorical_list.append(i)
  else :
    numerical_list.append(i)
# dtype == 'O' → Object 타입 (문자열, 범주형으로 주로 사용)
# dtype == int64, float64 → 숫자형 (integer, float)

print("categorical_list :", categorical_list)
print("numerical_list :", numerical_list)
```

## One hot encoding

```python
from sklearn.preprocessing import OneHotEncoder

for col in categorical_list :
  encoder = OneHotEncoder()
  encoder.fit(x_train[[col]])
  # Use get_feature_names_out() instead of get_feature_names()
  onehot_train = pd.DataFrame(encoder.transform(x_train[[col]]).toarray(), columns = encoder.get_feature_names_out([col]), index=x_train.index)
  onehot_test = pd.DataFrame(encoder.transform(x_test[[col]]).toarray(), columns = encoder.get_feature_names_out([col]), index=x_test.index)
  # 기존 Col은 삭제
  x_train = pd.concat([x_train,onehot_train], axis = 1).drop(columns = [col])
  x_test = pd.concat([x_test,onehot_test], axis = 1).drop(columns = [col])
  
  x_train.iloc[0,:], x_test.iloc[0,:]
```

```
# train
(total_amt      205.06
 max_amt        205.06
 min_amt        205.06
 total_cnt        1.00
 max_cnt         14.00
 min_cnt         14.00
 total_qty      369.00
 max_qty         96.00
 min_qty          1.00
 Country_ETC      0.00
 Country_UK       1.00
 Name: 3162, dtype: float64,
 
 # test
 total_amt      257.48
 max_amt        257.48
 min_amt        257.48
 total_cnt        1.00
 max_cnt         15.00
 min_cnt         15.00
 total_qty      124.00
 max_qty         12.00
 min_qty          2.00
 Country_ETC      0.00
 Country_UK       1.00
 Name: 2260, dtype: float64)
 
 # 값의 스케일이 크게 벗어나지 않음
 # train / test 분리 과정에서 데이터 분포가 잘 유지됨
```

## Label encoding

```python

from sklearn.preprocessing import LabelEncoder

# Use the original x_train and x_test before one-hot encoding
x_train_le = X.copy()
x_test_le = X.copy()

# Update categorical_list to include only 'Country' for label encoding
categorical_list_le = ['Country']

for col in categorical_list_le :
  encoder = LabelEncoder()
  # Fit on the combined data to ensure all categories are seen
  encoder.fit(pd.concat([x_train_le[col], x_test_le[col]], axis=0))

  # 기존 Col 대체
  x_train_le[col] = encoder.transform(x_train_le[col])
  x_test_le[col] = encoder.transform(x_test_le[col])

print("레이블 인코딩된 학습 데이터:")
display(x_train_le.head())
print("\n레이블 인코딩된 테스트 데이터:")
display(x_test_le.head())
```

`레이블 인코딩된 학습 데이터:`

|  | total_amt | max_amt | min_amt | total_cnt | max_cnt | min_cnt | total_qty | max_qty | min_qty | Country |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1864.27 | 1587.07 | 277.20 | 2 | 82 | 8 | 967 | 96 | 1 | 0 |
| 1 | 600.72 | 600.72 | 600.72 | 1 | 37 | 37 | 754 | 100 | 3 | 0 |
| 2 | 679.92 | 346.10 | 333.82 | 2 | 19 | 12 | 753 | 120 | 1 | 0 |
| 3 | 291.34 | 291.34 | 291.34 | 1 | 11 | 11 | 60 | 16 | 2 | 0 |
| 4 | 791.34 | 631.96 | 159.38 | 2 | 12 | 10 | 319 | 48 | 1 | 0 |

`레이블 인코딩된 테스트 데이터:`

|  | total_amt | max_amt | min_amt | total_cnt | max_cnt | min_cnt | total_qty | max_qty | min_qty | Country |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1864.27 | 1587.07 | 277.20 | 2 | 82 | 8 | 967 | 96 | 1 | 0 |
| 1 | 600.72 | 600.72 | 600.72 | 1 | 37 | 37 | 754 | 100 | 3 | 0 |
| 2 | 679.92 | 346.10 | 333.82 | 2 | 19 | 12 | 753 | 120 | 1 | 0 |
| 3 | 291.34 | 291.34 | 291.34 | 1 | 11 | 11 | 60 | 16 | 2 | 0 |
| 4 | 791.34 | 631.96 | 159.38 | 2 | 12 | 10 | 319 | 48 | 1 | 0 |

```python
x_train.iloc[0,:], x_test.iloc[0,:]
```

```python
# TRAIN
(total_amt      205.06
 max_amt        205.06
 min_amt        205.06
 total_cnt        1.00
 max_cnt         14.00
 min_cnt         14.00
 total_qty      369.00
 max_qty         96.00
 min_qty          1.00
 Country_ETC      0.00
 Country_UK       1.00
 Name: 3162, dtype: float64,
 
 # TEST
 total_amt      257.48
 max_amt        257.48
 min_amt        257.48
 total_cnt        1.00
 max_cnt         15.00
 min_cnt         15.00
 total_qty      124.00
 max_qty         12.00
 min_qty          2.00
 Country_ETC      0.00
 Country_UK       1.00
 Name: 2260, dtype: float64)
```

# 02 Model Selection

> 학습할 Model들을 List-up하는 과정
> 

```
대표적인 알고리즘에 대해서는 모두 Test 하는 것이 유리
분석하고자하는 데이터의 특성이 모두 다르므로 완벽한 알고리즘은 없음
"All models are wrong, but some are useful"
최신 개발되고 복잡한 알고리즘이 가장 성능이 높은 것은 아님 Data by Data
동일한 작동원리의 알고리즘을 사용하기보다는, 다른 작동원리를 가지고 있는 다양한 알고리즘을 선정 -> 그래야 신뢰성 높음
```

> Regression
> 

```
* 선형회귀 모델 (Ridge, Lasso, Elastic Net)
* 비선형회귀 모델 (polynomial, log모형)
* Tree 계열 Regression 모델
  - bagging 앙상블 (Randomforest)
  - boosting 앙상블 (lightGBM)

```

> Classification
```
* 로지스틱 모델 (logistic regression)
* Tree 계열 Classification 모델
  - bagging 앙상블 (Randomforest)
  - boosting 앙상블 (lightGBM)
```

![image.png](attachment:cec0ebe2-1598-4498-ac1b-f237c459f4c6:image.png)
