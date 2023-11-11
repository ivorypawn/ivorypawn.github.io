---
title: "将棋 AI を利用した学習に関する雑感"
date: 2023-11-07T16:15:15+09:00
draft: true
tags: ["将棋", "Chess"]
---

自身の敗局を並べるのは，有効な棋力向上の手段である．そこには主に 2 つの観点が存在する．

1. どれが悪手で何が最善だったのか（**客観的・演繹的**）
2. なぜ最善を選べなかったのか（**主観的・帰納的**）

実際には『このあたりで間違えたな．何が正解だったんだろう．こうか．なるほど』の段階で反省が終わり，なぜ最善手が見つけられなかったのか，あるいはなぜ見つけていながらほかの手を選んでしまったかという分析までは行わないことが多い．それが悪いわけではない．たとえば単なる技術不足が敗因なら，1. までに留めて反省のサイクルを早く回すほうがよい．また意識的な 2. の実施は自らの欠点を直視しなければならず，辛い作業かもしれない．

そもそも心理的な仮説の検証は難しい．心理学一般においても，ダニング・クルーガー効果や自我消耗仮説など，かつて定説のように扱われたこともある多くの主張が再現性を疑われている（cf. [心理学・行動経済学等の著名な研究論文が次々に追試失敗【心理学】](https://note.com/s1000s/n/na0dbd2e8632d)）

一方 1. に時間を費やしているにも関わらず，勝率が伸びない状況はしばしばある．普段なら見える最善手を実戦で逃してしまう，似たような間違いを繰り返しているかもしれない，何かのきっかけで勝てる気もするがその何かがわからない……などなど．かような状態は 2. の分析不十分が原因かもしれない．

一例をジョッシュ・ウェイツキン著（吉田俊太郎訳）『習得への情熱』から引用する．

> あの時期の僕にとって，大会の対局中に犯したミスのほとんどが，例外なく，その時点で僕が背負っていた心の重荷と深く関係していた．チェス盤上に現れる問題が，チェス以外の日常の心の問題を如実に示していたことに僕は気がついた．(p.91)

> チェスにおけるそういった僕の心理的問題点は，つまりは，それまでの形に拘泥してしまうことに帰結するものであり，それは僕が根深いところでホームシックの心理状態になっているからにほかならない．ようやくこの関連性に気が付いた僕は，チェスにおいても，日常においても，意識してこの変わり目の問題に対処するようになった．チェスの対局では，戦いの性質が変化したところで何度か深呼吸をして頭をすっきりさせるようにした．日常では，変化に逆らうのではなく，気持ちよく受け入れるようにした．(p.93)

ウェイツキンは 2. を重要視しているが，たとえば上記のパフォーマンスに悪影響を及ぼしていた心理状態について，彼がどのような過程で自己診断を下したのかなどの記述はない．この部分は結局，ウェイツキン個人の経験や洞察力，チェスへの理解度に依存している．非才の身なる我々は，どうすれば『思うように勝てないのはホームシックが関係しているかもしれない』などと思いつけるだろう．そしてその仮説や考えうる対策の効果を，どうやって検証すればよいのか．

ウェイツキンの著書は，大半が彼の経験談と観念的な議論からなる．そのようにしか語りえないことはあるだろうし，彼の文章は実際魅力的だが，再現性のある方法論には計量と統計はやはり不可欠だ．本記事の後半で，そのような方法の構築を試みる．

## ◆
何が悪手だったか把握しないまま最善手を選べなかった理由に悩むのはナンセンスだ．反省は常に 1. ⇒ 2. の順に進行する．AI 以前は必然的に 1. にリソースの大半が充てられていた．自分より強い存在が傍らにおり，疑問にすぐ答えが返ってくるような環境は例外的だった．

AI の発達によって，人間の学習において 1. の精度向上およびリードタイム短縮が可能になった．現在の AI 活用の大半がこの形式をとる．とはいえ 1. の重要性自体は AI 前後でなにも変わっていない．上位プレイヤーはそれを理解しているから，課題局面における AI の判断を最終的には受け入れるにせよ，初めから無条件に受け入れはしない．これによって，微妙に条件が違う局面が現れても大勢を見誤ることは少なく，まったく未知の局面において既知の局面との類似点を見出し方針を立てることができる．

上記のアンチパターンについても考えてみよう．

(Under Construction)

<!-- こうした向きはアマ五段あたりをボリュームゾーンに点在する．

あるいは，ある局面の AI の判断の意図を汲み取ることと，自分が同じ判断を下すことの難度の差を過小評価しているかもしれない（この関係は $\mathrm{P}\neq\mathrm{NP}$ 予想を連想させる）．

自分がいかに多くの間違いを犯しているか直視することに慣れていない． -->

## ◆
これまで 1. における既存の AI 活用について見てきた．次は 2. における活用の可能性を考えたい．具体的には，評価値による定量的な心理分析である．

私が大学将棋部にいたとき（ちょうど情報処理学会が，ソフトウェアがトッププロの実力を超えたことを宣言し，プロジェクトを終了したころだったと思う），評価値を用いた心理統計法的な分析手法を考え，ほかの部員にも頼んで試したことがあった．

あらかじめ断っておくと，データ収集が想像以上に面倒だったため，データ検証段階以前に当時の試みは頓挫した（後述するがこの部分は改善できる）．だからこの手法が役に立つ保証は今のところない．とはいえ，基本的な考え方は今も正しいと思っている．

### ◇
評価値は「勝ちやすさ」という人間にとって明瞭な解釈がある．AI の評価値が高い局面が必ずしも人間にとって勝ちやすいわけではないが（e.g. 非常に難解な唯一の勝ち筋がある局面でも AI は $+\infty$ の評価を下す），両者の相関は高く，定量分析の変数として十分信頼できると言ってよい．

まず自身の棋譜の解析結果から，最善手を指したときとの評価値の差が閾値 $T$ を超えた着手を取得し，特徴量のテーブルデータを作成する．抽出する特徴量はいろいろ考えられるが，一例を以下に示す．

- 最善手を選んだときの評価値 $E$
- 実戦の着手の評価値 $E_*$
- 評価値の差 $\Delta E=E-E_*$
  - $\Delta E>T$ のときその手を取得
  - 同じ $\Delta E=300$ であっても，$0\Rightarrow -300$ と $1500\Rightarrow 1200$ では深刻度が異なる．閾値 $T$ は $E=0$ で極小となるような変数にするのがよいかもしれない
- 先後
- 手数，進行度合
- 最終的な勝敗
- 駒の種類
- 盤上の駒 or 持ち駒
- 互いの玉との距離
  - 攻め／受けの推定

最後の項目はいまの思いつきだが，さらに想像をたくましくすると，盤上の駒全体の利きを分布とみなし，飛角香などに関しては途中に駒があれば減衰するなどの条件を課して，双方の玉を中心としたカーネルで畳み込めば，実際の玉の危険度とかなり相関しそうである．Conv 層は平面上の信号をカーネルを使って畳み込む処理の離散化だから，この類の特徴が CNN 系のソフトが内部で評価していることなのかもしれない．

上で挙げた特徴量は取得時に指し手の意思が介入せず，データ収集は完全に自動化できる．先述の将棋部での試みは，自動化のノウハウがなく手動でテーブルデータを作成したため，あまりに煩雑だった．理想的にはオンライン対局場のバックエンドで実装し，ユーザーからも隠蔽してしまうのがよいと思う．

いったんデータ収集が完了すれば，あとは一般的な特徴量エンジニアリングと効果検証の範疇である．

- 間違うときの傾向の推定
  - 序／中／終盤
  - 攻め／受け
  - 有利時／不利時
  - etc.
- 推定をもとにした施策の決定
- 施策導入前後の推定の比較

あとはこのサイクルを回せばよい．

### ◇
そのときの回答者の気分で結果が変わるかもしれない項目は，慎重に導入しなければならない．たとえば『このときの自身の形勢判断を 5 段階評価せよ』という設問はどうか．

<!-- またバックエンドにも隠蔽できない． -->

(Under Construction)

## ◆
ここまでの記述は，用語を変えればあらゆる分野に適用されるような，汎用的な議論である．AI が発達するずっと前から，本質的に同様のことを論じたテキストはたくさんあるに違いない．このテキストは，単に書き手の私が将棋界隈に首まで浸かってしまっているからという理由で，それらを将棋用語を使って焼き直したものにすぎない．

ではなぜ書くのか？ すでに書かれた，より洗練されたテキストを探せばよいのではないか？ あるいは LLM に生成させればよいのではないか？

そのように感じるならば，おそらく学習における 1. の重要性を評価できていない．こういうのは自分で考えて書くのがよいのだ．
