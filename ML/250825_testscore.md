# CSV  파일에서 데이터 프레임 불러오기

## CSV 파일이란?

**comma 로 열을 구분하고 줄바꿈으로 행을 구분하는 파일** 

## CSV 파일 함수

**ilepath_or_buffer #**파일의 경로명

**sep** #(인수는 문자열 / 기본값은 ',') 구분자를 지정하는 인자

**header** (인수는 정수, 정수의 리스트 / 기본값은 'infer')

columns를 지정하는 인자. 지정하지 않으면 대부분 맨 윗줄이 columns가 된다.(기본 값이 0인것과 비슷하다)

리스트로 지정하면 멀티 인덱스인 columns가 된다.

**index_col**  #(인수는 정수, 정수의 리스트 / 기본값은 None)

index를 지정하는 인자. 지정하지 않으면 RangeIndex가 index로 부여된다. 리스트로 지정하면 멀티 인덱스인 index가 된다.

## 오늘 쓸 CSV URL

[csv_link](https://raw.githubusercontent.com/panda-kim/pandas/main/ch04.csv)

## 1. CSV 연습

```python
import pandas as pd

pd.set_option("display.max_rows", 8) # 입력 시 최대 행의 개수는 8개로 지정
url = "https://raw.githubusercontent.com/panda-kim/pandas/main/ch04.csv"
df = pdf.read_csv(url) # URL 읽기
df
```

|  | 이름 | 점수 |
| --- | --- | --- |
| 0 | 노성빈 | 22 |
| 1 | 문주용 | 52 |
| 2 | 최태주 | 71 |
| 3 | 황혁범 | 53 |
| ... | ... | ... |
| 96 | 신민기 | 22 |
| 97 | 지도훈 | 24 |
| 98 | 류형석 | 41 |
| 99 | 유남길 | 67 |

100 rows × 2 columns

```python
# Data Frame
df_ex1 = pd.read_csv(url)
df_ex1
```

|  | 이름 | 점수 |
| --- | --- | --- |
| 0 | 노성빈 | 22 |
| 1 | 문주용 | 52 |
| 2 | 최태주 | 71 |
| 3 | 황혁범 | 53 |
| ... | ... | ... |
| 96 | 신민기 | 22 |
| 97 | 지도훈 | 24 |
| 98 | 류형석 | 41 |
| 99 | 유남길 | 67 |

100 rows × 2 columns

```python
# info() 함수로 데이터 정보 확인하기
df_ex1.info()  
```

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 100 entries, 0 to 99
Data columns (total 2 columns):
 #   Column  Non-Null Count  Dtype
---  ------  --------------  -----
 0   이름      100 non-null    object
 1   점수      100 non-null    int64
dtypes: int64(1), object(1)
memory usage: 1.7+ KB
```

```python
# describe() 함수로 데이터 확인하기
def_ex1.describe()
```

|  | 점수 |
| --- | --- |
| count | 100.000000 |
| mean | 50.890000 |
| std | 18.143588 |
| min | 20.000000 |
| 25% | 35.750000 |
| 50% | 50.500000 |
| 75% | 65.500000 |
| max | 79.000000 |

## 2. Feature Engineering

기존 열을 가공해 더 유용한 특성을 가지는 열을 만드는 것

## 3. 등수 매기기 (rank)

데이터의 순위나 시리즈의 순위를 매기는 함수 

<img src=https://i.ibb.co/s5g5fxw/04-03.png, width=600>

### Method

```markdown
- average: 평균 순위
- min: 최소 순위
- max: 최대 순위
- first: 출현 순서에 따라 순위 부여
- dense: ‘min’ 과 같지만 동점자가 여러명 있어도  다음 순위가 1을 더해서  부여한다

 ex. 90, 89, 89, 88을 각각 1, 2, 2, 3을 부여한다 (4위가 아니라 3위를 부여한다)

**ascending** (인수는 bool / 기본 값은 True)

오름차순과 내림차순을 결정하는 인자, 기본값은 오름차순 (True)
```

```python
# 실습용 코드
import pandas as pd
data = [['송중기', 91, 82, 17], ['권보아', 82, 95, 17],
['김나현', 71, 95, 18], ['박효신', 90, 72, 18],
['김선미', 80, 72, 19], ['강승주', 78, 95, 19]]
df = pd.DataFrame(data, columns=['이름', '영어', '국어', '나이'])
df1 = df.copy() # df와 똑같은 데이터프레임을 복사해 df1으로 선언
df
```

| 이름 | 영어 | 국어 | 나이 |
| --- | --- | --- | --- |
| 0 | 송중기 | 91 | 82 |
| 1 | 권보아 | 82 | 95 |
| 2 | 김나현 | 71 | 95 |
| 3 | 박효신 | 90 | 72 |
| 4 | 김선미 | 80 | 72 |
| 5 | 강승주 | 78 | 95 |

```python
# 영어 오름이라는 열을 만들고 영어 점수의 등수를 매김 - 오름차순으로
# 여기서 오름차순은 점수가 제일 낮은 사람이 1을 줌

df1['영어오름'] = df1['영어'].rank()
df1
```

|  | 이름 | 영어 | 국어 | 나이 | 영어오름 |
| --- | --- | --- | --- | --- | --- |
| 0 | 송중기 | 91 | 82 | 17 | 6.0 |
| 1 | 권보아 | 82 | 95 | 17 | 4.0 |
| 2 | 김나현 | 71 | 95 | 18 | 1.0 |
| 3 | 박효신 | 90 | 72 | 18 | 5.0 |
| 4 | 김선미 | 80 | 72 | 19 | 3.0 |
| 5 | 강승주 | 78 | 95 | 19 | 2.0 |

```python
# 이렇게 매기면 어색하기 때문에 내림차순 정렬을 하는게 보기 좋음
# 따라서 asceding 함수를 사용해 내림차순을 이용함

df1['영어내림'] = df1['영어'].rank(ascending=False)
df1
```

|  | 이름 | 영어 | 국어 | 나이 | 영어오름 | 영어내림 |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 송중기 | 91 | 82 | 17 | 6.0 | 1.0 |
| 1 | 권보아 | 82 | 95 | 17 | 4.0 | 3.0 |
| 2 | 김나현 | 71 | 95 | 18 | 1.0 | 6.0 |
| 3 | 박효신 | 90 | 72 | 18 | 5.0 | 2.0 |
| 4 | 김선미 | 80 | 72 | 19 | 3.0 | 4.0 |
| 5 | 강승주 | 78 | 95 | 19 | 2.0 | 5.0 |

```python
# 국어열로 오름차순으로 등수매길때 동점자를 다르게
df1['average'] = df1['국어'].rank()
df1['min'] = df1['국어'].rank(method='min')
df1['max'] = df1['국어'].rank(method='max')
df1['first'] = df1['국어'].rank(method='first') 
# 위에서부터 차례대로 순서부여
df1['dense'] = df1['국어'].rank(method='dense') 
# min과 유사, 가장 밀집된 순위를 부여df1
```

## 4. 자료형 바꾸기 (astype)

```markdown
**dtyp**dtype**

자료형을 지정하는 인자. dtype을 입력

특정 열에만 적용하고 싶을 경우 열의 레이블과 자료형을 딕셔너리로 넣기

ex> A열은 문자열로 B열은 정수로 바꾸고 싶을 경우 dtype인자에  {'A':'str', 'B':'int'}을 인수로 입력
```

```python
import pandas as pd
data = [[8.2, 9, '17'], [7.1, 9, '18'], [9.3, 7, '18'], [7.8, 7, '19']]
df = pd.DataFrame(data, colums = ['A'.'B','C']
df1 = df.copy() #그대로 복사
df
```

|  | A | B | C |
| --- | --- | --- | --- |
| 0 | 8.2 | 9 | 17 |
| 1 | 7.1 | 9 | 18 |
| 2 | 9.3 | 7 | 18 |
| 3 | 7.8 | 7 | 19 |

```python
# 자료형 파악하기
df1.info()
```

```python
# 데이터 자료형을 문자열로 바꾸기
df1.astype('str').info()
```

```python
# 특정 열의 자료형 바꾸기

df1['C'] = df1['C'].astype('int')
df1.info()

df.astype({'C':'int'}).info()

df.astype({'A': 'str', 'C':'int'}).info()
```

## 5. 데이터프레임 필터링 - 불리언 인덱싱 사용하기

bool 배열로 인덱싱하여 리스트 뿐만 아니라 시리즈도 만들 수 있어 판다스에서 필터링 할 때 유용하게 쓰인다

```python
# 조건문 (시리즈) - 명령어가 이니기 때문에 변수 지정 가능
df['국어'] > 80

# 변수 사용하기 
cond1 = df['국어'] > 80

# 실습 준비 코드
import pandas as pd

data = [[85, 96, 94], [79, 87, 94], [93, 85, 73], [81, 84, 88]]
df = pd.DataFrame(data, index=list('ABCD'), columns=['국어', '영어', '수학'])
df

# 행을 불리언 인덱싱 (대괄호 인덱싱이나 loc 인덱싱으로 가능)
df.loc[[True, False, True, True]]
df[[True, False, True, True]]

# 이렇게 되면 행을 기준으로 B의 행이 지워지고 나머지 A,C,D행만 출력됨

# 열을 불리언 인덱싱 
df.loc[:, [True, False, True]]
# 열을 기준으로 2번째 열인 영어를 제외한 국어 수학 열을 출력

# 국어점수가 80점보다 큰 사람만 필터링
df[df['국어'] > 80]

# 평균이 85점 이상인 사람만 필터링
df[df.mean(axis=1) >= 85]

#axis = 0이 열 axis = 1이 행

# 2차원 배열로 불리언 인덱싱을 할때
df[df > 80]
```

```python
# 프로젝트에서 필터링하기

import pandas as pd
pd.set_option('display.max_rows', 6)
url = "https://raw.githubusercontent.com/panda-kim/pandas/main/ch04.csv"

# 등수 매겨주기
df_ex1 = pd.read_csv(url)
df_ex1['등수'] = df_ex1['점수'].rank(method='min', ascending = False).astype('int')
df_ex1

# 1등(공동1등 포함)의 명단 추출하기
df_ex1.loc[df_ex1['등수'] == 1, '이름']

# 40점 이하는 낙제이다. 낙제가 아닌 성적만 필터링
df_ex1[df_ex1['점수'] > 40] 
```

## 6. 조건에 따라 값 부여하기 - 불리언 마스킹

불리언 마스킹: if 조건문처럼 참, 거짓에 따라 값을 부여한다(마스킹한다)

```python
# pandas mask - .mask()

> pandas mask

<img src=https://i.ibb.co/vP4MstR/04-06.png, width=600>

True or False(bool)에 따라 값을 씌우는(masking) 함수

즉 불리언 마스킹을 하는 함수 (boolean masking)

**cond** (인수는 bool 시리즈, 데이터프레임)

조건문처럼 작동하는 True 또는 False의 배열을  입력받는 인자  mask 함수는 True일 때의 값을  바꾼다.

조건문의 기능을 하는 것이지 실제로는 명령어가 아니고 bool 시리즈나 데이터프레임인 배열
(그래서 변수로 지정 가능)

**other** (인수는 스칼라 , 시리즈, 데이터프레임, 함수 / 기본값은 nan)

조건문이 True일 때 바꿀 값을 지정하는 인자. 기본값은 NaN

```

```python
# 실습 준비 코드
import pandas as pd
list1 = [85, 79, 93, 81]
data = [[85, 96, 94], [79, 87, 94], [93, 85, 73], [81, 84, 88]]
df = pd.DataFrame(data, index=list('ABCD'), columns=['국어', '영어', '수학'])
df

# masking
result = []

for i in list1:
	if i > 80:
		result.append("합격")
	else:
		result.append("불합격")
print(list1)
print(result)

# 리스트 내포(List Comprehension)을 사용하면 한줄 코드로 가능
['합격' if i > 80 else '불합격' for i in list1]

# 80보다 큰 데이터 합격으로 바꾸기
cond1 = df > 90
df.mask(cond1, '합격')

# 80보다 큰 데이터는 합격 80보다 작은 데이터는 불합격
df.mask(cond1, '합격').mask(~cond1, '불합격')

# 시리즈에 mask 함수 적용해 국어열 수정하기
df1 = df.copy()
cond2 = df1['국어'] <= 80
df1['국어'] = df1['국어'].mask(cond2, '불합격')
df1

# 다시 df와 같은 원본으로 실습해 80이하는 불합격 80이상은 합격으로
df1 = df.copy()
df1['국어'] = df1['국어'].mask(cond2, '불합격').mask(~cond2, '합격')
df1

# 다시 df와 같은 원본으로 실습하고 위 결과를 별도의 열로 만들기
df1 = df.copy()
df1['국어성적'] = df1['국어'].mask(cond2, '불합격').mask(~cond2, '합격')
df1

# numpy 함수 np.where으로도 가능하다
import numpy as np
df1 = df.copy()
df1['국어성적'] = np.where(cond2, '불합격','합격')
df1
```

```python
# 프로젝트 코드
import pandas as pd
pd.set_option('display.max_rows', 6) # 6행까지만 출력하는 코드
url = 'https://raw.githubusercontent.com/panda-kim/pandas/main/ch04.csv'
df_ex1 = pd.read_csv(url)
df_ex1['등수'] = df_ex1['점수'].rank(method='min', ascending=False).astype('int')
df_ex1

cond1 = df_ex1['점수'] > 40
df_ex1['비고'] = df_ex1['점수'].mask(cond1, '패스').mask(~cond1, '낙제')
df_ex1

import numpu as np
df1['비고'] = np.where(cond1, '패스', '낙제')
```

## 7. 범주화해서 학점 매기기 - cut

학점과 같이 구간을 나눠서 특정 값을 부여하는 것은 [merge_asof](https://kimpanda.tistory.com/70) 함수나 [mask](https://kimpanda.tistory.com/82) 함수로 가능하지만 둘다 코드가 복잡해진다

그럴 때 cut 함수를 사용하면 손쉽게 그룹을 나눠 범주화해서 학점을 매길 수 있다

```python
> pandas cut

<img src=https://i.ibb.co/fv6BWLp/04-07-1.png, width=600>

숫자와 같은 데이터를 구간별로 나눠서 범주화(categorization)하는 함수

**x** (인수는 배열)

구간을 나눌 배열을 입력받는 인자. 반드시 1차원이어야 한다

**bins** (인수는 정수, 순서의 배열)

구간을 나누는 기준을 입력받는 인자

- 정수 : 정수만큼의 균등한 구간으로 분할한다
- 순서의 배열  : ex1) [0, 20, 40, 60] 이라면 0 ~ 20, 20 ~ 40, 40 ~ 60 까지의 3개의 구간으로 분할한다

**right** (인수는 bool / 기본값은 True)

구간에서 우측 경계를 포함할지 여부를 결정하는 인자

- ex1의 경우에 True라면 우측 경계를 해당 구간에 포함하므로 0초과 20이하, 20초과 40이하, 40초과 60이하로 분할한다

**labels** (인수는 배열 또는  False / 기본값은 None)

구간의 레이블을 지정하는 인자.  False는 가장 왼쪽 구간부터 0, 1, 2, 3... 으로 레이블을 부여한다

- 기본값은 구간의 경계를 구간의 레이블로 부여한다

- ex1의 경우라면 (0, 20], (20, 40], (40. 60] 으로 부여한다

- 반드시 bins 인자로 나누어진 구간수와 같아야 한다
```

```python
# 학점 열 만들기

df_ex1['학점'] = pd.cut(df_ex1['점수'], bins=[0, 40, 50, 60, 70, 80], 
labels=['F', 'D', 'C', 'B', 'A'])

df_ex1
```

## 8. Category 자료형

범주화해서 분류하기 위함

자료형이 카테고리일 필요까지는 없고 문자열로도 분류 가능

카테고리형을 사용하면 메모리 절약, 순서 부여를 통한 원하는 순서로 정렬 가능

```python
# 수업 준비코드
import pandas as pd
pd.set_option('display.max_rows', 6)
s = pd.Series(['S', 'M', 'XL']*10000000)
s

# category 자료형은 메모리를 크게 절약해준다
print(s.memory_usage())
print(s.astype('category').memory_usage())
```

```
240000128
30000260
```

```python
# Categorical 함수로 category 자료형으로 바꾸면 순서를 부여할 수 있다
s1 = pd.Series(pd.Categorical(s, categories=['S', 'M', 'XL'], ordered=True))
s1

# s1을 S와 M, XL 순서로 정렬가능(알파벳 순서로는 오름, 내림차순 모두 불가능)
s1.sort_values()
```

## 9. 빈도수 파악하기 value_counts 시각화 plot

1. 시리즈일 때

```
s.value_counts(normalize=False, sort=True, ascending=False)
```

**normalize** (인수는 bool / 기본값은 False)

True일때는 표준화해서 비율로 보여준다

**sort** (인수는 bool / 기본값은 True)

빈도에 따라 정렬한다

**ascending** (인수는 bool / 기본값은 False)

정렬 방식을 결정하는 인자, 기본값은 내림차순

1. 데이터 프레임일 때

```
df.value_counts(subset=None, normalize=False, sort=True, ascending=False)
```

**subset** (인수는 레이블 / 기본값은 None) - 열을 지정하는 것

고유값의 개수를 파악할 열을 지정하는 인자

그외의 인자는 시리즈에 적용할 때와 동일하다

```shell
프로젝트 코드

import pandas as pd
pd.set_option('display.max_rows', 6) # 6행까지만 출력하는 코드
url = 'https://raw.githubusercontent.com/panda-kim/pandas/main/ch04.csv'

# csv 파일에서 데이터 프레임 부르기
df_ex1 = pd.read_csv(url)

# 등수 매기기
df_ex1['등수'] = df_ex1['점수'].rank(method='min', ascending=False).astype('int')

# 낙제(40점이하)와 패스를 구분한 비고 열 만들기
cond1 = df_ex1['점수'] > 40
df_ex1['비고'] = df_ex1['점수'].mask(cond1, '패스').mask(~cond1, '낙제')

# 학점 열 만들기
bin = [0, 40, 50, 60, 70, 80]
label = ['F', 'D', 'C', 'B', 'A']
df_ex1['학점'] = pd.cut(df_ex1['점수'], bins=bin, labels=label)

# 각 학점당 인원수 파악하고 그래프로 그리기
df_ex1['학점'].value_counts().sort_index(ascending=False).plot(kind='bar')
```
