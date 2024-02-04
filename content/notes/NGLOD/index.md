---
title: "Neural Geometric Level of Detail"
draft: true
tags: ["論文読み", "Geometry Processing"]
---

## 資料
- Towaki Takikawa, Joey Litalien, Kangxue Yin, Karsten Kreis, Charles Loop, Derek Nowrouzezahrai, Alec Jacobson, Morgan McGuire, & Sanja Fidler (2021). [Neural Geometric Level of Detail: Real-time Rendering with Implicit 3D Shapes](https://research.nvidia.com/labs/toronto-ai/nglod/).
- Talk: [Toronto Geometry Colloquium - Siddhartha Chaudhuri & Towaki Takikawa]()
- Towaki Takikawa, Or Perel, Clement Fuji Tsang, Charles Loop, Joey Litalien, Jonathan Tremblay, Sanja Fidler, & Maria Shugrina. (2022). [Kaolin Wisp: A PyTorch Library and Engine for Neural Fields Research](https://github.com/NVIDIAGameWorks/kaolin-wisp).

## Neural SDFs
符号つき距離関数（signed distance functions; SDFs）$f: \mathbb{R}^{3} \rightarrow \mathbb{R}$ は，オブジェクト $\mathcal{M}$ の境界 $\mathcal{S}=\partial\mathcal{M}$ からの符号つき距離を表す 3D 表現の一種．

オブジェクト内部で $f<0$ を約束し，境界 $\mathcal{S}$ を 0-等値面で表す．SDF は点群やメッシュとは異なり境界の構成要素が陽に存在せず，境界全体が陰関数表現される．

![](./01.jpg#center)

*Neural SDF* $f_\theta$ は $f$ を近似するパラメータ $\theta$ のニューラルネットワーク．向きの揃った法線つき点群やマルチビューから推定することが多い．

Neural SDF によるレンダリングは視点によらず滑らかである．また空間的コストが $|\theta|$ で抑えられるため，複雑な形状ではメッシュなどと比べてコンパクトになる特性をもつ．

いっぽう一般に Neural SDF はレンダリングと学習のコストが高いという課題がある．典型的な Neural SDF は sphere tracing というアルゴリズムでレンダリングするが，これは各レイに対し比較的サイズの大きい MLP を相当ステップ適用するため，計算コストが高くなる．

![](./02.jpg#center)

このため Neural SDF のリアルタイムアプリケーションへの適用は非現実的であったが，この論文は LOD と組み合わせてこの課題を解決している．

## 関連：NeRF
> Ben Mildenhall, Pratul P. Srinivasan, Matthew Tancik, Jonathan T. Barron, Ravi Ramamoorthi, & Ren Ng (2020). [NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis](https://www.matthewtancik.com/nerf). In ECCV.

関数 $F_{\Theta}:(x, y, z, \theta, \phi) \rightarrow(R, G, B, \sigma)$ をマルチビューから最適化する．$\theta, \phi$ は座標 $(x,y,z)$ におけるレイの方向（前節の $\theta$ とはべつもの），$\sigma$ はオブジェクト密度を表す．

レンダリングは色関数と密度関数の積のレイに沿った重みつき積分によって計算される．重みは $\sigma$ の累積に対して指数的に減衰するため，レイが手前側で干渉する境界上の点以外の影響は無視される．

$F_{\Theta}$ の出力は入力に対してセンシティブであるが，ニューラルネットワークは一般に高周波成分の大きい関数の近似が苦手．これを解決するため positional encoding によってあらかじめ $(x, y, z, \theta, \phi)$ を成分分解したものを入力する工夫をしている．

NeRF は 6 ヶ月以内に少なくとも 25 の後続研究が登場し，高速化，動的化，リライティング，合成の拡張がされた：[NeRF Explosion 2020](https://dellaert.github.io/NeRF/)．

## 提案手法：NGLOD
![](./03.jpg#center)

オブジェクトは $\mathcal{B}=[-1,1]^3$ に収まるようスケーリングされているとする．

