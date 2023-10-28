---
title: "Stochastic Poisson Surface Reconstruction"
draft: true
tags: ["論文読み", "Geometry Processing"]
---

## 概要
> Silvia Sellán, & Alec Jacobson (2022). [Stochastic Poisson Surface Reconstruction](https://www.dgp.toronto.edu/projects/stochastic-psr/). ACM Transactions on Graphics.

を読んだ．

ベースとなる先行研究の [Poisson Surface Reconstruction](https://hhoppe.com/proj/poissonrecon/) は，法線つき点群からメッシュを復元する手法．PSR は点群が欠損していたりノイズを含んでいる場合に精度が悪かった．

この論文では PSR にベイズ推定を組み合わせて，事前分布を指定することで上の課題をクリアしている．

発想が天才．

## Poisson Surface Reconstruction
入力となる法線つき 3 次元点群 $\mathcal{S}$ は，あるソリッドモデル $\Omega$ のサーフェス $\partial\Omega$ 近傍に観測されるものとする．各 $s\in\mathcal{S}$ は座標 $p_s$ および正規化された法線 $\vec{N}_s$ を持つ．

$\Omega$ の陰関数表現 $f_\text{PSR}:\mathbb{R}^3\rightarrow\mathbb{R}$ を推定し（⇔ $\partial\Omega$ からの符号つき距離；$\Omega$ 内部で $f_\text{PSR}>0$ とする），$0$ 等値面によって $\partial\Omega$ を再構成する．

対象領域をグリッド $\mathcal{O}$ で分割する（論文では点群に対して adaptive なグリッドを取ることを推奨している）．グリッドの各ノード $o$ に対して，$F_o:\mathbb{R}^3\rightarrow\mathbb{R}$ をノード中心が極大の RBF カーネル（をコンパクト台をもつよう近似した関数）とおく．

$f_\text{PSR}$ の勾配 $\vec{V}\_\text{PSR}:\mathbb{R}^3\rightarrow\mathbb{R}^3$ を以下で定める：
$$
\vec{V}\_\text{PSR}(q)=\sum\_{s \in \mathcal{S}} \frac{1}{W(p\_s)} \sum\_{o \in B(s)} \alpha\_{o, p\_s} F\_o(q) \vec{N}\_s,\quad W(q)=\sum\_{s \in \mathcal{S}}\sum\_{o \in B(s)}\alpha\_{o, p\_s} F\_o(q).
$$
ここで $B(s)$ および $\alpha_{o, p_s}$ は，$F_o>0$ となるグリッド集合および相対位置によって定まる係数である．Stochasitic PSR の論文では，trilinear 補間で定まる 8 近傍および係数を採用している（これは簡略化したもので，もとの PSR 論文では Octree 上の深さ $D$ によって定まる近傍によって定式化している）．また $W(q)$ は局所サンプル密度と呼ばれる．

上記は RBF カーネルによる（フーリエ）畳みこみで軟化した指示関数に対するストークスの定理を離散化した結果になっている（たぶん）．右辺に RBF カーネルが現れるのはそういった理由による．
- [滑らかなカットオフ函数](https://ja.wikipedia.org/wiki/%E8%BB%9F%E5%8C%96%E5%AD%90#%E6%BB%91%E3%82%89%E3%81%8B%E3%81%AA%E3%82%AB%E3%83%83%E3%83%88%E3%82%AA%E3%83%95%E5%87%BD%E6%95%B0)

あとは Poisson 方程式 $\Delta f_\text{PSR}=\nabla\cdot\vec{V}_\text{PSR}$ を解けばよい．離散化したラプラシアンとダイバージェンスによって，
$$
\mathbf{Lf}\_\text{PSR}=\mathbf{Zv}\_\text{PSR}
$$
をノイマン境界条件 $\nabla f\cdot n=0$ のもとで解く．解 $\mathbf{f}\_\text{PSR}$ は平行移動に関して不定性があるため，$\mathcal{S}$ にフィットする切片を選ぶ．

以上で $\Omega$ に含まれるグリッドノードが決定した．ここから，たとえばマーチングキューブ法によってメッシュが生成できる．

### （メモ：マーチングキューブ法）
オリジナルの 15 種によるマーチングキューブ法は，いくつかのケースで曖昧性があり，解像度が低いと穴が開いてしまうらしい（未精査）．実装時にはちゃんと調べたい．
- [穴の開かない Marching Cubes 法の lookup table 2種](https://blog.oimo.io/2022/06/11/mc-tables/)

## 陰関数表現のベイズ化

上記の $\vec{V}_\text{PSR}$ を生成するようなガウス過程を構成することを考える．つまり法線つき点群 $\mathcal{S}=\{(p_s,\vec{N}_s)\}$ が観測されているとき，
$$
\vec{V}(q)|\mathcal{S} \sim \mathcal{N}(\mathbf{k}\_2^\mathsf{T} \mathbf{K}\_3^{-1} \vec{N}\_s, k\_1-\mathbf{k}\_2^\mathsf{T} \mathbf{K}\_3^{-1} \mathbf{k}\_2), 
$$
$$
k_1=k(q, q), \quad \mathbf{k}\_2=(k(q, p\_s))\_{s \in \mathcal{S}} \in \mathbb{R}^{|\mathcal{S}|}, \quad \mathbf{K}\_3=(k(p\_s, p\_{s^{\prime}}))\_{s, s^{\prime} \in \mathcal{S}} \in \mathbb{R}^{|\mathcal{S}| \times|\mathcal{S}|}
$$
を満たすようなガウス過程を定めるカーネル関数 $k(x,x')$ を設計したい．

なお上の定式化では，まだ観測にノイズが含まれることを考慮していない．実際 [ベイズ線形回帰モデルとの関係]() において，$\sigma^2_y=0$ とおいて書き直せば上式と等価になる（SPSR 論文では $\sigma^2_n$ という変数になっている．なぜ添え字が $n$ かはよく分からない）．

またここでは適当な前処理によって，平均 $\mathbf{m}\equiv\mathbf{0}$ を仮定している．この条件はガウス過程を使用する上で，しばしば簡単のため仮定されることもあるが，恣意的に変えられる．たとえば $\mathbf{m}$ が適当な球面上で $0$ を取るような事前分布を与えることによって，点群の欠損部をうまく補完したメッシュを生成できる．

### カーネル関数 $k_\text{SPSR}$
"半"共分散関数 $k_\text{PSR}(x,y)$ を以下で定義する．
$$
k\_\text{PSR}(x,y):=\sigma\_g\sum\_{o\in B(x)}\alpha\_{o,x} F\_o(y).
$$
このとき，$\vec{V}\_\text{PSR}(q)$ は以下のように書ける．
$$
\vec{V}\_\text{PSR}(q)=\sum\_{s \in S} k\_\text{PSR}(p\_s,q) \frac{1}{\sigma\_g W(p_s)} \vec{N}_s.
$$

$k_\text{PSR}$ をカーネルに採用したいところだが，明らかに対称性を満たしていない．そこで以下の $k_\text{SPSR}$ および $\vec{V}\_\text{SPSR}$ を代えて採用する：
$$
k\_\text{SPSR}:={1\over 2}(k\_\text{PSR}(x,y)+k\_\text{PSR}(y,x)),\quad \vec{V}\_\text{SPSR}(q)=\sum\_{s \in S} k\_\text{SPSR}(p\_s,q) \frac{1}{\sigma_g W(p\_s)} \vec{N}_s.
$$
（正定値性と可逆性についてはいったん保留）

このようにしても数値的な差はほとんどない．

### 集中共分散行列 $\mathbf{D}$
ガウス過程は，学習・推論時にグラム行列 $\mathbf{K}^{-1}$ を計算しなければならない．これは $O(N^3)$ かかり，$|\mathcal{S}|>1000$ あたりから実質的に計算不可能になる．
- 持橋大地『[ガウス過程の基礎と教師なし学習](https://www.ism.ac.jp/~daichi/lectures/H26-GaussianProcess/)』
  - 統計数理研究所 H26年度公開講座「ガウス過程の基礎と応用」

一般的な回避措置としては，たとえば [誘導入力＋変分推論法]() がある．これは仮想的な $m$ 個の入力 $X_m$ を考え，これらが $\Omega$ を「代表」するように最適化することで計算量を $O(m^2N)$ に落とす．

しかし SPSR の状況を考えると，ソリッドモデル $\Omega$ を代表する $m$ 個の点が存在することが一般に言えるか定かではない．また低ランク近似を使わなくとも，変分推論において各入力の独立性を仮定すると（おそらく平均場近似のことを言っている），SPSR の場合は精度上問題があることが論文で指摘されている．

代えて，SPSR は以下の集中共分散行列（対角行列）$\mathbf{D}$ を提案する：
$$
\mathbf{D}:=\text{diag}(\sigma\_g w)\approx \mathbf{K}\_3.
$$
一見して「こんなんでいいの？」という感じだが，よりよい近似になっているとのこと．これは有限要素法における集中質量行列（lumped mass matrix）からの類推である．ガウス過程の計算量削減の手法としては，ほかに例を見かけない．

上の近似によって，$\vec{V}(q)$ を以下のように書き直す：
$$
\vec{V}(q)|\mathcal{S} \sim \mathcal{N}(\mathbf{k}\_2^\mathsf{T} \mathbf{D}^{-1} \vec{N}\_s, k_1-\mathbf{k}\_2^\mathsf{T} \mathbf{D}^{-1} \mathbf{k}\_2).
$$
平均 $\mathbf{k}\_2^\mathsf{T} \mathbf{D}^{-1} \vec{N}\_s$ を具体的に計算すると，実は $\vec{V}\_\text{SPSR}(q)$ に一致する．このことより，上式は勾配関数 $\vec{V}\_\text{SPSR}$ のガウス過程への拡張である．

先述の離散ラプラシアン $\mathbf{L}$ および離散グラディエント $\mathbf{Z}$ によって，最終的な陰関数表現のガウス過程 $\mathbf{f}|\mathcal{S}$ を以下のとおり得る．
$$
\mathbf{f} |\mathcal{S} \sim \mathcal{N}(\mathbf{L}^{-1} \mathbf{Z} \mathbf{V}\_\text{SPSR}, \mathbf{L}^{-1} \mathbf{Z K}\_{\mathbf{V}} \mathbf{Z}^\mathsf{T}(\mathbf{L}^\mathsf{T})^{-1})
$$

### （メモ：Galerkin 行列）
集中共分散行列 $\mathbf{D}$ は集中質量行列からの類推と書いたが，有限要素法では Galerkin 行列もしばしば使われる．たとえば Poisson 方程式 $\Delta u=f$ の離散化を解くとき，Galerkin 行列のほうが精度がよい．

Galerkin 行列の逆行列を求めるアルゴリズムを探してみると，以下の論文がヒットした（中はちゃんと読んでない）：
> Mario Bebendorf. (2005). [Efficient Inversion of the Galerkin Matrix of General Second-Order Elliptic Operators with Nonsmooth Coefficients](https://www.jstor.org/stable/4100175). Mathematics of Computation, 74(251), 1179–1199.

とはいえ集中共分散行列ですでに精度が悪くないのなら，このあたりは改善の余地がないかもしれない．

## 応用
(Under construction)