---
title: "最適輸送問題覚書"
draft: true
tags: ["Machine Learning"]
---
## 資料
基本的なところは佐藤竜馬先生の『最適輸送の理論とアルゴリズム』で勉強している．読んだ資料や活用事例，参加したオンライン勉強会へのリンクなど．

- [輸送問題を近似的に行列計算で解く（機械学習への応用つき）](https://theory-and-me.hatenablog.com/entry/2021/05/09/181435)
- [ネットワークフロー問題たちの関係を俯瞰する](https://theory-and-me.hatenablog.com/entry/2021/04/15/171702)
- [グラフからコミュニティ構造を抽出する 〜リッチフローによるグラフの時間発展〜](https://zenn.dev/lotz/articles/eebd7ea4d1fe776f1f66)
- [最適輸送の計算アルゴリズムの研究動向](https://www.slideshare.net/KentaroOhno1/ss-241141892)

### [0x セミナー第 3 回『最適輸送の情報科学における進展』](https://sites.google.com/view/uda-0x-seminar/home/0x03)
- [最適輸送の使いかた](https://speakerdeck.com/eumesy/how-to-leverage-optimal-transport)
- [最適輸送の解きかた](https://speakerdeck.com/joisino/zui-shi-shu-song-nojie-kifang)

### [第 8 回ザッピングセミナー『Kullback-Leibler ダイバージェンスの良さについて考える』](https://zappingseminar.connpass.com/event/214404/)
板書スタイルのセミナーだった．資料の二次配布はダメとのこと．

- [Relaxation of optimal transport problem via strictly convex functions](https://arxiv.org/abs/2102.07336)
- [New characterizations of log-concavity](https://arxiv.org/abs/2004.13381)

## 輸送問題の定式化
離散的な最適輸送問題は線形計画問題クラスに入る．したがって主問題と双対問題を考えることができる（変数の数と拘束条件の数がスワップする）．

線形計画問題はソルバ (CPLEX など) に投げれば解けるが $O(n^3\log n)$ かかる．また「最適輸送の答えが小さくなるような配置を見つけたい」のように，べつの最適化の中に最適輸送を組み込むときには，問題の最適化の性質を知っておく必要がある．

上記 0x セミナーで横井先生に尋ねたところ，タスクによるが，NLP で最適輸送を使うときは $n$ が $30$ くらいのことが多いため，ソルバにとりあえず投げてみることが多いとのこと．

### 主問題
- $n$ 軒の工場．工場 $i$ は $a_i\geq 0$ の在庫をもつ
- $m$ 軒の店舗．店舗 $j$ は $b_j\geq 0$ の需要がある
- 在庫の総数と需要の総数は等しい：$\displaystyle \sum_{i=1}^{n} a_{i}=\sum_{j=1}^{m} b_{j}$
- 工場 $i$ から店舗 $j$ への商品 1 つあたりの輸送コストは $C_{ij}$
- 工場 $i$ から店舗 $j$ への商品の輸送量は $P_{ij}\geq 0$

このとき，全店舗の需要を満たしつつ輸送コストの和を最小化する（カップリング）行列 $P=(P_{ij})$ を決定せよ：
$$
\begin{aligned}
\text{argmin} \sum_{i=1}^{n}\sum_{j=1}^{m} & C_{ij}P_{ij}, \quad \text{s.t.} \\: \sum_{i=1}^{n} P_{ij}=a_i, \\:\sum_{j=1}^{m} P_{ij}=b_i
\end{aligned}
$$

なおセミナーで佐藤先生に尋ねたところ，輸送コストが輸送量に対して線形とは限らない条件への拡張もいくつか考えられているとのこと．たとえば凸関数や劣モジュラ性を満たす関数など．

### 双対問題
外部の物流業者を考える．
- $a_i,b_j$ に関する条件は主問題と同様
- 業者は工場 $i$ から商品を単価 $f_i$ で引き取る
- 業者は店舗 $j$ まで商品を単価 $g_j$ で引き渡す

このとき，工場・店舗を擁する企業が自社で輸送する最適コスト以下という条件で，物流業者の利益が最大化する $f, g$ を求めよ：
$$
\begin{aligned}
\text{argmax} \sum_{i=1}^{n} a_{i} f_{i}+\sum_{j=1}^{m} b_{j} g_{j}, \quad \text{s.t.}\\: f_{i}+g_{j} \leq C_{i j}.
\end{aligned}
$$

### 諸性質
- 双対問題について，$(f_{1}, \cdots, f_{n}, g_{1}, \cdots, g_{n})$ が最適解なら，$x\in\mathbb{R}$ について $\left(f_{1}+x, \cdots, f_{n}+x, g_{1}-x, \cdots, g_{m}-x\right)$ は最適解
- 次を満たす主問題の実行可能解 $X$ と双対問題の実行可能解 $f,g$ が存在する（**強双対性**）：
$$\displaystyle\sum_{i=1}^{n} a_{i} f_{i}+\sum_{j=1}^{m} b_{j} g_{j}=\sum_{i=1}^{n} \sum_{j=1}^{m} C_{i j} P_{i j}$$
  - 強双対性によって，主問題と双対問題の最適値が一致することが示される
  - $\leq$ がつねに成り立つことは容易に示せる（**弱双対性**）．逆は若干面倒
- **定義**：主問題の実行可能解 $P$ に対して双対問題の実行可能解 $f,g$ が次を満たすとき **相補的** であるという：$P_{ij}>0$ ならば $f_i+g_j=C_{ij}$
  - 相補的であることと $P$ と $f,g$ がそれぞれの最適解であることは同値
- **定義**：$f_i+g_j=C_{ij}$ が成り立つ組 $(i,j)$ を **タイト**，$f_i+g_j<C_{ij}$ が成り立つ組 $(i,j)$ を **ルーズ** とよぶ
  - $g'_j:=-g_j$ とする．双対問題は $f$ が回収価格，$g'$ が店舗での引き取りコストとみなせる．$f,g'$ を同じ空間に埋め込んで紐で結んだとき，紐がピンと張るときがタイト，たわんでいるときがルーズであると直観的に解釈可能
    - cf. 最適輸送の解き方，双対問題の話題 3: 双対問題の物理的イメージ (p.137~)

(Under Construction)

## ネットワークシンプレックス法
## ハンガリアン法
## Sinkhorn-Knopp アルゴリズム
### なぜ凸緩和するのか
## ニューラルネットワークによる推定
### 定式化
## Wasserstein 距離
### Wasserstein GAN
## スライス法
## 活用事例
