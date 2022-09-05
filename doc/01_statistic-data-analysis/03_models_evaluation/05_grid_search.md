# グリッドサーチ
# もくじ
- [グリッドサーチ](#グリッドサーチ)
- [もくじ](#もくじ)
- [1. グリッドサーチとは](#1-グリッドサーチとは)
- [2. 【実装例】手書き数字(0~9)のデータセットdigitsをSVMで考える](#2-実装例手書き数字09のデータセットdigitsをsvmで考える)
  - [2.1. データの準備](#21-データの準備)
  - [2.2. 最適化したいパラメータの設定](#22-最適化したいパラメータの設定)
  - [2.3. 最適化の実行](#23-最適化の実行)
  - [2.4. 結果の表示](#24-結果の表示)
- [99. 参考](#99-参考)

# 1. グリッドサーチとは

グリッドサーチとは、**学習モデルに用いられるハイパーパラメータを調整していき、モデルの汎化性能を向上させる方法を探す代表的な手法のことです。**

グリッドサーチでは、**指定したハイパーパラメータの全ての組み合わせ**に対して学習を行い、もっとも良い精度を示したパラメータを採用していきます。

この特性上、考えられるすべての組合せを調べようとすると**計算コストが膨大になる恐れがある**ため、利用時には注意が必要です。

- 実装例

```python
from sklearn.model_selection import GridSearchCV
from sklearn import svm

parameters = [
    {'C': [1, 5, 10, 50],
     'gamma': [0.001, 0.0001]}
]

svc = svm.SVC()
clf = GridSearchCV(svc, parameters)
clf.fit(X_train, y_train)

# 結果の確認
print(clf.best_params_)
```

上記では、SVMにおいて、Cとgammaをそれぞれ{1,5,10,50},{0.001,0.0001}で最適化したいとし、このようにリスト内に、keyにCとgammaを持つ辞書を入れ、グリッドサーチをしています。

なお、Scikit-learnを用いて実装しています。

# 2. 【実装例】手書き数字(0~9)のデータセットdigitsをSVMで考える

実装例として、手書き数字(0~9)のデータセットdigitsをSVMで考える際のグリッドサーチ利用方法について触れていきます。

## 2.1. データの準備

手書き数字のdigitsをインポートします。

```python
from sklearn import datasets
from sklearn.cross_validation import train_test_split
from sklearn.model_selection import GridSearchCV
from sklearn.metrics import classification_report
from sklearn.svm import SVC

digits = datasets.load_digits()
n_samples = len(digits.images) # 標本数 1797個
X = digits.images.reshape((n_samples, -1)) # 8x8の配列から64次元のベクトルに変換
y = digits.target # 正解ラベル
```

`[train_test_split](http://scikit-learn.org/stable/modules/generated/sklearn.cross_validation.train_test_split.html)`で、データセットをトレーニング用とテスト用に2分割します。

```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.5, random_state=0)
print(X.shape)
>>> (1797, 64)
print(X_train.shape)
>>> (898, 64)
print(X_test.shape)
>>> (899, 64)
```

## 2.2. 最適化したいパラメータの設定

次に、最適化したいパラメータをリストで定義します。例題のものに加えて、polyとsigmoidのkernelを追加します。

```python
tuned_parameters = [
    {'C': [1, 10, 100, 1000], 'kernel': ['linear']},
    {'C': [1, 10, 100, 1000], 'kernel': ['rbf'], 'gamma': [0.001, 0.0001]},
    {'C': [1, 10, 100, 1000], 'kernel': ['poly'], 'degree': [2, 3, 4], 'gamma': [0.001, 0.0001]},
    {'C': [1, 10, 100, 1000], 'kernel': ['sigmoid'], 'gamma': [0.001, 0.0001]}
    ]
```

## 2.3. 最適化の実行

`GridSearchCV`を使って、上で定義したパラメータを最適化します。

指定した変数は、使用するモデル、最適化したいパラメータセット、交差検定の回数、モデルの評価値の４つです。評価値は`f1`  としています。`precision`や`recall`でも問題ありません。

参考：[https://scikit-learn.org/stable/modules/model_evaluation.html#scoring-parameter](https://scikit-learn.org/stable/modules/model_evaluation.html#scoring-parameter)

```python
score = 'f1'
clf = GridSearchCV(
    SVC(), # 識別器
    tuned_parameters, # 最適化したいパラメータセット 
    cv=5, # 交差検定の回数
    scoring='%s_weighted' % score ) # モデルの評価関数の指定
```

トレーニング用データセットのみを使い、最適化を実行します。パラメータセットなどが表示されます。

```python
clf.fit(X_train, y_train) 
>>> GridSearchCV(cv=5, error_score='raise',
>>>       estimator=SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0,
>>>  decision_function_shape=None, degree=3, gamma='auto', kernel='rbf',
>>>  max_iter=-1, probability=False, random_state=None, shrinking=True,
>>>  tol=0.001, verbose=False),
>>>       fit_params={}, iid=True, n_jobs=1,
>>>       param_grid=[{'kernel': ['linear'], 'C': [1, 10, 100, 1000]}, {'kernel': ['rbf'], 'C': [1, 10, 100, 1000], 'gamma': [0.001, 0.0001]}, {'kernel': ['poly'], 'C': [1, 10, 100, 1000], 'gamma': [0.001, 0.0001], 'degree': [2, 3, 4]}, {'kernel': ['sigmoid'], 'C': [1, 10, 100, 1000], 'gamma': [0.001, 0.0001]}],
       pre_dispatch='2*n_jobs', refit=True, scoring='f1_weighted',
       verbose=0)
```

## 2.4. 結果の表示

`clf.grid_scores_`で各試行でのスコアを確認できます。

```python
clf.grid_scores_
>>> [mean: 0.97311, std: 0.00741, params: {'kernel': 'linear', 'C': 1},
>>> mean: 0.97311, std: 0.00741, params: {'kernel': 'linear', 'C': 10},
>>> mean: 0.97311, std: 0.00741, params: {'kernel': 'linear', 'C': 100}, 
>>> ...
>>>  mean: 0.96741, std: 0.00457, params: {'kernel': 'sigmoid', 'C': 1000, 'gamma': 0.0001}]
```

`clf.best_params_`で最適化したパラメータを確認できます。

```python
clf.best_params_
{'C': 10, 'gamma': 0.001, 'kernel': 'rbf'}
```

各試行のスコアをテキストで、最適化したパラメータでの精度をテーブルで表示します。

```python
print("# Tuning hyper-parameters for %s" % score)
print()
print("Best parameters set found on development set: %s" % clf.best_params_)
print()

# それぞれのパラメータでの試行結果の表示
print("Grid scores on development set:")
print()
for params, mean_score, scores in clf.grid_scores_:
    print("%0.3f (+/-%0.03f) for %r"
          % (mean_score, scores.std() * 2, params))
print()

# テストデータセットでの分類精度を表示
print("The scores are computed on the full evaluation set.")
print()
y_true, y_pred = y_test, clf.predict(X_test)
print(classification_report(y_true, y_pred))
```

以下のように表示されます。

```python
# Tuning hyper-parameters for f1

Best parameters set found on development set: {'kernel': 'rbf', 'C': 10, 'gamma': 0.001}

Grid scores on development set:

0.973 (+/-0.015) for {'kernel': 'linear', 'C': 1}
0.973 (+/-0.015) for {'kernel': 'linear', 'C': 10}
0.973 (+/-0.015) for {'kernel': 'linear', 'C': 100}

 [長いので中略]

0.967 (+/-0.009) for {'kernel': 'sigmoid', 'C': 1000, 'gamma': 0.0001}

The scores are computed on the full evaluation set.

             precision    recall  f1-score   support

          0       1.00      1.00      1.00        89
          1       0.97      1.00      0.98        90
          2       0.99      0.98      0.98        92
          3       1.00      0.99      0.99        93
          4       1.00      1.00      1.00        76
          5       0.99      0.98      0.99       108
          6       0.99      1.00      0.99        89
          7       0.99      1.00      0.99        78
          8       1.00      0.98      0.99        92
          9       0.99      0.99      0.99        92

avg / total       0.99      0.99      0.99       899
```

# 99. 参考

- [https://aiacademy.jp/texts/show/?id=542](https://aiacademy.jp/texts/show/?id=542)

[グリッドサーチ | AI Academy](https://aiacademy.jp/texts/show/?id=542)

- [https://qiita.com/WealthyFirst/items/c81f7cea72a44a7bfd3a](https://qiita.com/WealthyFirst/items/c81f7cea72a44a7bfd3a)

[Scikit learnより グリッドサーチによるパラメータ最適化 - Qiita](https://qiita.com/WealthyFirst/items/c81f7cea72a44a7bfd3a)

- [https://qiita.com/shugo256/items/52f6ad33560d7c37535c](https://qiita.com/shugo256/items/52f6ad33560d7c37535c)

[Python機械学習プログラミング 第6章 まとめ - Qiita](https://qiita.com/shugo256/items/52f6ad33560d7c37535c)