# 正規化と標準化
# もくじ
- [正規化と標準化](#正規化と標準化)
- [もくじ](#もくじ)
- [1. 正規化（Rescaling）](#1-正規化rescaling)
  - [1.1. 定義](#11-定義)
  - [1.2. 特徴](#12-特徴)
- [2. 標準化（Centering）](#2-標準化centering)
  - [2.1. 定義](#21-定義)
  - [2.2. 特徴](#22-特徴)
- [3. 規格化（Standardization）](#3-規格化standardization)
- [99. 参考](#99-参考)

# 1. 正規化（Rescaling）

## 1.1. 定義

各特徴量の値の幅を**最大値が1、最小値が0**のデータとなるように揃えます。

こちらは主に**共通のスケールで公平に特徴量を比較**できるようにするために行われます。

ニューラルネットからKNNまで、幅広いアルゴリズムに有効です。

具体的な計算処理としては、$X_{max},X_{min}$をそれぞれ特徴$X$の最大値、最小値、$\sigma$を各特徴量の標準偏差とした以下があります。

$$
x^{i}_{new}=\frac{x^{i}-x_{min}}{x_{max}-x_{min}}
$$

```python
import numpy as np
def min_max(x, axis=None):
    x_min = x.min(axis=axis, keepdims=True)
    x_max = x.max(axis=axis, keepdims=True)
    result = (x-x_min)/(x_max-x_min)
    return result

a = np.random.randint(10, size=(2,5))
print(a)
print(zscore)

# sklearnを使った場合
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler([0,1]) # 0~1の正規化
# 引数にdataという変数を入れた場合です
# X_MinMaxScaler = scaler.fit_transform(data)
```

## 1.2. 特徴

正規化では主に0、1の範囲内にスケーリングしますが、**最大値及び最小値が決まっている場合などに利用します。**

画像処理などでは学習コストを下げる事ができるため、良く用いられます。

# 2. 標準化（Centering）

## 2.1. 定義

標準化は元のデータの平均を0、標準偏差が1のものへと変換する正規化法のことを指します。$\mu$を平均、$\sigma$を標準偏差とします。

$$
x^{i}_{new}=\frac{x^{i}-{\mu}}{\sigma}
$$

- 実装例

```python
import numpy as np
def zscore(x, axis = None):
    x_mean = x.mean(axis=axis, keepdims=True)
    x_std  = np.std(x, axis=axis, keepdims=True)
    z_score = (x-x_mean)/x_std
    return z_score

# ランダムなNumPyアレイを作成。（※毎回値が変わります）
a = np.random.randint(10, size=(2,5))
print(a) # 実行結果は毎回異なります。

b = zscore(a)
print(b)

# 出力結果サンプル 
"""
array([[-1.41421356,  0.        ,  1.41421356,  0.35355339,  1.41421356],
       [-1.41421356,  0.35355339, -1.06066017,  0.70710678, -0.35355339]])
"""

# sklearnを使った場合
from sklearn.preprocessing import StandardScaler
# 標準化
sc = StandardScaler()

# 標準化させる値（今回は変数aが標準化する値ですが、x_trainやx_testなどの説明変数を渡します。）
a = np.random.randint(10, size=(2,5))
X_std = sc.fit_transform(a)
print("平均", X_std.mean())
print("標準偏差", X_std.std())
```

## 2.2. 特徴

**一般的に標準化を用いる場合は、最大値及び最小値が決まっていない場合や外れ値が存在する場合に利用**します。

基本的には正規化ではなく、標準化を利用する事が多いです。正規化の場合は、外れ値が大きく影響してしまうためです。

標準化が必要なシーンとして、例えば、年齢と年収の数値があった場合に標準化をしないと、どちらかの変数を重視してしまうモデルができてしまいます。
（誤差の重みでモデルを更新した際に、年収ばかりが重要視されるモデルになってしまうでしょう。）
そこで、標準化することで、特徴量がもつ値の重みを平等にすることが出来ます。
一般的には、機械学習を用いた予測モデルを作る上で、標準化は特徴量に対して適用しておいたほうが良いとされています。

ただし、機械学習を用いた予測モデルを作る上で、**必ずしも**標準化の必要はありません。

例えば、決定木系のアルゴリズムですとスケーリングは不要です。これは決定木のアルゴリズムが各特徴量の大小関係しか見ないためです。（決定木が特徴量の数値の大小関係(順位)だけを見ていて、数値にどの程度の差があるかを見ていないため）
つまり決定木などではスケーリング自体不要になります。精度が変わることもありません。

# 3. 規格化（Standardization）

正規化と標準化を組み合わせたものです。

# 99. 参考

- [https://aiacademy.jp/texts/show/?id=555](https://aiacademy.jp/texts/show/?id=555)