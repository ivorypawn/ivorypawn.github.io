---
title: "デザインに関する覚書"
draft: true
---
UI/UX，資料づくり，プレゼン本番で心がけることなどの個人用まとめ．

## UI 制作
- **これだけ守れば見やすくなるデザインの基礎** [1](https://speakerdeck.com/kinakobooster/koredakeshou-rebajian-yasukunarudezainfalseji-chu) / [2](https://speakerdeck.com/kinakobooster/uidezainwohazimeyou) / [3](https://speakerdeck.com/kinakobooster/jin-ri-karadekiruuxdezain)
  - 神資料．とくに 1 の RGB に関する解説は蒙が啓かれた
- [【個人開発・ポートフォリオに】簡単にいい感じのデザインにできるサービスまとめ](https://qiita.com/aiandrox/items/4196c8f5b564d29fdce7)
- [Sociomedia ヒューマンインターフェース ガイドライン](https://www.sociomedia.co.jp/category/shig)

### 色覚多様性
統計上，男性の 5% と女性の ~1% は同輝度の赤と緑の見分けがつかない（P 型色覚）．現実には明度の違いで見分けられているケースも多いと思うが，そうであっても赤と緑が素朴に使われている UI を見ると，「えっ大丈夫なの」とはなる．

異なる色覚の持ち主から UI がどう見えているかは，Chrome DevTools などを使えば確認できる．

- [色覚多様性(色覚異常)の見え方を Chrome DevTools でシミュレーションする](https://dev.classmethod.jp/articles/google-chrome-devtools-vision-deficiencies/)

### WYSIWYG について
WYSIWYG は *What You See Is What You Get* のアクロニムで，すべての操作を直感的に可能にするような UI のこと．代表例は PowerPoint．

WYSIWYG の宿命として「ボタン多すぎ問題」があり，ごく基本的な操作ですらメニューからいちいちボタンを探してマウスポチポチさせられがち．実際これが煩わしくて（ほかにも理由は色々あるが），私はパワポはなるべく使わない．
- [【まだパワポ！？】Markdownから爆速スライド作成【パワポが許されるのは小学生までだよね】](https://trap.jp/post/1341/)

マッキンゼーの『[なぜ今、日本に「デザイン」が必要なのか](https://www.mckinsey.com/jp/our-insights/japans-design-imperative)』によれば，
> WIPO (World Intellectual Property Organization、世界知的所有権機関) が公表している 2021 年のグローバルイノベーションインデックスでは、日本は 130 カ国中 13 位に止まっている 。日本製品は全般的に高品質であるというイメージが定着しているものの、時に、不必要な機能を多く備え、オーバースペックな場合がある。日本のメーカーは、他のグローバルな競合企業と比較して、変化への適応にも後れを取っている。

とのこと．ボタンではなく不必要な機能が多い場合は，UI だけでなんとかするよりモジュールを分ける戦略も検討したい．

なおスライド資料作成の文脈で言えば，個人的には [Marp](https://github.com/marp-team/marpit) をいちばん使っている．UI/UX のシンプルさと出力の美しさ，少しデザインを凝りたくなったときに必要な手間のバランスがとてもよい．バージョン管理が容易な点や，$\LaTeX$ や HTML が使える点も嬉しい．もっと流行ってほしい．

### Misc
- テキストの敬体と常体はなるべく統一する
- テキストの用言止めと体言止めをなるべく統一する
  - e.g.「～を削除」／「～を削除する」

## データ可視化
- Nussbaumer Knaflic, C. (2015). [**Storytelling with data**](http://www.storytellingwithdata.com/) (C. N. Knaflic, Ed.). John Wiley & Sons.
  - 神資料２．以前は上記リンクからダウンロードできたが，いまはどうか分からない
  - （邦題の『Google 流 資料作成術』はウケ狙いが過ぎる印象で，一部の邦訳も微妙だった）
- [Stanford CS course on data visualization techniques (Winter 2020)](https://magrawala.github.io/cs448b-wi20/)
- [可視化や統計でデータに『恣意的なストーリーを語らせる』16の闇の魔術【bad charts】](https://qiita.com/Ringa_hyj/items/a938ec99d0fde052837e)
- [OpenCVでのデモの見栄えを工夫したまとめ(ディープラーニング系)](https://qiita.com/Kazuhito/items/f8f21956436dc410f8d2)

## フォント
基本的に UD（ユニバーサルデザイン）フォントを使うべき．読字に困難のあるロービジョンやディスレクシアの負担を軽減できる．

[ユニバーサルデザインを意識したスライドづくりのポイント③「UDフォントの使い方」](https://note.com/tep_kikuchi/n/n43ce66c33870) が分かりやすかった．ユニバーサルデザイン一般を学ぶ上で，菊池先生のほかの note も非常に勉強になる．

### UD デジタル教科書体
Windows にデフォルトで入っている UD フォント．アカデミアの人がつくる資料で比較的よく見かける印象．

文字詰めに関して以下の 3 種がある：
- N: 等幅
- NP: 英数字がプロポーショナル
- NK: 英数字＋かな文字がプロポーショナル

### MORISAWA BIZ UD
教科書体よりビジネス向けのフォント．

- [フォントが大好物な人に朗報🎉 MORISAWA BIZ UDゴシックとUD明朝がオープンソースになったぞ！！](https://coliss.com/articles/build-websites/operation/work/googlefonts-morisawa-biz-ud-gothic-and-mincho.html)

オープンソースになったため，ほかのフォントと自在に組み合わせられる．私はコーディングに JetBrains Mono と合成した [UDEV Gothic](https://github.com/yuru7/udev-gothic/) を使っている．

UDEV Gothic を使っていて個人的に嬉しいのはリガチャの存在で，例えば `<=` が `≼` になる．ただし一部のリガチャは，環境によっては効かなかったりした．

## 話しかた
- [非エンジニアサイドに技術的負債や設計を説明するノウハウ](https://qiita.com/MinoDriven/items/2d63dcaa92b50b049889)
- David Tong, [How to Make Sure Your Talk Doesn't Suck](http://www.damtp.cam.ac.uk/user/tong/talks/talk.pdf)
- [研究発表は【道案内ゲーム】- 聞き手を迷子にさせないための９ルール](https://note.com/hisashi_is/n/nab735f3cb4f8)