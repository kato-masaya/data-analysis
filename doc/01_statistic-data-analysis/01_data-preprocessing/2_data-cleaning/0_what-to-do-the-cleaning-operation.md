# クリーニング
# もくじ
- [クリーニング](#クリーニング)
- [もくじ](#もくじ)
- [1. クリーニング作業でやるべきこと](#1-クリーニング作業でやるべきこと)
- [2. 列名・表記揺らぎ・データ型のチェック](#2-列名表記揺らぎデータ型のチェック)
- [3. 重複行の削除](#3-重複行の削除)
- [4. 不要列の削除](#4-不要列の削除)
- [99. 参考](#99-参考)

# 1. クリーニング作業でやるべきこと

クリーニング作業でやるべきことは以下の通りです。

- 列名の変更
- 表記揺らぎのチェック
- 重複行のチェック
- 不要列の削除
- 欠損値のチェック
    - 別資料で触れます。
- 外れ値のチェック
    - 別資料で触れます。
- 不均衡データのチェック
    - 別資料で触れます。

別資料で掘り下げて触れるところもありますが、順を追って触れていきます。

# 2. 列名・表記揺らぎ・データ型のチェック

データの事前分析でも触れているように始めにデータの全体像を確認し、そこで列名や表記の揺らぎについても存在していないか確認する必要があります。

以下参考にデータの列や型、表記など問題ないか確認してください。

- データ確認時のコマンド（dfが対象のデータフレーム）

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

- 列名変更

```python
# 列名の変更 (辞書形式で変更する)
df = df.rename(columns={'品　　　名':'品名'})
# 列名の変更（一番最初の列を変更する）
df.rename(columns={df.columns[0]: "品名"})
```

- データの抽出・検索の例

```python
#特定の列の抽出（元データの変更はしない）
df[["colB","colD"]]

#正規表現を使って抽出する
tanka_nan = df["単価"].str.extract("(^\D*)", expand=False)
tanka_nan.value_counts()

#「[単価列の値が"バルク品"]の行抽出」且つ　「単価列の列抽出」で抽出
df.loc[df["単価"] == "バルク品","単価"]

# リストから行フィルタし、新しいDFを作る
df2 = df[df["品番"].isin(mylist)]
```

また、データ型が適切か判断するには以下のように適切なデータ型か確認するようにしてください。

- データ型チェックの例

```python
#データ型の参照
df2.info()
#データ型の変換（新しい列の追加にて実現）
df["品番2"] = df["品番"].astype(object)

#データ型の変換(元の列にそのまま代入して変換)
df_s["test"] = df_s["test"].astype(object) # object(string)
df_s["test"] = df_s["test"].astype(str) # object(string)
df_s["test"] = df_s["test"].astype(float) # Float
df_s["test"] = df_s["test"].astype(int) # integer

# 型違いの値を強制的に排除（To Nan）しつつ数値型に変換
df_s["test"] = pd.to_numeric(df_s["test"] , errors="coerce")
```

なお、数値型に変換する場合には、事前に文字列を数値にしておいたり、Nullデータを0にしておいたり、といった作業も覚えておく必要があるので型チェック時には注意してください。

# 3. 重複行の削除

重複を確認し、重複行を削除します。

- データフレームの定義と確認

```python
import pandas as pd
import numpy as np

df = pd.DataFrame([['A','a',110], ['C','c',130], ['C','c',130], ['D', 'a',140],['A','a',110]],
columns=['col01', 'col02', 'col03'],
index=['idx01', 'idx02', 'idx03','idx04','idx05'])
df
```

- pandasデータフレームのdrop_duplicatesメソッド使用例

```python
df.drop_duplicates()
```

引数に何も指定しない場合は、要素の値が全て重複する行を削除します。

そのため、重複していたidx03とidx05が削除されます。

```python
df.drop_duplicates(subset=['col02'])
```

カラムを指定して値の重複を削除するには、引数subsetにカラムを指定します。

実行します。

# 4. 不要列の削除

重複している列を例に不要列の削除例を記載します。

- データの準備

```python
import pandas as pd
df = pd.DataFrame([
    [1, 11, 1, 3],
    [2, 12, 2, 4],
    [3, 13, 3, 5]
], columns = ["a", "a", "c", "d"])
df
```

- カラム名の重複をを削除したいとき

```python
#重複してないカラム名のみ選択
df.loc[:,~df.columns.duplicated()]
```

なお、以下のようにすると最後のaカラムが採用されます。（参考：[https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.duplicated.html](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.duplicated.html)）

```python
df.loc[:,~df.columns.duplicated(keep = "last")]
```

- カラム要素の重複を削除したいとき

```python
#転置してから重複行を削除し，再度転置して元に戻す
df.T.drop_duplicates().T
```

- カラム名重複のあるデータフレームを，カラムの欠損を保管し合う＆重複削除したいとき

```python
def agg_dup_col(df_: pd.core.frame.DataFrame) -> pd.core.frame.DataFrame:
    '''カラム名重複のあるデータフレームを，カラムの欠損を保管し合う+重複削除し，データフレームとして返す
    Args:
        df_ (pd.core.frame.DataFrame): any dataframe
    Returns:
        pd.core.frame.DataFrame
    '''
    df = df_.copy()
    dup_col = set(df.columns[df.columns.duplicated()])
    for col in dup_col:
        value = [[v for v in values if v == v and v is not None]
                 for values in df[col].values.tolist()]
        value = [v[0] if v != [] else None for v in value]
        df = df.drop(col, axis=1)
        df[col] = value
    return df
```

# 99. 参考

- [Python Pandas データ前処理 個人メモ - Qiita](https://qiita.com/jooji/items/89fc03018e31f3e89519)

- [https://www.cresco.co.jp/blog/entry/18543/](https://www.cresco.co.jp/blog/entry/18543/)

- [【毎日Python】Pandasのデータフレームの重複する行を削除する方法｜drop_duplicates](https://kino-code.com/python-pandas-drop_duplicates/)

- [https://qiita.com/s2hap/items/6e8ddfa1c73363f6117e](https://qiita.com/s2hap/items/6e8ddfa1c73363f6117e)
