---
title: "Wavelet Matrix"
draft: false
tags: ["Data Sturucture"]
math: true
---

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