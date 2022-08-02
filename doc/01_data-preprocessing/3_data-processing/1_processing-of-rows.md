# 列の加工
# もくじ
- [列の加工](#列の加工)
- [もくじ](#もくじ)
- [1. 列の追加](#1-列の追加)
- [2. 列名変更と列への代入](#2-列名変更と列への代入)
  - [2.1. 列名の変更](#21-列名の変更)
  - [2.2. 列への代入](#22-列への代入)
  - [2.3. 特定の列を指定して代入](#23-特定の列を指定して代入)
  - [2.4. 正規表現で列を追加](#24-正規表現で列を追加)
- [99. 参考](#99-参考)

# 1. 列の追加

- lambdaを利用する方法

```python
dftest["grade"] = dftest["品名"].apply(lambda x : x[0:2] )
```

- map関数を利用する方法

```python
dftest = pd.DataFrame({"prefecture":["hokkaidou","shizuoka","okinawa"],"city":["sapporo","shizuoka","naha"]})
dftest_map = {"hokkaidou":10,"shizuoka":20,"okinawa":30}
dftest["maptest"] = dftest.prefecture.map(dftest_map)
dftest
```

- apply関数を利用する方法

```python
#下記の関数を作成する。
## 引数の値に“限定”が含まれていたら“限定製品”という文字列を返し、
## 引数の値に“新”が含まれていたら“新製品”という文字列を返し、
## 引数の値がそれ以外だったら“na”という文字列を返す。
def newitem(x):
  if "限定" in str(x):
    return "限定製品"
  elif "新" in str(x):
    return "新製品"
  else:
    return "na"

# 作成した関数で品名列を評価しながら、新しい列“製品カテゴリ”を作成して結果を返す。
df["製品カテゴリ"] = df["品名"].apply(newitem)
```

# 2. 列名変更と列への代入

## 2.1. 列名の変更

```python
# 列名の変更 (辞書形式で変更する)
df = df.rename(columns={'品　　　名':'品名'})
# 列名の変更（一番最初の列を変更する）
df.rename(columns={df.columns[0]: "品名"})
```

## 2.2. 列への代入

```python
df.loc[:,"単価"] = 0
```

## 2.3. 特定の列を指定して代入

```python
df.loc[df["単価"] == "支給", "単価"] = 0
```

## 2.4. 正規表現で列を追加

```python
df["grade"] = df["品名"].str.extract(
"(^[Ａ-Ｚ]{2}|^[A-Z]{2}|^/d/d?／/d+|^[０-９][０-９]?／[０-９]*)", expand=False)
```

# 99. 参考

- [Python Pandas データ前処理 個人メモ - Qiita](https://qiita.com/jooji/items/89fc03018e31f3e89519)