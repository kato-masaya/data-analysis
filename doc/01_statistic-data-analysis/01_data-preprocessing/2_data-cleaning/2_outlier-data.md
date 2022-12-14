# 外れ値データへの対応
# もくじ
- [外れ値データへの対応](#外れ値データへの対応)
- [もくじ](#もくじ)
- [1. 外れ値と異常値](#1-外れ値と異常値)
  - [1.1. 外れ値とは](#11-外れ値とは)
  - [1.2. 異常値](#12-異常値)
- [2. 外れ値が及ぼす影響](#2-外れ値が及ぼす影響)
  - [2.1. 外れ値が及ぼす影響（1次元）](#21-外れ値が及ぼす影響1次元)
  - [2.2. 外れ値が及ぼす影響（2次元）](#22-外れ値が及ぼす影響2次元)
- [3. 外れ値の検出方法](#3-外れ値の検出方法)
  - [3.1. 1次元のデータの場合（四分位範囲（IQR）と箱ひげ図）](#31-1次元のデータの場合四分位範囲iqrと箱ひげ図)
    - [3.1.1. 四分位範囲（IQR）](#311-四分位範囲iqr)
    - [3.1.2. 箱ひげ図](#312-箱ひげ図)
    - [3.1.3. zスコア](#313-zスコア)
    - [3.1.4. トリム平均](#314-トリム平均)
  - [3.2. 多次元データの場合](#32-多次元データの場合)
    - [3.2.1. Isolation Forest](#321-isolation-forest)
    - [3.2.2. LocalOutlierFactor（LOF）](#322-localoutlierfactorlof)
    - [3.2.3. OneClassSVM （OCSVM）](#323-oneclasssvm-ocsvm)
    - [3.2.4. EllipticEnvelope](#324-ellipticenvelope)
- [4. 外れ値に対する考え方](#4-外れ値に対する考え方)
  - [4.1. 異常値であるかを検討する](#41-異常値であるかを検討する)
- [4.2. 外れ値の除外は慎重に](#42-外れ値の除外は慎重に)
- [5. 外れ値検出の実装](#5-外れ値検出の実装)
  - [5.1. １次元データの外れ値の検出（四分位数）](#51-１次元データの外れ値の検出四分位数)
  - [5.2. 2次元データの外れ値の検出](#52-2次元データの外れ値の検出)
- [99. 参考](#99-参考)

# 1. 外れ値と異常値

## 1.1. 外れ値とは

外れ値（outlier）とは測定された値の中で他のデータとかけ離れているものを指します。

実験結果を記録している中で、他のデータの分布とは明らかに異なる場所に数値が出現したりする際に外れ値と呼ばれることが多いです。外れ値が発生する原因は様々ですが、そのままにしておくと、データ分析の際に統計指標を歪める可能性があるため、何らかの対処が必要な場合があります。

- 外れ値の例（赤枠で囲われているところが外れ値）

![Untitled](img/2_missing-data/Untitled.png)

## 1.2. 異常値

異常値は外れ値の中で入力ミスや測定ミス等が原因となっているものを指します。つまり、**物理的にとりうるはずがない値**ということです。

例えばものの数を測定するときには、通常は0もしくは正の値しかとりません。その中で負の値を持ったデータがあるなど、とりうるはずのない値が異常値として判定されます。

- 例：倉庫内の製品と在庫数の関係

![Untitled](img/2_missing-data/Untitled%201.png)

在庫数において負の値をとることはあり得ないためこちらは異常値となります。

# 2. 外れ値が及ぼす影響

データセットによって外れ値がもたらす影響は異なります。ここでは人為的に作成したデータで欠損値の影響を再現しますが、実際には使用しているデータセット毎に考えなければならないため注意が必要です。影響の例は以下のとおりです。

- 外れ値による統計指標の歪み
- 作業コストの増加
- データ分析の精度の低下 など

## 2.1. 外れ値が及ぼす影響（1次元）

まずは理解しやすい1次元のデータを示します。表Bと表Cはあるクラスの生徒の身長をまとめたものです。2つの表を見比べて何が起こっているかを確認してみます。

![Untitled](img/2_missing-data/Untitled%202.png)

表Bと表Cを見比べると、表Cには外れ値が存在し、生徒「G」の値が明らかに不自然なことがわかります。表Bを確認してみると本来の値は「152cm」であったことがわかります。つまり、何かしらの要因により身長がメートルで記載されてしまったと考えられます。ちなみに「1.52cm」の身長の生徒が存在する確率は限りなく0です。そのため、人的ミスであることが確認できればほぼ**異常値**として断定していいと言えます。

次は平均に注目します。外れ値がない場合は「165.80」ですが、外れ値がある場合は「150.75」になっています。これが**統計指標の歪み**です。

今回は生徒の人数が10人だけであるため、影響がかなり大きくで出ています。正しいデータの数がもっと多ければ、外れ値から受ける影響も少なくなりますが、それでも外れ値が表Cにとって悪影響であることは間違いありません。

## 2.2. 外れ値が及ぼす影響（2次元）

2次元の場合についても見ていきます。なお、機械学習の分野は多次元データから予測することがほとんどですが、2次元の方が視覚的に理解しやすいため解説ではそちらを使用します。

図3、図4ではある企業での20歳〜40歳までの年齢（歳）と平均年収（万円）の相関が高いと仮定したプロットに、近似直線を描画しています。

![Untitled](img/2_missing-data/Untitled%203.png)

図3と図4を見比べると、外れ値がない場合は年齢が上がるにつれて平均年収も高いことがわかります。しかし、外れ値がある場合は21歳の平均年収が1500万になっているだけで、その他の値は全て同じに設定されています。そのため、近似直線（赤線）に従うと年齢が上がるにつれて平均年収が下がると読み取ることができます。

このように2次元でも外れ値は全体のデータに影響を与える場合があります。多次元の場合も同様で外れ値に対して全体のデータが引っ張られる場合があります。

大きな影響を与えているこの21歳の平均年収ですが、この外れ値の原因を少し考えています。例としては以下のようなことが挙げられます。

- 入力を間違えたことによる人的ミス
- 本当に対象企業の21歳の平均年収が1500万である　など

このように、外れ値に対しては色々な考えを挙げることができます。外れ値の確かな原因が不明なときは仮説を立ててデータ分析を進めていく必要もあります。ただデータを全体的に眺めて流すのではなく、データが持つ意味を考えながら外れ値に対応していくことが重要です。

# 3. 外れ値の検出方法

一概に外れ値の検出方法と言っても様々な手法があります。データセットに合わせた正しい手法の選択と、正しい外れ値を検出できることが重要となるため実施時には注意が必要です。

## 3.1. 1次元のデータの場合（四分位範囲（IQR）と箱ひげ図）

### 3.1.1. 四分位範囲（IQR）

「四分位範囲（Interquartile range）」とはデータの広がり具合を示す指標の１つです。

四分位範囲は四分位数を使って求められます。四分位数は、データの値を小さい順に並べた時にデータを4つに分割する時の区切り値のことを指します。小さい順に第1四分位数（Q1）、第2四分位数（Q2）、第3四分位数（Q3）となり、第2四分位数はデータ全体の中央値を指します。第1四分位数と第3四分位数は、第2四分位数と最大値・最小値の範囲内の中央値を指します。

四分位範囲の求め方は以下の通りです。前節で用いた表Cを小さい順に整頓したものを利用して表Dとして表示します。

$$
四分位範囲（IQR) ＝ 第3四分位数（Q3) ー 第1四分位数（Q1）
$$

![Untitled](img/2_missing-data/Untitled%204.png)

四分位範囲を使って外れ値を検出できます。一般的に、四分位範囲を**1.5倍**に拡大し、そこから外れる値を外れ値とします。四分位範囲における外れ値は以下の式で表されます。四分位範囲と同じように前節で用いた表を利用して外れ値を計算し、表示します。

- 外れ値の定義と計算例

$$
外れ値 ＜ 第1四分位数 ー （1.5 × 四分位範囲）
$$

$$
外れ値 ＞ 第3四分位数 ＋ （1.5 × 四分位範囲）
$$

$$
外れ値 ＜ 151.5　　　　　（151.5 ＝ 162 ー 1.5 × 7）
$$

$$
外れ値 ＞ 179.5　　　　　（179.5 ＝ 169 ＋ 1.5 × 7）
$$

上の計算結果であれば外れ値は「G」の1.52になり、四分位範囲を使用して外れ値を検出できます。次はこの四分位範囲から求めた外れ値を可視化できる箱ひげ図についてみていきます。

### 3.1.2. 箱ひげ図

箱ひげ図（box-and-whisker plot）は外れ値を可視化する際に非常に便利な図です。四分位範数を含めたデータの要約統計量を確認できます。

![Untitled](img/2_missing-data/Untitled%205.png)

箱ひげ図はデータの四分位数と最大値、最小値を表示してくれます。最大値、最小値、外れ値の表示に関しては、箱ひげ図の設定によって異なり、上記に述べたような四分位範囲の1.5倍の範囲で表示することもあれば、外れ値を表示せずに、単にデータ全体の最大値と最小値を表示することもできます。図5では外れ値も記載しています。

四分位範囲を用いた外れ値検出は、過程となる確率分布がありません。そのため、比較的、汎用性が高いという利点があります。また、他のデータの箱ひげ図と並べた時にデータのばらつきを簡単に可視化できます。しかし、データの数が少なかったりすると外れ値の検出がうまくいかない場合もあるため、注意が必要です。

### 3.1.3. zスコア

zスコア（z-score）はそれぞれのデータの点数を平均値を0、標準偏差を1になるように変換した値（＝標準化）のことです。これにより**データ内の相対的な位置を知ること**ができます。

zスコアによる外れ値検出は有意水準によって決定されます。有意水準が-2と2であれば、外れ値でないデータは95.45%の中に含まれることになり、-3と3では99.73%の中に含まれることになります。また、有意水準の基準については下記の通りです。


|有意水準|信頼度|
|---|---|
|<-1.65 かつ +1.65<|90 %|
|<-1.96 かつ +1.96<|95 %|
|<-2 かつ +2<|95.44%|
|<-2.58 かつ +2.58<|99 %|
|<-3 かつ +3<|99.73 %|


![Untitled](img/2_missing-data/Untitled%206.png)

![Untitled](img/2_missing-data/Untitled%207.png)

生徒Gのzスコアは-2.98でした。有意水準が2であれば外れ値と判定できます。zスコアを用いた外れ値の検出には似たものとしてスミルノフ・グラブス検定も存在します。部分的には同じですが、スミルノフ・グラブス検定は有意水準の求め方が決まっており、再帰的に行うという特徴があります。

注意点として、zスコアを使用した検定は、**データ全体が正規分布に従っていること**を仮定としている点を理解しておいてください。データの形状が正規分布に従わない場合、外れ値検出がうまく行かない場合が存在します。そのため、対数変換をすることでデータの分布の形状を変化させる必要があったりもします。

### 3.1.4. トリム平均

トリム平均とはデータ全体の最大値と最小値から一定の割合だけ削除し、残ったデータで計算した平均をとったものです。例えばデータの両側5%を削除した場合は「5%トリム平均」と言います。ここでも先ほどまでと同じ表Dを用いてトリム平均についてみていきます。

![Untitled](img/2_missing-data/Untitled%208.png)

表Fは小さい順に並べたデータの両側から10%ずつを削除し、10%トリム平均を求めたものです。削除後の平均が本来の平均値（165.80）に近づきました。実際は、データ全体の20%を削除するようなことは、非常に稀です。今回は元からのデータ数が10個であることと分かりやすいことに重きを置いたことでこのような結果になっています。トリム平均は対象の値が有効かどうかの検証は行いません。

実装は非常に簡単ですが、むやみに使用すると外れ値でない値まで削除する可能性があるため、注意が必要です。

## 3.2. 多次元データの場合

次に多次元データに対する外れ値の検出方法を見ていきます。

機械学習では基本的に多次元データから予測を立てることが多いです。1次元の外れ値検出では検出できない分布の例は複数存在します。図6ではその1例を示した2次元のものを示しています。

![Untitled](img/2_missing-data/Untitled%209.png)

図6は分布の真ん中に正しい値を取り、周囲に外れ値が存在してしまっている場合の分布です。このような分布の場合、x軸、もしくはy軸単体でデータを確認しても外れ値を検出できなくなってしまいます。そのため、2次元や多次元のまま外れ値を検出する必要があります。

以降で多次元データに使用できる外れ値の検出方法に触れていきます。多次元の外れ値検出の方法は他にも多く存在しますが、ここでは[scikit-learn](https://www.codexa.net/scikit-learn-intro/)で紹介されている外れ値検出の方法を取り上げます。

### 3.2.1. Isolation Forest

Isolation Forestは[決定木](https://www.codexa.net/decision-tree-random-forest/)を用いた外れ値検出です。与えられたデータ全体から特徴をランダムに選択し、その特徴の中の最大値と最小値の範囲から分割値を選択します。外れ値が存在していた場合、その部分の決定木の深さは他のノードまでの深さに比べて短くなります。

### 3.2.2. LocalOutlierFactor（LOF）

LOFは値に対する密度を用いた外れ値検出です。データ内のそれぞれの点から近くのk個までの値に対する局所密度を求め、比較します。外れ値が存在していた場合他の点が持つ局所密度よりも、密度が低くなります。これにより、外れ値を検出することができます。k個の値に関しては、実装の段階で指定することができます。

### 3.2.3. OneClassSVM （OCSVM）

OCSVMは[SVM](https://www.codexa.net/support-vector-machine-tutorial/)の概念を利用した外れ値検出です。元々、OCSVMは外れ値に非常に影響を受けやすく、訓練用データに外れ値が潜んでいた場合、良い結果を得ることができません。正しい値を原点から遠くに、外れ値を原点付近に分布させるような特徴空間に写像させることにより外れ値を検出できます。写像の際に使用するカーネルなどは実装の際に選択できます。有効なカーネルを選択するためにもパラメータの確認は必須です。

### 3.2.4. EllipticEnvelope

EllipticEnvelopeはデータ全体が正規分布に従っていると仮定し、楕円を検出します。正規分布を仮定しているため、従わないデータセットに関しては検出がうまくいきません。それでも、1次元のデータの時にも述べたように、正規分布に近づける手段は存在するので組み合わせながら実装していくことをオススメします。

# 4. 外れ値に対する考え方

## 4.1. 異常値であるかを検討する

異常値であるかはしっかりと検討する必要があります。

明らかに単位が違っていたり、絶対に取り得るはずのない数値であれば異常値と判断する場合もあるかもしれませんが、実際のデータセットで分析者が独断で異常値と判断するのは慎重にならなければなりません。データ分析者とデータ収集者が別の場合、情報の差し違いなどにより、誤認してしまう可能性があります。

「この異常値と思われる値はなぜ発生したのか」という**理由まで確認した上で**異常値と判断することが必要です。

# 4.2. 外れ値の除外は慎重に

機械学習を始めたばかりの方の多くは[線形回帰](https://www.codexa.net/tutorial-linear-regression-for-beginner/)や[ランダムフォレスト](https://www.codexa.net/decision-tree-random-forest/)などのモデル構築から勉強していく場合が多いです。それは決して悪いことではありません。しかし、モデルの精度を上げることだけを目的とするとデータセット内の邪魔な値（外れ値含む）を極力無くし、汎用的なモデルを作成する方に考えをつぎ込みがちです。[kaggle](https://www.codexa.net/what-is-kaggle/)などのデータ分析コンペティションにおいては単純にスコアを競う場合もあるため、推奨される場合もありますが、現実問題では慎重に考える必要もあります。

時に外れ値というのは、それ自体が必要な情報である場合もあります。例えば工場などにおける異常検知などでは、エラー時の値というのが外れ値として記録されたりもします。外れ値（以上検知時の値）を排除して作成した機械学習モデルは通常の作業においては高い精度を出力できるモデルになりますが、異常検知という動作に関しては全くの無意味になる可能性があります。

![Untitled](img/2_missing-data/Untitled%2010.png)

また、上記の図のようにデータ分析者が安易に外れ値を除外することで本来の目的を達成できない可能性もあります。そのため、外れ値への対応は目的と照らし合わせた上でしっかりと検討することが重要です。

# 5. 外れ値検出の実装

例として、Google Colab上でアヤメのサンプルデータについて、変数同士の関係を見てみます。

- データの準備

```python
from sklearn.datasets import load_iris
import seaborn as sns
import pandas as pd

iris_dataset = load_iris()
df = pd.DataFrame(iris_dataset.data, columns=iris_dataset.feature_names)
sns.pairplot(df)
```

![Untitled](img/2_missing-data/Untitled%2011.png)

可視化することで、例えば丸をつけたデータが、外れ値の可能性がありそうなことを見つけることが出来ます。

## 5.1. １次元データの外れ値の検出（四分位数）

今回は四分位数の考え方を利用して、外れ値の検出を行います。

```python
import seaborn as sns
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from numpy.random import randint
%matplotlib inline

# グラフのサイズを指定
plt.figure(figsize=(20, 4))

# 正常データ、外れ値データを生成
correct_data = randint(0,100,100)
outer_data = randint(300,500,10)

# Pandas Seriesに変換
sample_data = pd.Series(np.concatenate([correct_data, outer_data]))

# 箱ひげ図を表示
plt.subplot(1, 4, 1)
sns.boxplot(data=sample_data)

# 第1四分位値
Q1 = sample_data.quantile(0.25)
# 第3四分位値
Q3 = sample_data.quantile(0.75)
# 第1四分位値 と 第3四分位値 の範囲
IQR = Q3 - Q1
# 下限値として、Q1 から 1.5 * IQRを引いたもの 
LOWER_Q = Q1 - 1.5 * IQR
# 上限値として、Q3 に 1.5 * IQRをたしたもの 
HIGHER_Q = Q3 + 1.5 * IQR

# 四分位数の観点から、外れ値を除外する
sample_data_iqr = sample_data[(LOWER_Q <= sample_data) & (sample_data <= HIGHER_Q)].dropna()

# 箱ひげ図を描画する
plt.subplot(1, 4, 2)
sns.boxplot(data=sample_data_iqr)
```

図示した結果が下記になります。

左側の生データの箱ひげ図では外れ値が含まれていますが、右側のグラフでは、四分位数から求めた境界で外れ値を除外することができていることが分かります。

![Untitled](img/2_missing-data/Untitled%2012.png)

## 5.2. 2次元データの外れ値の検出

説明変数が1次元の場合は考えやすかったですが、2次元以上となる場合は複雑になります。

今回はIsolationForestと呼ばれる手法を使って、外れ値の検出を行います。

IsolationForestのメリットしては、教師なし学習が可能(正解ラベルが不要)という点です。

なので、比較的簡単に使うことができます。

```python
from sklearn.datasets import make_regression
import numpy as np
from matplotlib import pyplot as plt
from sklearn.ensemble import IsolationForest

# 回帰ダミーデータを作成する
X, Y, coef = make_regression(
    random_state=12,
    n_samples=100,
    n_features=1,
    n_informative=1,
    noise=10.0,
    bias=-0.0,
    coef=True,
)

print("X =", X[:5])
print("Y =", Y[:5])
print("coef =", coef)

# 外れ値を追加する
X = np.concatenate([X, np.array([[2.2], [2.3], [2.4]])])
Y = np.append(Y, [2.2, 2.3, 2.4])

# X座標の値とY座標の値がばらばらなので、一つの変数にまとめる
X_train = np.concatenate([X, Y[:, np.newaxis]], 1)

# 生データすべてをプロットする
plt.figure(figsize=(20, 4))
plt.subplot(1, 4, 1)
plt.title("raw data")
plt.plot(X, Y, "bo")

# IsolationForestインスタンスを作成する
# https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html
clf = IsolationForest(
    contamination='auto', max_features=2, random_state=42
)

# 学習用データを学習させる
clf.fit(X_train)

# 検証用データを分類する
y_pred = clf.predict(X_train)

# IsolationForest は 正常=1 異常=-1 として結果を返す
# 外れ値と判定したデータを赤色でプロットする
plt.subplot(1, 4, 2)
plt.title("outlier data(auto)")
plt.scatter(
    X_train[y_pred == -1, 0],
    X_train[y_pred == -1, 1],
    c='r',
)

# 外れ値スコアを算出する
outlier_score = clf.decision_function(X_train)
# 外れ値スコアの閾値を設定する
THRETHOLD = -0.08
# 外れ値スコア以下のインデックスを取得する
predicted_outlier_index = np.where(outlier_score < THRETHOLD)

# 外れ値と判定したデータを緑色でプロットする
predicted_outlier = X_train[predicted_outlier_index]
plt.subplot(1, 4, 3)
plt.title("outlier data(manual)")
plt.scatter(
    predicted_outlier[:, 0],
    predicted_outlier[:, 1],
    c='g',
)
```

実行結果は、下記画像のようになります。

一番左の青色でプロットしたグラフは、生成したダミーデータを単純にプロットしたものになります。

真ん中の赤色でプロットしたグラフは、IsolationForestをデフォルト設定で使用し、外れ値を検出したものになります。

一番右の緑色でプロットしたグラフは、IsolationForestのdecision_function関数を使い出力した外れ値スコアを、こちらで設定した閾値で外れ値を検出しものになります。

ソースコード内のTHRETHOLDの値を変化させると、検出結果が変化することが分かると思います。

![Untitled](img/2_missing-data/Untitled%2013.png)

複数の説明変数が存在する場合、何を外れ値とするかは難しい問題かと思います。

IsolationForestを使うと、ゆるく外れ値を検出することができるので、手作業で外れ値を除くよりも、効率よく出来るなどのメリットも考えられます。

# 99. 参考

- [データに含まれる異常値を検出してみよう - Qiita](https://qiita.com/papi_tokei/items/6f77a2a250752cfa850b)

- [https://www.codexa.net/python-outlier/](https://www.codexa.net/python-outlier/)

- [機械学習における前処理3　欠損値・外れ値・不均衡データ - Qiita](https://qiita.com/ngayope330/items/7dce95abc42cfe6566bf)

- [【初心者】特徴量エンジニアリング（外れ値）について調べてみた - Qiita](https://qiita.com/zumax/items/2a07f6d3502fc9a16a44)