---
title: "Wavelet Matrix"
draft: false
tags: ["Data Sturucture"]
---

## 概要
～数十億の大規模データ上で、特定範囲の極値や頻度などが高速に求められる構造。

多次元への拡張も可能。

## 読んだやつ
1. Francisco Claude and Gonzalo Navarro. [The Wavelet Matrix](https://users.dcc.uchile.cl/~gnavarro/ps/spire12.4.pdf). Proc. SPIRE'12, pages 167-179. LNCS 7608
2. [ウェーブレット木の世界](https://tech.preferred.jp/ja/blog/wavelettree_world/)
3. [人工知能学会 私のブックマーク Vol.26 No.6 (2011/11) 簡潔データ構造](https://www.ai-gakkai.or.jp/resource/my-bookmark/my-bookmark_vol26-no6/)
4. [簡潔ビットベクトル(完備辞書)](https://tiramister.net/blog/posts/bitvector/)
5. [ウェーブレット行列最速攻略](https://www.scribd.com/doc/102636443/Wavelet-Matrix)
6. Eating Your Own Cat Food
   - [コード](https://github.com/MitI-7/WaveletMatrix)
   - [ウェーブレット行列(wavelet matrix)](https://miti-7.hatenablog.com/entry/2018/04/28/152259)
   - [ウェーブレット行列で競プロの問題を解く](https://miti-7.hatenablog.com/entry/2019/02/01/152131)
   - [動的ウェーブレット行列(dynamic wavelet matrix)](https://miti-7.hatenablog.com/entry/2019/02/01/143655)
7. [3 次元空間のクエリを処理する Wavelet Matrix](https://noshi91.hatenablog.com/entry/2021/06/02/165408)
8. [SIGGRAPH Asia の最優秀賞論文を解説してみた（2D Wavelet 行列を用いた定数時間メディアンフィルタ）](https://qiita.com/TumoiYorozu/items/a8193e5ecd957a1b9b0d)

ウェーブレット行列の構造についてはすでに分かりやすい資料がある（とくに 2., 6., 8. がおすすめ）。

ここでは基本的な部分は取り上げない。以下は個人的なメモ。

### （余談）
上に載せていないが、ウェーブレット行列の名をもつとあるライブラリが、節点情報をアルファベット数分保持する実装になっていた。

[資料](#読んだやつ) 2. p.51 の通り、前身のウェーブレット木に対するウェーブレット行列のアドは、各ビット列の $1$ の開始位置だけ保持すればよいことにある。したがってこのライブラリの実装は誤り。

## 表記
文字列データを $T[0,n),\\;0\leq T[i]<s$ で表す。$s$ はアルファベットの数。
- DNA：$s=4$ (ATCG)
- ASCII: $s=2^7$
- 日本語： $s=$ 数万
- etc.

とくに $s=2$ のときのビット列を $B[0,n)$ と表す。

## 実装
### 構築
以下 C++ で書く。
```cpp
class WaveletMatrix {
   private:
    vector<SuccinctBitVector> bit_arrays;
    vector<uint64_t> begin_one;              // 各 bit に着目したときの 1 の開始位置
    map<uint64_t, uint64_t> begin_alphabet;  // 最後のソートされた配列で各文字の開始位置

    const uint64_t size;             // 与えられた配列のサイズ
    const uint64_t maximum_element;  // 文字数
    const uint64_t bit_size;         // 文字を表すのに必要な bit 数

   public:
    WaveletMatrix(const vector<uint64_t> &array)
        : size(array.size()), maximum_element(*max_element(array.begin(), array.end()) + 1), bit_size(get_num_of_bit(maximum_element)) {
        bit_arrays.resize(bit_size, SuccinctBitVector(size));
        begin_one.resize(bit_size);

        vector<uint64_t> v(array), temp[2];
        temp[0].reserve(size);
        temp[1].reserve(size);
        for (uint64_t i = 0; i < bit_size; ++i) {
            temp[0].clear();
            temp[1].clear();

            for (uint64_t j = 0; j < size; ++j) {
                uint64_t c = v[j];
                uint64_t bit = (c >> (bit_size - i - 1)) & 1;
                temp[bit].push_back(c);
                bit_arrays[i].setBit(bit, j);
            }
            bit_arrays[i].build();

            begin_one[i] = temp[0].size();
            temp[0].insert(temp[0].end(), temp[1].begin(), temp[1].end());
            v.swap(temp[0]);
        }

        for (int i = size - 1; i >= 0; --i) {
            begin_alphabet[v[i]] = i;
        }
    }
}
```
`SuccinctBitVector` やいくつかのメンバ関数の実装は省略。コンストラクタでウェーブレット行列を構築している。

`begin_alphabet` は `rank` を高速化する。動的ウェーブレット行列では使えない（要確認）。

また範囲指定クエリのみが必要で（空間の干渉判定など）、特定の値についてカウントする必要がない場合は、`rank` の出番がないため、`begin_alphabet` を省略できる。

#### 基数ソートとの違い
実装のウェーブレット行列の振り分けステップは基数ソートに似ているが、基数ソートと違って最上位ビットから比較している。このため最終的に bit-reversed にソートされた配列が生成される。

この仕様はオリジナルの論文に準拠しており、ビットサイズを節約できる利点がある（cf. [資料](#読んだやつ) 1. p.7 *Practical considerations*）。

上の `WaveletMatrix` の実装は引数の `array` ごとにビットサイズを決定するため、最下位ビットから比較しても効率は変わらない。このほうが基数ソートの理解をそのまま使えて可読性が高いかもしれない。

### `rank(c, pos)`
$T[0,\text{pos})$ における $c\in [0,s)$ の頻度を返す。

ビット列 $B$ 上の `rank` は、まず $B$ を 2 段階にブロック分割し、構築時に各ブロックの `rank` を計算して記録しておき、残りを `popcount` ないしテーブルから求めて足し上げる。
分割ブロックの適切なサイズについては [資料](#読んだやつ) 4. 参照。

```cpp
uint64_t rank0(uint64_t pos) {
    if (pos == 0) {
        return 0;
    }
    pos = min(pos, size);
    const uint64_t ones = (uint64_t)((1 << (pos % blockBitNum)) - 1);
    return pos - (L[pos / LEVEL_L] + S[pos / LEVEL_S] + __popcnt16(B[pos / blockBitNum] & ones));
}
```

文字列 $T$ 上の `rank` は、$c$ に対応する葉に至るまでウェーブレット木をたどって、ビット列 $B$ 上の `rank` を適用する。[資料](#読んだやつ) 5. p.44 など参照．

```cpp
uint64_t rank(const uint64_t c, uint64_t pos) {
    if (c >= maximum_element) {
        return 0;
    }
    auto it = begin_alphabet.find(c);
    if (it == begin_alphabet.end()) {
        return 0;
    }
    for (uint64_t i = 0; i < bit_size; ++i) {
        uint64_t bit = (c >> (bit_size - i - 1)) & 1;
        pos = bit_arrays[i].rank(bit, pos);
        if (bit) {
            pos += begin_one[i];
        }
    }
    const uint64_t begin_pos = it->second;
    return pos - begin_pos;
}
```

### `rangefreq(x, y, l, r)`

$T[l,r)$ における $[x,y)\subset [0,s)$ に含まれる値の頻度を返す。

特別な場合 `lowfreq(c, l, r) := rangefreq(0, c, l, r)` が実装できればよい。実装は `rank` 同様に書ける。

```cpp
// [0, c) && [begin, pos)
uint64_t lowFreq(uint64_t c, uint64_t begin_pos, uint64_t end_pos) {
    end_pos = min(size, end_pos);
    if (c == 0 || end_pos <= begin_pos) {
        return 0;
    }
    if (c >= sup) {
        return end_pos - begin_pos;
    }
    uint64_t count = 0;
    for (uint64_t i = 0; i < bit_size && begin_pos < end_pos; ++i) {
        const uint64_t bit = (c >> (bit_size - i - 1)) & 1;
        const uint64_t rank_begin = bit_arrays[i].rank0(begin_pos);
        const uint64_t rank_end = bit_arrays[i].rank0(end_pos);
        if (bit) {
            count += rank_end - rank_begin;
            begin_pos = begin_one[i] + begin_pos - rank_begin; // BeginOne + rank1(begin_pos)
            end_pos = begin_one[i] + end_pos - rank_end; // BeginOne + rank1(end_pos)
        } else {
            begin_pos = rank_begin;
            end_pos = rank_end;
        }
    }
    return count;
}

// [min_c, max_c) && [begin, pos)
uint64_t rangeFreq(const uint64_t min_c, const uint64_t max_c, uint64_t begin_pos, uint64_t end_pos) {
    if (max_c <= min_c || end_pos <= begin_pos) {
        return 0;
    }
    return lowFreq(max_c, begin_pos, end_pos) - lowFreq(min_c, begin_pos, end_pos);
}
```

## 多次元への拡張
ウェーブレット行列 `WaveletMatrix` は簡潔ビットベクトル `SuccinctBitVector` のベクトルを持ち、文字列 $T$ の `rank` や `lowfreq` の計算を `SuccinctBitVector.rank` に帰着していた。

同様に `WaveletMatrix` のベクトルをもつウェーブレットテンソル `WaveletTensor` なるクラスを考え、3 次元空間上のクエリ `lowfreq` を `WaveletMatrix.rank` に帰着して求めることができる。

`WaveletTensor` の使用例は以下：
- メディアンフィルタ画像処理（cf. [資料](#読んだやつ) 8.）
- 3 次元空間の点群の干渉判定（cf. [資料](#読んだやつ) 2. p.44, [資料](#読んだやつ) 7.）

試しに後者を簡易実装してみる。座標 $(X,Y,Z)$ は適当な変換により非負になっているものとする。また適当な解像度により $X,Y$ は離散化され整数になっているとする。
1. 点群を $Z$ 座標でソート
2. $Y$ 座標のビットを最上位から注目し `WaveletMatrix` 同様に振り分け
3. 2. で定まる順序について以下の文字列 $T$ を生成：
   - Bit 0: $X$ 座標
   - Bit 1: $X$ の上界
4. $T$ の `WaveletMatrix` を構築。以降、$Y$ 座標の最下位ビットにいたるまで繰り返し

すなわち $X,Y$ 座標を $b$ ビット整数で表すとき、最大 $b$ 個の `WaveletMatrix` を構築する。

```cpp
class WaveletTensor {
private:
    const uint64_t size;
    const uint64_t supX, supY, bit_size;

    vector<WaveletMatrix> tensor; // 最上位ビットから
    vector<uint64_t> begin_one;

public:
    WaveletTensor(const vector<tuple<uint64_t, uint64_t, float> >& points3D, const uint64_t maxX, const uint64_t maxY) :
        size(points3D.size()), supX(maxX + 1), supY(maxY + 1), bit_size(get_num_of_bit(supY))
    {
        tensor.resize(bit_size);
        begin_one.resize(bit_size);

        vector<pair<uint64_t, uint64_t> > v, temp[2];
        temp[0].reserve(size);
        temp[1].reserve(size);
        for (uint64_t i = 0; i < bit_size; ++i) {
            temp[0].clear();
            temp[1].clear();

            vector<uint64_t> codX(size);
            for (uint64_t j = 0; j < size; ++j) {
                const uint64_t cX = i == 0 ? get<0>(points3D[j]) : v[j].first;
                const uint64_t cY = i == 0 ? get<1>(points3D[j]) : v[j].second;
                const uint64_t bit = (cY >> (bit_size - i - 1)) & 1;
                temp[bit].emplace_back(cX, cY);
                codX[j] = bit ? supX : cX;
            }
            tensor[i].init(codX, supX);

            begin_one[i] = temp[0].size();
            temp[0].insert(temp[0].end(), temp[1].begin(), temp[1].end());
            v.swap(temp[0]);
        }
    }
};
```

### `rangefreq(x0, x1, y0, y1, z0, z1)`
`WaveletMatrix.rangefreq` 同様、特別な場合 `lowFreq(cx, cy, l, r) := rangefreq(0, cx, 0, cy, l, r)` が実装できればよい。

```cpp
// [0, cX) && [0, cY) && [begin, end)
uint64_t lowFreq(uint64_t cX, uint64_t cY, uint64_t begin_pos, uint64_t end_pos) {
    end_pos = min(size, end_pos);
    if (cX == 0 || cY == 0 || end_pos <= begin_pos) {
        return 0;
    }
    cX = min(cX, supX);
    cY = min(cY, supY);
    if (cX == supX && cY == supY) {
        return end_pos - begin_pos;
    }
    uint64_t count = 0;
    for (uint64_t i = 0; i < bit_size && begin_pos < end_pos; ++i) {
        const uint64_t bit = (cY >> (bit_size - i - 1)) & 1;
        const uint64_t rank_begin = tensor[i].lowFreq(supX, 0, begin_pos);
        const uint64_t rank_end = tensor[i].lowFreq(supX, 0, end_pos);
        if (bit) {
            count += tensor[i].lowFreq(cX, begin_pos, end_pos);
            begin_pos = begin_one[i] + begin_pos - rank_begin;
            end_pos = begin_one[i] + end_pos - rank_end;
        } else {
            begin_pos = rank_begin;
            end_pos = rank_end;
        }
    }
    return count;
}

// [x0, x1) && [y0, y1) && [z0, z1)
uint64_t rangeFreq(const uint64_t x0, const uint64_t x1, const uint64_t y0, const uint64_t y1, uint64_t z0, uint64_t z1) {
    if (x1 <= x0 || y1 <= y0 || z1 <= z0) {
        return 0;
    }
    return lowFreq(x1, y1, z0, z1) - lowFreq(x0, y1, z0, z1) - lowFreq(x1, y0, z0, z1) + lowFreq(x0, y0, z0, z1);
}
```

雑に動作テストずみだが、カバレッジはぜんぜんないので参考程度に。