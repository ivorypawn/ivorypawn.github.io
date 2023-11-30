---
title: "Unity 覚書"
draft: true
tags: ["Unity"]
---
Unity 備忘録を集約．初歩的な内容が中心．環境は Windows 10 + Visual Studio 2022．

## 資料
適宜追加．
### オンライン
- [Unity ユーザーマニュアル（日本語）](https://docs.unity3d.com/ja/current/Manual/index.html)
- [C# プログラミング ガイド](https://learn.microsoft.com/ja-jp/dotnet/csharp/programming-guide/)
- [Unity 入門の森](https://3dunity.org/)
- [入門 Unity @KINDAI Info-Tech HUB](https://zenn.dev/sane21/books/b42ecebc457eb6/viewer/cf8cdb)
- [Unity 開発に関する 50 の Tips 〜ベストプラクティス〜 (2016 Edition)](https://zenn.dev/sh1ch/books/4f49e00cfd92be082e1c/viewer/8a5619780235f1bc3f74)
- [(非公式和訳）Unity - level up your code with game programming patterns](https://zenn.dev/twugo/books/21cb3a6515e7b8)
- [Unity「2Dゲーム開発全般をカバーする本（英語）」を日本語で読む](https://zenn.dev/sh1ch/books/35fbffce89f26b)

### 書籍
- Harrison Ferrone / 吉川 邦夫 訳 (2021)『[Unity 3D ゲーム開発ではじめる C# プログラミング](https://book.impress.co.jp/books/1120101145)』
- 北村 愛実 (2016)『[Unity 5 の教科書](https://www.sbcr.jp/product/4797386790/)』
- 赤坂 玲音 (2015)『[エンジニアのための Unity 実践リファレンス](https://gihyo.jp/book/2015/978-4-7741-7303-0)』
- 岩井 雅幸 (2015)『[uGUI ではじめる Unity UI デザインの教科書](https://book.mynavi.jp/supportsite/detail/9784839956400.html)』

## 導入
1. [Unity Hub](https://unity.com/ja/download) インストール
   - 2019 年からバージョン管理ソフトになった．Hub から適宜旧バージョンを入れられる
2. Unity 2022 インストール
3. 歯車 > `Add modules`

## Unity Editor
おもな GUI 構成要素：
- Toolbar（上段のリボン状のパネル）
- Hierarchy（左カラム）
- Scene, Game（中央）
- Inspector（右カラム）
- Project, Console（下段）

### レベル構築
- アウトドア環境
  - [Terrain](https://docs.unity3d.com/ja/2021.3/Manual/script-Terrain.html)
- インドア環境
  - shape, geometry
- インポート（Blender など）

### Material
任意の GameObject はマテリアルとシェーダーをもつ．

マテリアルは，GameObject がどうレンダリングされるかを決定する．シェーダーはライティングとテクスチャデータの組み合わせからなり，マテリアルを制御する．

### (Misc)
- 原則 Unity エディタ以外からファイル操作するとバグる可能性があるのでやらない
- プロジェクトを zip 圧縮しない
  - meta が壊れる
  - 7z ならオーケーらしい
- Inspector のコンポーネント横の情報アイコンからマニュアルに飛べる
- コンポーネントの `public` 変数は Inspector で一時変更できる（`private` 変数は無理）．この変更はスクリプトを書き換えない．再生中の変更は停止するともとに戻る
  - Inspector 上でスクリプトの初期値に戻すこともできる(`Reset`)
- 同じディレクトリで定義したクラスは，アタッチや `using` なしに参照される．これらのクラスと同様，デフォルトで入っている `Transform`, `Camera` などのコンポーネントは，再生時，メンバやメソッドをもつオブジェクトとしてメモリ上に展開される

- Unity の 2D は 3D シーンを平面に投影したもの．エディタの基本操作はほぼ共通
  - 3D で Y 軸が上向きなのはそのため
- シーンビューに配置した画像は **スプライト** と呼ばれる
- 最初にスケールを決定して，すべて同じスケールになるようにビルドする
  - 3D なら $1\\,\mathrm{unity\ unit} = 1\\,\mathrm{m}$，2D なら $1\\,\mathrm{unity\ unit} = 1\\,\mathrm{pixel}$ など（[参照](https://zenn.dev/sh1ch/books/4f49e00cfd92be082e1c/viewer/a590091472434f75e0a1)）

## Visual Studio
エディタで適当に C# スクリプトを作ってダブルクリックし Visual Studio で開く．何も起こらなければ `Edit` > `Preference` > `External Tools` > `External Script Editor` から設定（これを忘れると入力補完が使えなかったりした）．

### (Misc)
- ファイル名とクラス名は一致させる．VS で勝手に変えるとたぶんバグる
  - 変更は Project から
  - 基底の [`MonoBehaviour`](https://docs.unity3d.com/ja/current/Manual/class-MonoBehaviour.html) は「シーン中のオブジェクトにアタッチできるクラス」の謂
- エディタと VS が同期しない
  - Project から当該ファイルを `Refresh`
  - `Edit` > `Preference` > `Asset Pipeline` から `Auto Refresh` にすることもできる
    - エディタのフォーカス時に自動コンパイルが走る
- メソッドの真上の行で `///` を入力すると `<summary>` タグの枠が自動補完される

## アタッチしたオブジェクトの確認
途中から参加するプロジェクトなど，どのコンポーネントがどのオブジェクトにアタッチされているのかわからないとき，調べる方法はおもに二つ．
- Project のスクリプトを右クリック > `Find Reference In Scene` で，Hierarchy がアタッチされているシーンに絞り込まれる
  - ヒエラルキーの検索欄をクリアするともとに戻る
  - オブジェクトの検索はできないので，インデックスを作成して検索するを導入するか，調べたいリソースの meta ファイル内の guid でプロジェクト全体を `find` する（Linux の `grep`）
- `Start()` に `Debug.Log($"{name}")` を挿入して実行

## C++ との違い
C++ との違いを適宜メモしていく．

### クラスと構造体
`class` は参照型，`struct` は値型になる．また `string` などが参照型になる．
```c#
public class Character
{
    public string name;
    public Character(string name) { this.name = name; }
    public void print() { Debug.LogFormat("name: {0}", name); }
}

void Start()
{
    var taro = new Character("taro");
    taro.print(); // taro
    var jiro = taro;
    jiro.name = "jiro";
    taro.print(); // jiro
}
```

## 雑に作る
もろもろを『Unity 5 の教科書』のゲームを作りながら覚える．

### 一般的なフロー
1. 登場するオブジェクトを列挙する
2. 動かすオブジェクトの **コントローラスクリプト** を書く
3. オブジェクトを自動生成する **ジェネレータスクリプト** を書く
4. UI を更新する **監督スクリプト** を書く
5. スクリプトを作る流れを考える

### Roulette
ルーレットを作る．フロー 3., 4. は不要．

コントローラスクリプトに，タップ（クリック）すると回転を始めて，徐々に減速する動作を記述する．

- `Input.GetMouseButtonDown(0)`
  - 引数 0 が左クリック，1 が右クリック，2 が中央クリックに対応
- `transform.Rotate(x, y, z)`
  - 引数が X, Y, Z 軸の回転量に対応

### SwipeCar
フラグに車を寸止めするゲーム．監督スクリプトを作成する．

- スワイプ距離の取得，オブジェクト速度への適用
  - `Input.GetMouseButtonUp(0)`
  - `Input.mousePosition.x`
  - `transform.Translate(x, 0, 0)`
    - 元位置を原点とするローカル座標
- uGUI
  - `using UnityEngine.UI`
    - `using UnityEngine` とは別に必要
    - `(GameObject).GetComponent<Text>()` などが使えるようになる
- `GameObject.Find()`
- AudioSource コンポーネント

UIText を追加すると UI 設計画面が作成される．UI 設計画面はゲームの設計画面よりはるかに大きいが，実行時はゲーム画面に対応する．

`EventSystem` はユーザの入力を UI 部品に中継する．入力の割り当て，無効化，キーやマウスの設定変更などが可能．

監督スクリプトは空のオブジェクトに渡す．

#### `GetComponent`
コンポーネントは `GameObject` の一つの性質，あるいは `GameObject` という箱に追加して機能実現する「設定資料」のようなもの．コンポーネントへのアクセスは，原則テンプレート関数 `GetComponent` を用いる．

- 座標・回転：`GetComponent<Transform>()`
- 物理挙動：`GetComponent<Rigidbody>()`
- 効果音：`GetComponent<AudioSource>()`
- UI テキスト：`GetComponent<Text>()`
- スクリプト：`car.GetComponent<CarController>()` etc.

ただし座標取得などでよく使う `Transform` に限り，簡易的な `car.transform.position.x` のような書き方ができる．

### CatEscape
避けゲー．ジェネレータスクリプトで Prefab を生成する．

- `Input.GetKeyDown(KeyCode.LeftArrow)` etc.
- `Vector2.magnitude`
- `Destroy(gameObject)`
  - `gameObject` は自身を指す 
  - 一時停止してヒエラルキービューを見ると実際にオブジェクトが消えている
- キャスト e.g. `Instantiate(Prefab) as GameObject`
- アンカーポイントによる画像配置
  - Play Mode 中でも編集が適用された

#### (Misc)
- サンプルコードは干渉時に毎回 `Find()` を呼んでいるが，たぶん `Start()` で初期化するほうが効率がよい
- 旧 ver. Unity Editor で作業中，設定等に問題がないのに `UnityEngine.UI` の画像生成で `MissingComponentException` が出る不具合に遭遇した．旧 ver. でたまに起きるらしい．最新 ver. では普通に動いた．再インストールで直った報告も見かけた

#### `Destroy` の挙動
サンプルコードは `Update()` のなかで `Destroy(gameObject)` を呼んだあと自身のメンバを参照する可能性があり，一見して危険に思えた．しかし `Destroy(gameObject)` の一行後に `Debug.Log(gameObject.transform.position)` としても，普通に直前の座標が返ってきた．

- [Object.Destroy（公式ドキュメント）](https://docs.unity3d.com/ScriptReference/Object.Destroy.html)
- [Unityにおけるnullチェック(1) - AがnullでなくともA == nullがtrueになる](https://qiita.com/msm2001/items/b57f6b92371026307c0b)
- [Unityにおけるnullチェック(2) - Destroyが何をしているか](https://qiita.com/msm2001/items/69a5ac3b1491723e756b)

上記によると，`Destroy` によってオブジェクトが破棄されるのは `Update` ループのあと．[`yield return`](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/statements/yield) して 1 フレーム待つと例外が返ってくる．

なお即座に破棄する `DestroyImmediate` メソッドもあるが，ランタイムでの使用が推奨されていないらしい．

#### Prefab $\fallingdotseq$ 設計図
ヒエラルキービュー上のオブジェクトをプロジェクトビューにドラッグ & ドロップすると Prefab ができる．一度 Prefab を作れば，シーン中のオブジェクトは削除しても構わない．

シーン中のプレハブは 1 つ 1 つが別個のインスタンスになる．

ブレハブに新しいオブジェクトを追加するときは，同じヒエラルキーにそのオブジェクトを作成し，右クリックから `Added GameObject` > `プレハブ'xxx'に適用`．

#### アウトレット接続
スクリプトにアウトレットを用意し，Inspector からアウトレットを経由してオブジェクトを代入することをアウトレット接続という．なお Outlet = コンセント（米英語）．

この例では `ArrowGenerator` のメンバに `public GameObject arrowPrefab` を用意し，インスペクタ上で Prefab をドラッグ & ドロップして紐づけしている．

### ClimbCloud
Jump King 的な．Physics チュートリアル．

- 剛体：`Add Component` > `Physics 2D` > `Rigidbody 2D`
  - デフォルトで重力や外力の影響を受ける．`Body Type` > `static` でビューに固定
- 当たり判定：`Add Component` > `Physics 2D` > `xxx Collider 2D`
  - 複数コライダでカプセル形状を作成
  - 下部が球状なので `Rigidbody 2D` > `Constraint` > `Freeze Rotation` で転倒防止

(Under Construction)

## .meta について
(Under Construction)
