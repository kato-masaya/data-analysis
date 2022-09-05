# データの事前分析



# 1. データの事前分析

データ前処理など実施する前にデータの情報量、状態などをチェックします。

ひとまず例として、Series、Dataframe両方でデータの準備を行います。

- Seriesのテストデータ作成例

```python
import numpy as np
from pandas import DataFrame,Series

# 開始値1～終了値100の値をランダムに10個持つSeriesを作成する。
series1 = Series(np.random.randint(1,100,10))

# 出力結果
print(series1)
```

- 出力結果

```python
0    62
1    46
2    30
3    23
4    80
5    45
6    27
7    34
8    43
9     3
dtype: int64
```

- Dataframeのテストデータ作成例

```python
import numpy as np
import pandas as pd 

df = pd.DataFrame(
    {
     "id": range(1,11),
     "data1" : np.random.randint(1,100,10),     
     "data2" : np.random.randint(100,500,10),     
    }
)

# 出力結果
print(df)
```

- 出力結果

```python
id  data1  data2
0   1     62    247
1   2     76    272
2   3     94    333
3   4     74    180
4   5     23    413
5   6     51    182
6   7     60    180
7   8     42    493
8   9     30    473
9  10     21    245
```

# 2. 事前分析時に利用するコマンド

事前分析時に利用するコマンド一覧です。以下利用しデータの特徴を捉えます。

```python
#最大値、最小値、平均値、標準偏差などの参照
df.describe()

#各列のデータ型の参照
df.info()

#データの先頭数行を閲覧する
df.head()

#データの末尾数行を閲覧する
df.tail()

#データの次元数（何行、何列あるか）
df.shape

# 値の個数
df["column"].value_counts()

# 値の個数（上と同じ）
pd.value_counts(df["column"])

# 列名の一覧①
df.columns

# 列名の一覧②
df.count()
```

データの特徴を捉えたら今度はデータクレンジングに移っていき、欠損値・外れ値・不均衡データなどの補正に移っていきます。

# 99. 参考

- [https://www.teijitaisya.com/python-pandas-datacleansing/#index_id4](https://www.teijitaisya.com/python-pandas-datacleansing/#index_id4)
