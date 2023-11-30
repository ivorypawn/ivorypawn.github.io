---
title: "連想コンテナ覚書 (C++)"
draft: true
tags: ["Data Sturucture"]
---
`std::unordered_map` や `boost::flat_map` などのハッシュマップはほぼ $O(1)$ で探索でき，赤黒木の `std::map` に比べて探索も走査も速いとされている．

- [[C++] STLの型の使い分け](https://qiita.com/h_hiro_/items/a83a8fd2391d4a3f0e1c)
- [std::mapを線形探索してはいけない100の理由](https://www.slideshare.net/kikairoya/map-17091504)
- [mapとunordered_mapのアイテム全走査の比較とmapの存在意義について考える](https://sleepy-yoshi.hatenablog.com/entry/20121111/p1)

また `unordered_map` と `map` の使用は似ており，置換すること自体はさほど難しくない．

では常に `map` の代わりに `unordered_map` を使えばいいのかというと，いろいろと問題がある．

まず `map` は半順序さえ定義しておけば任意のユーザー定義型をキーに使えるが，`unordered_map` は `std::hash` の特殊化などが必要になる．

- [pairをキーにしたstd::unordered_mapを手軽に使えるようにする](https://qiita.com/hamamu/items/4d081751b69aa3bb3557)

また要素数が少ないときは `unordered_map` のほうが定数倍が大きく，遅くなることがある．実際，適当な競プロで比較実験したところ，実行時間にはほとんど差がなかった．入力が大きい問題であっても，重複を削除したあとの要素数がさほどでもなく，定数倍の影響が効く可能性がある．競プロではとりあえず `map` で書いておき，TLE 境界例が出たときだけ考慮すればよいと思う．

さらに VC++ では以下のような罠が存在した．
- [VC++のunordered_set/unordered_mapが遅い件](https://xoinu.hatenablog.com/entry/2016/12/10/092748)

仮に `map` を `unordered_map` に置換する場合，動作テストだけでなくベンチマークが必須．

なおコンテナの検索が不要で，単に（ソートされた）ユニークなリストが欲しいだけであれば，`std::vector` + `std::sort` + `std::unique` のほうが遥かに高速である．
- [コンテナへの挿入の速度について](https://xoinu.hatenablog.com/entry/2014/08/14/105740)
- [std::sortとstd::uniqueでstd::vectorの重複要素を削除する](https://qiita.com/ysk24ok/items/30ae72f4f1060b088588)

ハッシュテーブル一般については私はほとんど理解していないが，`unordered_map` より高速なハッシュマップはいくつもあるとのこと．
- [google::dense_hash_mapを使う](https://dfukunaga.hatenablog.com/entry/2018/04/28/104242)
- [高速なハッシュテーブルを設計する](https://postd.cc/designing-a-fast-hash-table/)
- [私が書いた最速のハッシュテーブル – PART 1](https://postd.cc/i-wrote-the-fastest-hashtable-1/)

必要に応じて勉強したい．