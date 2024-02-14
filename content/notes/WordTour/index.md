---
title: "Word Tour"
draft: false
tags: ["論文読み", "NLP"]
---
単語埋め込みを健全性を保持したまま一次元へ次元削減する手法の提案．World ではない．

## 資料
1. Sato, R.. (2022). [Word Tour: One-dimensional Word Embeddings via the Traveling Salesman Problem](https://arxiv.org/abs/2205.01954). (NAACL 2022)
2. [NLP コロキウム資料](https://www.slideshare.net/joisino/word-tour-onedimensional-word-embeddings-via-the-traveling-salesman-problem-naacl-2022)
3. [最適輸送の使いかた](https://speakerdeck.com/eumesy/how-to-leverage-optimal-transport)

## 導入
自然言語の計算機可読表現をどのように実装するか考える．

"사과 (sagwa)" という語の意味を知りたいとする．"사과" のコーパスを引くと，たとえば以下の用例が見つかる．
- 사과 の⽊を植える
- 冷え冷えの 사과 ジュースがうまい
- ＊＊県は 사과 の⽣産⾼が⽇本⼀

$\\{$ 木，ジュース，＊＊県，... $\\}$ は "사과" の **共起表現** と呼ばれる．また **分布仮説** とは『ある単語の意味はその共起表現を⾒ればわかる』というものである．

分布仮説に基づいて，共起する単語を予測できるような計算機可読表現を作ることを考えたい．

単語の集合 $\mathcal{V}$ に対し，写像 $\mathcal{V}\rightarrow\mathbb{R}^d;\\:w\mapsto v$ を **単語埋め込み** といい，$v$ を **単語ベクトル** という．以下とくに混乱がなければ，単語ベクトルと単語を同一視する．

（数学用語のベクトルや埋め込みを想起させるが，空間の線形性や距離の保存などは陽には考えないことが多い）

このとき単語ベクトル $v_w$ に対してほかの単語が共起する確率 $p(\cdot|w)$ を定式化し，コーパスで共起する単語は近くに埋め込まれるように設計されるのが望ましい．

代表的な単語埋め込みの [word2vec](https://code.google.com/archive/p/word2vec/) は，3 層 NN を ~100 billion words コーパスで学習している．共起確率は以下で定式化される：

$$
p(c|w ; \theta)=\frac{e^{v_{c} \cdot v_{w}}}{\sum_{c^{\prime} \in C} e^{v_{c^{\prime}} \cdot v_{w}}}
$$

⼈間が感じる意味の類似度と word2vec による単語ベクトルペアのなす $\cos$ 類似度は⾼い相関を示した．word2vec はたとえば [gensim](https://radimrehurek.com/gensim/index.html) から利用できる．

[BERT](https://ledge.ai/bert/) はコーパスに現れる文章の一部をマスクして穴埋め問題を解かせまくる学習で，語順も考慮した周辺文脈の獲得を目指した．これが成功し NLP の広範なアプリケーションで飛躍的に性能が向上した．

- [自然言語処理におけるEmbeddingの方法一覧とサンプルコード](https://yukoishizaki.hatenablog.com/entry/2020/01/03/175156)

### 課題
分布仮説に基づく⾃然⾔語処理でうまく対処できない⾔語現象は存在するが，それとは別に実用上の課題がある．

現実に使用される単語埋め込みは典型的には $d=300$ 程度であり，メモリ・実行時間の効率面に課題がある．現状 $400,000 \text{ words}\times 300\fallingdotseq1\text{GB}$ をスマホに載せるのはつらい．

また高次元に埋め込まれた単語の解釈可能性という問題がある．t-SNE や PCA で次元削減して可視化したところで，各成分に何らかの解釈ができる望みはほぼない．これは画像処理における GAN で得られる特徴量が，ある方向への射影が何らかの解釈を齎すこととは対照的である（e.g. 肌の色，表情）[Unsupervised Discovery of Interpretable Directions in the GAN Latent Space. ICML 2020. etc.]．

単語ベクトルに摂動を加えて adversarial example や data augmentation を生成する手法も提案されているが，たとえば "$\text{cat}$" の単語ベクトルに摂動を加えた $v_\text{cat}+\epsilon$ は，単語レベルでは意味を解釈できない．これは解釈可能なデータが画像と違って単語の場合は離散的であることによる．

## 内容
以下，word tour の論文の内容について．

### 健全性，完全性
論文では，一般に単語埋め込みが満たしてほしい性質を 2 つ挙げている．
- **健全性**：単語ベクトルの距離が近ければ，単語の意味は近い
- **完全性**：単語の意味が近ければ，単語ベクトルの距離は近い

ポアンカレ埋め込みの教訓より，どちらかを諦めれば大きく精度劣化せず解釈可能性の高い次元削減が得られる可能性がある．論文の手法は健全性のみを採用する．すなわち取りこぼしはあり得るが，距離の近い 2 点の意味はつねに近いことは保証されており，単語検索・文書検索などに応用できる可能性がある．

これらの用語は恐らく NLP において一般的ではなく，論文筆者の造語で，論理学の概念に由来している．例えば古典命題論理の形式体系 $\text{LK}$ の健全性／完全性は以下のように定義される．
- 健全性：$\text{LK}$ で証明可能な式はトートロジーである
- 完全性：トートロジーは $\text{LK}$ で証明可能である

$\text{LK}$ において，これらはどちらも証明される．ただし上記の完全性は，厳密には意味論的完全性と呼ばれる概念である．有名なゲーデルの（第一）不完全性定理のステートメントにおける完全性は構文論的完全性であり，（関連はあるが）これとは別物である．

「証明可能」は「単語ベクトルの距離が近い」，「トートロジー」は「単語の意味が近い」に対応する．

### 巡回セールスマン問題
1 次元への次元削減を以下のように定式化する．

#### Input
- 単語の集合 $\mathcal{V}$
  - $n:=|\mathcal{V}|$
- 学習ずみ単語埋め込み $\\{x_v\mid v\in\mathcal{V}\\}\subset\mathbb{R}^d$
  - 実験では $d=300$ 次元 Glove

#### Output
- 健全な一次元埋め込み $\sigma:\mathcal{V}\rightarrow\{1,2,\ldots,n\}$ （全単射）
- minimize $\sum_{i=1}^{n}\left\|x_{\sigma^{-1}(i)}-x_{\sigma^{-1}(i+1)}\right\|$
  - ただし $\sigma^{-1}(n+1)=\sigma^{-1}(1)$ とする（端の単語を特別視しない）
- 座標や距離は考えず，並び順のみを考える
  - 超軽量（40,000 単語で 312 KB）
  - 解釈可能

この問題設定は巡回セールスマン問題にほかならない．NP 困難で，厳密解法は素朴な動的計画法より優れたものは知られておらず， $O(2^n n^2)$ かかる（$n\leq 20$ くらいまでなら解ける）．

- [指数時間アルゴリズム入門](https://www.slideshare.net/wata_orz/ss-12131479)

いっぽう近似解法については，100,000 点の入力に対して誤差 0.0019% の近似解が現実的な時間で得られることもある．

- [Kernighan–Lin algorithm](https://en.wikipedia.org/wiki/Kernighan%E2%80%93Lin_algorithm)
- [Simplified LKH(Lin-Kernighan Heuristic)の実装](http://shopping2.gmobb.jp/htdmnr/www08/np/tsp/tsp_slkh01.html)

論文では，40,000 単語からなる埋め込み空間の点群を上記 Lin-Kernighan Heuristic ソルバーに投げ，理論下界 236300.947 に対し総距離 236882.314 となる解を得たとのこと．

### 評価
- https://github.com/joisino/wordtour#-results

WordTour から適当な区間を切り出したものを眺めると，他手法を圧倒して解釈性が高い．とくに序数詞を教師なしで取り出せている点は面白く，Glove がそのような構造を含んでいることがわかる．

文書（単語の集合）の類似度評価では，WordTour 上で「ぼかして」BoW (Bag of Words) を適用する手法を提案している（カーネル密度推定に似ている）．通常の BoW と比べて類似度を十分考慮できるため，精度が向上する．

#### FYI: Word Mover's Distance
比較対象である WMD (Word Mover's Distance) は，埋め込み空間の点群間の最適輸送問題を解いている．

もとの WMD の定式化は NLP ドメイン知識を踏まえるとやや不自然なところがあり，これを修正したところスコアが跳ね上がったという報告がある．

- [Word Rotator's Distance](https://arxiv.org/abs/2004.15003). In Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing (EMNLP), pp.2944–2960, 2020.
  - 点群の各点に単語ベクトルのノルムに比例する重みを付与
  - 輸送コストの距離関数を L2 から cos 類似度に
    - もとの単語埋め込みの学習は原点依存

## Kaggle
位置情報のプラットフォーマーである Foursquare のデータを題材に，店舗や施設の名前・住所・座標・カテゴリ・URL・電話番号が含まれるレコードが与えられ，POI が同一のものをマッチングするという kaggle コンペが開催された．

同一 POI であっても，座標が少し違ったり，名前や住所の表記が異なっていたり，欠損があったりと，情報に揺らぎがある．

このコンペで WordTour を活用し，solution を公開した人が現れた．
- [7th place solution(with inference code)](https://www.kaggle.com/competitions/foursquare-location-matching/discussion/335800)

Embedding を対照学習で生成したのち k-means++ で 2000 のクラスタに分割し，centroid の集合に WordTour を適用することで，後段の GBDT の学習が少ない弱学習器（決定木）で効率的に進んだとしている．

残念ながらコンペ終了後に leakage 発覚し，上位モデルは過学習していただけではないかとの疑いがもたれた．

- [Leakage (67% of test rows leaked - more than anyone expected.)](https://www.kaggle.com/competitions/foursquare-location-matching/discussion/335799)

その後 Food-101 に対して WordTour を検証したノートブックが公開された．
- [word tour experiment](https://www.kaggle.com/code/nadare/word-tour-experiment/notebook)

LKH ソルバーではなく google の OR-Tools を使っており，遅いが手軽に計算できるとのこと．WordTour によって学習に必要な iteration が半減している．

WordTour と GBDT の親和性が高いのは直観的に納得できるし，面白いアイデアなので，leak-free コンペでよい結果が出てほしい．