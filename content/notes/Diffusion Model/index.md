---
title: "Diffusion Model"
draft: false
tags: ["論文読み", "Machine Learning"]
math: true
---

![](00.png)
- 画像に逐次的にノイズを付加しランダムな画像を生成するプロセスを考える
- 上記の逆過程を推定することで、ランダムな画像から何らかのそれっぽい画像を生成する

（FYI: DALL-E や Imagen など最近話題の画像生成はだいたい拡散モデルベース）

### （脳のモデル？）
> 推測の域を出ない話だが、現時点では拡散モデルが脳内の学習/推論の中心的な仕組みの最有力候補と思っている。学習が観測をデノイズするだけで容易で局所的な更新で実現可能。推論時に複数候補を生成でき（エネルギーモデル同様）条件付を後付けでき複数の推論の統合もでき、局所解にはまらない
> https://twitter.com/hillbig/status/1557687263160836096

## 資料
1. Ho, J., Jain, A., & Abbeel, P.. (2020). [Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239).
2. Weng, L. (2021). [What are Diffusion Models?](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/). lilianweng.github.io.
3. 第 10 回全日本コンピュータビジョン勉強会「生成モデル論文縛り読み会」[A Conditional Point Diffusion-Refinement Paradigm for 3D Point Cloud Completion](https://speakerdeck.com/takmin/a-conditional-point-diffusion-refinement-paradigm-for-3d-point-cloud-completion)
4. [最近話題の"Diffusion Model（拡散モデル）"について、簡潔にまとめてみた](https://nakajimeee.hatenablog.com/entry/2022/01/03/041646)
5. Shitong Luo, & Wei Hu (2021). [Score-Based Point Cloud Denoising](https://arxiv.org/abs/2107.10981). In 2021 IEEE/CVF International Conference on Computer Vision (ICCV). IEEE.
6. [Awesome-Diffusion-Models](https://github.com/heejkoo/Awesome-Diffusion-Models)（拡散モデルの応用研究をまとめたレポジトリ）
7. Luo, C.. (2022). [Understanding Diffusion Models: A Unified Perspective](https://arxiv.org/abs/2208.11970).
8. 須山敦志 (2019)『ベイズ深層学習』講談社

## 基礎
### 表記
- ステップ $t$ における確率変数：$\mathbf{x}_t\in\mathbb{R}^D$
  - $t=0,1,\ldots ,T$
  - たとえば画像なら $D=\text{Height}\times\text{Width}\times\text{RGB}$ など
- $\mathbf{x}_{0:T}:=\mathbf{x}_0,\mathbf{x}_1,\ldots,\mathbf{x}_T$

変数 $\mathbf{x},\mathbf{y}$ の同時分布 (joint distribution) $p(\mathbf{x},\mathbf{y})$ に対する以下の操作を周辺化(marginalization) と呼び、得られる $p(\mathbf{y})$ を $\mathbf{y}$ の ''周辺分布''(marginal distribution) と呼ぶ。
$$
p(\mathbf{y})=\int p(\mathbf{x},\mathbf{y})d\mathbf{x}
$$

また同時分布 $p(\mathbf{x},\mathbf{y})$ に対して $\mathbf{y}$ に特定の値を定めたときの $\mathbf{x}$ の確率分布を ''条件つき分布''(conditional distribution) と呼び、以下で定義する：
$$
p(\mathbf{x} | \mathbf{y}):=\frac{p(\mathbf{x}, \mathbf{y})}{p(\mathbf{y})}
$$

周辺分布と条件つき分布は確率分布の条件を満たす。

ベイズ推論による統計解析では、$\mathbf{X}$ を観測データ集合、$\mathbf{Z}$ をパラメータや潜在変数をまとめた集合とするとき、まず確率モデル $p(\mathbf{X},\mathbf{Z})$ を何らかの仮定の下に設計し、学習や予測を事後分布 $p(\mathbf{Z}|\mathbf{X})$ の計算によって実現する。

ニューラルネットワークなど実用的なモデルの多くは $p(\mathbf{Z}|\mathbf{X})$ を解析的に求められない。したがって問題を後述の情報量の最適化にシフトし、勾配などを用いて近似的に求める手法がしばしば使われる（e.g. 変分推論）。

### 拡散過程
概要で述べたノイズを付加／除去する過程は以下のように定式化される。

- 画像の ''真の'' 確率密度関数：$q(\mathbf{x}_0)$
- 拡散モデルによって推定される確率密度関数：$p_\theta(\mathbf{x}_0)$
  - $\theta$ は拡散モデルのパラメータ
- ノイズを付加していく過程：*Forward diffusion process* $q( \mathbf{x}_t | \mathbf{x}\_{t-1} )$
- ノイズを除去していく過程：*Reverse diffusion process* $p_\theta(\mathbf{x}_{t-1} | \mathbf{x}_t)$

### 情報量
ある確率分布 $p(\mathbf{x})$ に対し、関数 $f(\mathbf{x})$ の期待値を以下で定義する。
$$
\mathbb{E}_{p(\mathbf{x})}[f(\mathbf{x})]:=\int f(\mathbf{x})p(\mathbf{x})d\mathbf{x}
$$

確率分布 $p(\mathbf{x}), q(\mathbf{x})$ に対し、KL ダイバージェンス(Kullback-Leibler divergence) を以下で定義する。
$$
\begin{align\*}
D_\text{KL}[q(\mathbf{x})||p(\mathbf{x})] &:= -\int q(\mathbf{x})\log\frac{p(\mathbf{x})}{q(\mathbf{x})}d\mathbf{x} \\\\\\
&= \mathbb{E}_{q(\mathbf{x})}[\log q(\mathbf{x})] - \mathbb{E}\_{q(\mathbf{x})}[\log p(\mathbf{x})]
\end{align\*}
$$
KL ダイバージェンスは非負で、$p\equiv q$ のときに限り $0$ になる。上界は存在しない。KL ダイバージェンスは $p,q$ の "近さ" のような概念だが、距離の公理などは満たさない。

#### （参考）
- [KL-divergence as an objective function](https://timvieira.github.io/blog/post/2014/10/06/kl-divergence-as-an-objective-function/)

### 変分推論
事後分布を求めるために周辺尤度 $p(\mathbf{X})=\int p(\mathbf{X},\mathbf{Z})d\mathbf{Z}$ を計算したいのだが、先述のとおり、モデルが複雑なときはこの積分を解析的に実行できない。

変分推論法では、''ELBO''(Evidence Lower Bound; エビデンス下界) と呼ばれる対数周辺尤度 $\log p(\mathbf{X})$ の下界 $\mathcal{L}(\xi)$ を導入し、変分パラメータ $\xi$（しばしば近似分布の平均や分散を表す）を最適化して ELBO を最大化することによって（対数）周辺尤度の近似解を得る。
$$
\log p(\mathbf{X})\geq\mathcal{L}(\xi)
$$

よく使われる手法の一つは、事後分布 $p(\mathbf{Z}|\mathbf{X})$ のあるパラメータ $\xi$ によって定まる分布 $q(\mathbf{Z};\xi)$ による近似であり、次の KL ダイバージェンスの最小化問題を解く。
$$
\arg\min_\xi D_\text{KL}[q(\mathbf{Z};\xi)||p(\mathbf{Z}|\mathbf{X})]
$$
また対数周辺尤度は以下のように分解される。
$$
\log p(\mathbf{X})=\mathcal{L}(\xi)+D_\text{KL}[q(\mathbf{Z};\xi)||p(\mathbf{Z}|\mathbf{X})], \quad \mathcal{L}(\xi):=\int q(\mathbf{Z};\xi)\log\frac{p(\mathbf{X},\mathbf{Z})}{q(\mathbf{Z};\xi)}d\mathbf{Z}
$$

対数周辺尤度 $\log p(\mathbf{X})$ は $\xi$ に依存しないため、この設定において ELBO の最大化は KL ダイバージェンスの最小化と等価になる。

#### （余談）
> この界隈、ちょっと気に入らないのが何でもELBOの最大化から話を始める点ですね。KL divergenceの最小化と等価であることをまず抑えたほうが良いです。元々変分ベイズとして解こうとしていたのが無邪気に変なテク入れているうちにEMアルゴリズムになっちゃってたり色々残念な感じになりがちです。
> (https://twitter.com/sammy_suyama/status/1563756088599920640 )

### ガウス分布
特別な確率分布として、以下の $D$ 次元ガウス分布が頻出する：
$$
\mathcal{N}(\mathbf{x}|\mathbf{\mu},\mathbf{\Sigma}):=\frac{1}{\sqrt{(2\pi)^D|\mathbf{\Sigma}|}}\exp\left(-\frac{1}{2}(\mathbf{x}-\mathbf{\mu})^\mathsf{T}\mathbf{\Sigma}^{-1}(\mathbf{x}-\mathbf{\mu})\right)
$$
ここで $\mathbf{\mu}\in\mathbb{R}^D$ は平均、$\mathbf{\Sigma}\in\mathbb{R}^{D\times D}$ は分散共分散行列。

ガウス分布の周辺分布や条件つき分布はガウス分布になる。ガウス分布は KL ダイバージェンスが解析的に求められるなど便利なため、論文でも多くの事前分布にガウス分布を仮定している。

## 拡散モデル
元画像 $\mathbf{x}_0$ にノイズを載せる処理を $T$ 回繰り返すことで、ランダムな画像 $\mathbf{x}_T$ を生成することを考える。この逆過程を推定し、ランダムな画像 $\mathbf{x}_T$ から意味のある画像 $\mathbf{x}_0$ を生成することが目標である。

### Forward diffusion process

（書きかけ）