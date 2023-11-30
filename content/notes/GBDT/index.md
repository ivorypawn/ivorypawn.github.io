---
title: "GBDT"
draft: false
tags: ["論文読み", "Machine Learning"]
---
## 概要
GBDT(Gradient Boosting Decision Tree; 勾配ブースティング決定木) は，決定木によるアンサンブル学習の一種．Kaggle で頻用されている．

GBDT は決定木の列の推定を統合（boosting）することで高精度な予測を行う．決定木の列は，既存の列のアンサンブルから生じる誤差を予測する決定木を新たに生成するということを繰り返して，逐次的に生成される．

## 資料
- [XGBoost](https://github.com/dmlc/xgboost)
- [LightGBM](https://github.com/Microsoft/LightGBM)

- [Gradient Boosting Interactive Playground](https://arogozhnikov.github.io/2016/07/05/gradient_boosting_playground.html)
  - トイデータで感じをつかむ
- [GBDTの理解に役立つサイトまとめ](https://copypaste-ds.hatenablog.com/entry/2019/09/05/184947)
- [XGBoost論文を丁寧に解説する(1)](https://qiita.com/triwave33/items/aad60f25485a4595b5c8)
- [GBDTのハイパーパラメータの意味を図で理解しつつチューニングを学ぶ](https://knknkn.hatenablog.com/entry/2021/06/29/125226)
- [XGBoostのお気持ちを一部理解する](https://qiita.com/kenmatsu4/items/226f926d87de86c28089)
- [手を動かして GBDT を理解してみる](https://techblog.nhn-techorus.com/archives/14801)

## GBDT をちゃんと理解しようと思ったきっかけ
{{<tweet user="mamas16k" id="1508001575603949568">}}
{{<tweet user="yoshimasaizaki" id="1508047490095857667">}}

## 内容
$m$ 次元特徴量データ $\mathbf{x}_i\in\mathbb{R}^m$ とそのラベル $y_i\in\mathbb{R}$ の組の集合 $\mathcal{D}=\{(\mathbf{x}_i, y_i)\}$ を訓練データセットとする．サンプル数を $n=|\mathcal{D}|$ とする．

GBDT を $\phi:\mathbb{R}^m\rightarrow \mathbb{R}$ とおき，$K$ 個の決定木からなるとする．
$$
\hat{y}\_{i}=\phi\left(\mathbf{x}\_{i}\right)=\sum\_{k=1}^{K} f\_{k}\left(\mathbf{x}\_{i}\right), \quad f\_{k} \in \mathcal{F}.
$$
$\mathcal{F}$ はとりうる弱学習器全体の集合であり（ここでは決定木），$\mathcal{F}=\left\\{f(\mathbf{x})=w_{q(\mathbf{x})}\right\\}$ と書ける．$q$ は決定木の葉の index を返す関数．

まとめると，入力 $\mathbf{x}$ から各決定木の index $q(\mathbf{x})$ が決まり，決定木ごとの重み $w$ が定まる．この総和が出力 $\hat{y}$ であり，学習時にはラベル $y$ との誤差が測られる．

### 損失関数と勾配
$\phi$ の損失関数 $\mathcal{L}(\phi)$ を以下で定義する．
$$
\mathcal{L}(\phi)=\sum\_{i} l\left(\hat{y}\_{i}, y\_{i}\right)+\sum\_{k} \Omega\left(f\_{k}\right)
$$
$l$ は微分可能な凸関数．$\Omega$ は木構造が複雑になることにペナルティを与える正則化項で，次のように定義する．
$$
\begin{aligned}
\Omega(f) &=\gamma T+\frac{1}{2} \lambda\|w\|^{2} \\\\
&=\gamma T+\frac{1}{2} \lambda\sum_{j=1}^Tw_j^2
\end{aligned}
$$
ここで $T$ は木 $f$ の葉の数．
### 勾配木ブースティング
概要で述べたように，GBDT $\phi$ の決定木は逐次的に生成される．

いま決定木 $f_1,\ldots,f_{t-1}$ が生成されており，新たに $f_t$ を生成するとする．生成ずみの決定木による出力を $\hat{y}^{(t-1)}$ とおくと，このときの損失関数 $\mathcal{L}^{(t)}$ は
$$
\mathcal{L}^{(t)}=\sum\_{i=1}^{n} l\left(y\_{i}, \hat{y}\_{i}^{(t-1)}+f\_{t}\left(\mathbf{x}\_{i}\right)\right)+\Omega\left(f\_{t}\right).
$$
ただし過去に生成した決定木のパラメータは変えないものとする．すなわち最適化の対象は $f_t$ のみである．

ここで $f_{t}(\mathbf{x}\_{i})$ を差分と見なし，関数 $l(y_i,\\:\cdot\\:)$ を $\hat{y}\_{i}^{(t-1)}$ について $2$ 次までのテイラー展開で近似することを考える．

$g_{i}:=\partial_{\hat{y}^{(t-1)}} l\left(y_{i}, \hat{y}^{(t-1)}\right),h_{i}:=\partial_{\hat{y}^{(t-1)}}^{2} l\left(y_{i}, \hat{y}^{(t-1)}\right)$ とおくと，
$$
\mathcal{L}^{(t)} \simeq \sum\_{i=1}^{n}\left[l\left(y\_{i}, \hat{y}^{(t-1)}\right)+g\_{i} f\_{t}\left(\mathbf{x}\_{i}\right)+\frac{1}{2} h\_{i} f\_{t}^{2}\left(\mathbf{x}\_{i}\right)\right]+\Omega\left(f\_{t}\right).
$$
$l(y_{i}, \hat{y}^{(t-1)})$ は関係ないので除去して $\tilde{\mathcal{L}}^{(t)}$ とおく．
$$
\begin{aligned}
\tilde{\mathcal{L}}^{(t)} &=\sum\_{i=1}^{n}\left[g\_{i} f\_{t}\left(\mathbf{x}\_{i}\right)+\frac{1}{2} h\_{i} f\_{t}^{2}\left(\mathbf{x}\_{i}\right)\right]+\Omega\left(f\_{t}\right) \\\\
&=\sum_{i=1}^{n}\left[g\_{i} f\_{t}\left(\mathbf{x}\_{i}\right)+\frac{1}{2} h\_{i} f\_{t}^{2}\left(\mathbf{x}\_{i}\right)\right]+\gamma T+\frac{1}{2} \lambda\sum\_{j=1}^Tw_j^2
\end{aligned}
$$
いま $I\_{j}:=\left\\{i \mid q\left(\mathbf{x}\_{i}\right)=j\right\\}$ とおく．つまり $I_j$ は index $j$ の葉に辿りつくような入力 $\mathbf{x}_{i}$ の添え字 $i$ 全体である．これによって総和を括ることができ，
$$
\tilde{\mathcal{L}}^{(t)}=\sum\_{j=1}^{T}\left[\left(\sum\_{i \in I\_{j}} g\_{i}\right) w\_{j}+\frac{1}{2}\left(\sum\_{i \in I\_{j}} h\_{i}+\lambda\right) w\_{j}^{2}\right]+\gamma T
$$
となる．

あとは $w_j$ で微分して極値を求めればよい．最適値 $w\_j^{\*}$ と損失 $\tilde{\mathcal{L}}^{(t)}$ は以下のとおり．
$$
w\_{j}^{*}=-\frac{\sum\_{i \in I\_{j}} g\_{i}}{\sum\_{i \in I\_{j}} h\_{i}+\lambda},\quad \tilde{\mathcal{L}}^{(t)}(q)=-\frac{1}{2} \sum\_{j=1}^{T} \frac{\left(\sum\_{i \in I\_{j}} g\_{i}\right)^{2}}{\sum\_{i \in I\_{j}} h\_{i}+\lambda}+\gamma T.
$$

#### e.g. $\mathcal{L}$ が平均二乗誤差 (MSE) のとき
損失関数 $\mathcal{L}:=\displaystyle\frac{1}{n}\sum\_{i=1}^n(y_i-\hat{y}_i)^2$ のときを考える．$g_i=2(y_i-\hat{y}_i^{(t-1)}),h_i=-2$ であるから，
$$
w\_j^{*}=\frac{\sum\_{i\in I_j}(y_i-\hat{y}_i^{(t-1)})}{|I_j|+\frac{1}{2}\lambda}.
$$

### 木構造の更新
木の深さや葉の数を制限していたとしても，木構造の数は膨大ですべて調べることは現実的でない．代えて 1 枚の葉から始めて繰り返し枝を追加していく貪欲アルゴリズムが採用される．

ある葉ノードを分割して 2 つの葉 $L,R$ を接続することを考える．分割前後の損失関数の差分を $\mathcal{L}\_\text{split}$ とおく．また $\mathcal{L}\_{L},\mathcal{L}\_{R}$ は$L,R$ それぞれに関する損失の増分である．
$$
\begin{aligned}
\mathcal{L}\_{\text {split }} &=\mathcal{L}-\left(\mathcal{L}\_{L}+\mathcal{L}\_{R}\right) \\\\
&=\frac{1}{2}\left[\frac{\left(\sum\_{i \in I\_{L}} g\_{i}\right)^{2}}{\sum\_{i \in I\_{L}} h\_{i}+\lambda}+\frac{\left(\sum\_{i \in I\_{R}} g\_{i}\right)^{2}}{\sum\_{i \in I\_{R}} h\_{i}+\lambda}-\frac{\left(\sum\_{i \in I} g\_{i}\right)^{2}}{\sum\_{i \in I} h\_{i}+\lambda}\right]-\gamma
\end{aligned}
$$

$L,R$ を貪欲に探索し，損失がもっとも小さくなるような split を続けて木構造を更新していく．
