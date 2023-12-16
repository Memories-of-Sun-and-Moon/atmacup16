# atmaCup #16 in collaboration with RECRUIT

![pic](https://media.connpass.com/thumbs/3d/82/3d82795d0f27c4b4e8336f4c9bc1e604.png)

# お題

リクルートのサービスである国内最大級の旅行予約サイト『じゃらんnet』の実データでセッション内の行動ログから、予約する宿を予測してもらいます（Session-Based Recommendation）

学習用データとしてある予約のセッションの様子とその予約結果が与えられますので、テストデータへの予測の際には、そのセッションの振る舞いや宿の属性等から、どの宿が予約されるか？を予測してください。

利用するデータや課題の詳しい説明・提出ファイル (submission file) の作成方法についてはデータタブから確認をしてください。

- Train / Testの分割方法: 時系列で分割されていて、TestはTrain期間より後に発生したセッションです。
- Public / Privateの分割方法: Testデータをランダムに分割しています。Public : 25% / Private : 75% です。

# 評価方法

MAP@10

$N$番目の予測が正しかった場合のスコアが$1/N$になる（1-indexed）

# データの説明

## train / test_log.csv

学習 / テスト用の期間に対するあるセッションごとに出現した宿を記録したログデータです。seq_noが見た順番を表します。

- session_id: セッションごとに割り振られたユニークなID.

### 補足

連続した同じ宿が出現する場合それらは1つの宿IDにまとめていることに注意してください。
セッション内で最後に出現する宿は必ず正解ラベルとは異なる宿となります。これは現在閲覧している宿とは違う宿をレコメンドしたいというニーズのためです。

## train_label.csv

学習データのセッションが最終的にどの宿を予約したかを記録したcsvファイルです。

- session_id: セッションごとに割り振られたユニークなID.
- yad_no: 予約した宿ID

## yado.csv

宿に関する属性情報が記録されたcsvファイルです。

yad_no: 宿ごとに割り振られたユニークなID
以下は宿の属性に関する情報です。

- yad_type : 宿泊種別（lodging type）
- total_room_cnt : 部屋数（total number of rooms）
- wireless_lan_flg : 無線LANがあるかどうか（wireless LAN connection）
- onsen_flg : 温泉を有しているかどうか（flag with hot spring）
- kd_stn_5min : 駅まで5分以内かどうか（within 5 minutes walk from the station）
- kd_bch_5min : ビーチまで5分以内かどうか（within 5 minutes walk to the beach）
- kd_slp_5min : ゲレンデまで5分以内かどうか（within 5 minutes walk to the slopes）
- kd_conv_walk_5min : コンビニまで5分以内かどうか（within 5 minutes walk to convenience store）

以下は宿の場所に関する情報です。「広域」から「小エリア」は階層構造になっています。（ex: 広域エリアには複数の県が紐づくが、1つの県に対しては1つの広域エリアのみが紐づく）

- wid_cd : 広域エリアCD（wide area CD）
- ken_cd : 県CD（prefecture CD）
- lrg_cd : 大エリアCD（large area CD）
- sml_cd : 小エリアCD（small area CD）

## image_embeddings.parquet

宿ごとに設定された画像の埋め込み結果です。

- yad_no: 宿No
- category: 画像の種類
- emb_0 ~ emb_511: 対応する画像埋め込み結果

## sample_submission.csv

サンプルの提出 (submission) ファイルです。今回のコンペティションではテストデータと同じ並び順に予測ラベルを10件紐付けて予測してください。

- predict_0 ~ predict_9: 予測した宿No.0がもっとも予測確度の高い宿でIndexの数字が大きくなるにつれて確度が下がるようにしてください。

## test_session.csv

提出用ファイルのセッションの並び順を表しているファイルです。test_log.csvに出現するすべてのsession_idを名前順にソートして作成されています。

予測ファイルを作成する場合にはこのセッションの並びと同じ様に予測ラベルを作成してください。

# 目標

- メダル圏内
- DataFrameには``polars``をつかう
    - いずれ流行するんだったら今から覚えよう
    - 特段pandasが手になじんでいる訳でもないし

# 日記

# 12/14

クッソ遅れたけど今から始める。

[はじめてのサブミット](https://github.com/Memories-of-Sun-and-Moon/atmacup16/issues/1)を読んだ。出現した宿をサブミットしたら``LB:0.2893``でたというもの。

train_log.csvなんだけど、$|seq|=1$のデータは、「宿Aを見た（$seq\_no = 0$）」→「宿Bを予約した」って感じなのかな。自信がない。

とにかくEDAをしてみることから始める。

[train/test_logをサクッとEDA](https://github.com/Memories-of-Sun-and-Moon/atmacup16/issues/3)

> train_log.csvなんだけど、$|seq|=1$のデータは、「宿Aを見た（$seq\_no = 0$）」→「宿Bを予約した」って感じなのかな。自信がない。

そういう認識でよさそう。

## やったこと

- 問題概要をここに写す
- MAP@10について深堀する
- 001_eda.ipynb

# 12/15

コンペ2日目です。後れを取りすぎているので、積極的に頑張りたいところ。

[#1 初心者向け講座 データと課題を理解してSubmitする!](https://github.com/Memories-of-Sun-and-Moon/atmacup16/issues/4)

をやった。

[#2 初心者向け講座 モデルを改善する](https://github.com/Memories-of-Sun-and-Moon/atmacup16/issues/5)

をみた。
- ``wid_cd`` が異なるものを候補集合に出すのは意味がなさそう
    - 99.5 % で一致している

- 1回も出現していない宿もある（そりゃそうか）

メモリ不足でjupyter notebookが落ちる。もうちょっと拡幅してあげねば。

24GBにしてきた。22.2GBくらい使って耐えた。

たえたけど、ときどきクラッシュするシーンが増えてきたから、オンプレ（言葉の使い方あってるかな）で機械学習をするのはあまりよくないのかもしれない。

LightGBMの直前までは実装できた。けれど、"session_id" ってどうやって扱えばいいのかな？

困ったからdropさせたのに、また復活したりと大変大変。

# 12/16

よくわからないまま時間だけが経つ。

やっとわかった。画像とがっちゃんこさせるときに、"max_session_yad_id"なるデータを混入させていたみたい。ちゃんと各データを追うべきだった。

## TODO

- スコア計算をする際は、seqの長さ毎にスコアがわかるようにするといいかもしれない

- ベースラインを基に提出できるようなものを作成
    - [ベースライン？](https://www.guruguru.science/competitions/22/discussions/15dda0cc-eefb-4125-8b1b-44ca26a86a13/)
    - [ベースライン](https://www.guruguru.science/competitions/22/discussions/3e9bfd60-2a43-452d-9f18-db37d20b77a1/)
    - [ベースライン](https://www.guruguru.science/competitions/22/discussions/20c54ca7-a389-43b0-9028-92011fb52fd5/)
    - [機械学習を使わないベースライン](https://www.guruguru.science/competitions/22/discussions/3ed58bd1-8d35-40e6-8202-db69ac858b3a/)

## 雑な方針検討メモ

- 宿の数が少ないならば、適当にその地域のオススメ順を提案する
- 行ったり来たりしているならば、最後に見ていない方を提案
    - 逆に、最後に見ている方は（性質上）確実に答えじゃないから省こう