# zasuko-fasth3-colab

MiniMax H3 の高速版 **FastH3**（4ステップ蒸留＋VSA）を Google Colab で動かすための Notebook 一式です。

AIみちくさチャンネルのメンバーシップ特典として配布している「ざすこ式 MiniMax H3 Notebook」を、FastH3 対応に改造したものです。


## ▶ Colabで開く

| Notebook | 必要なDrive容量 | 開く |
| --- | --- | --- |
| **Driveレス版**（毎回ダウンロード） | **不要** | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/zasuko/zasuko-fasth3-colab/blob/main/notebooks/FastH3_Colab_Driveless.ipynb) |
| **Drive常駐版**（2回目以降が速い） | **44GB以上** | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/zasuko/zasuko-fasth3-colab/blob/main/notebooks/FastH3_Colab_DriveSaved.ipynb) |

> 開く前に、下の「⚠️ 最初に読んでください」を必ず確認してください。
> **GPUは L4 か A100 を選ぶ必要があります**（無料枠のT4では高速化が効きません）。

---

## ⚠️ 最初に読んでください

### これは実験段階の構成です

FastH3 の実行に必要な次の3点は、**2026年9月1日時点でまだ正式リリースされていません**。

| 必要なもの | 状態 |
| --- | --- |
| ComfyUI 本体の VSA 対応（[PR #15958](https://github.com/Comfy-Org/ComfyUI/pull/15958)） | **未マージ**（kijai 氏の作業ブランチを使用） |
| comfy-kitchen の Sol-Attention（[PR #117](https://github.com/Comfy-Org/comfy-kitchen/pull/117)） | マージ済みだが **PyPI 未反映** |
| カスタムノード `sol_attn_minimax_v5.py` | **PR に添付された暫定ファイル** |

つまり **動作保証はありません**。正式マージ後はこの Notebook は不要になります。上流が更新されると突然動かなくなることがあります。

### GPU の条件（重要）

Sol-Attention のカーネルは **Compute Capability 8.0 以上（sm_80+）** を要求します。

| Colab の GPU | 使えるか | 備考 |
| --- | --- | --- |
| **T4**（無料枠で割り当てられやすい） | ❌ **使えません** | sm_75。高速化が効かず通常計算に落ちます |
| **L4** | ✅ **推奨** | sm_89。VRAM 24GB |
| **A100** | ✅ 使える | sm_80。**VRAM 80GB**（Colabで確認）。長尺・高解像度に有利 |

**無料枠の T4 では高速化の恩恵を受けられません。** Colab Pro 以上で L4 か A100 を選んでください。

### 必要な容量

FastH3 はモデルの合計が **約 44GB** あります。

| Notebook | Google Drive の空き | 起動のたびの待ち時間 |
| --- | --- | --- |
| **Driveレス版**（毎回ダウンロード） | **不要** | 毎回 44GB をダウンロード |
| **Drive常駐版** | **44GB 以上必要** | 2回目以降は Drive から読むだけ |

> **Drive常駐版は、Google ドライブの無料枠（15GB）では使えません。** Google One 100GB 以上のプランが必要です。

---

## 使い方

1. 上の **「Colabで開く」** のバッジを押す（`notebooks/` の `.ipynb` を直接開いても同じです）
2. Colab で「ランタイムのタイプを変更」→ **L4 または A100** を選ぶ
3. セルを上から順に実行する
4. 実行ログに出る青い「ComfyUIを開く」ボタンをクリックする

---

## FastH3 で気をつけること

実際に RTX 4090 で検証した結果からの注意点です。

### 1. 解像度 × 秒数には上限があります

```text
メガピクセル × 秒 ≦ 約16   （VRAM 24GB = L4 の場合）
メガピクセル × 秒 ≦ 30以上 （VRAM 80GB = A100。実測で30.1まで確認）
```

**L4（24GB）の場合**

| | 5秒 | 8秒 | 15秒 |
| --- | --- | --- | --- |
| **768p**（1376×768 / 1.06MP） | ✅ | ✅ | ✅ |
| **1080p**（1920×1088 / 2.09MP） | ✅ | ✅ | ❌ **落ちます** |

**A100（80GB）の実測**

| 条件 | 総時間 |
| --- | --- |
| 1376×768（1.06MP）× 15秒 | **3分44秒** |
| 1632×928（1.51MP）× 15秒 | **6分54秒** |
| 1888×1056（1.99MP）× 15秒 | **9分50秒** |

超えると、プロセスが落ちるか、GPU使用率100%のまま永久に終わらなくなります。**GPUの消費電力が100W前後で張り付いていたら、それは計算ではなくメモリの往復です。** 諦めて条件を下げてください。


### 2. 幅と高さは 32 の倍数にしてください

1080p は **1920 × 1088** です（1080 は 32 で割り切れません）。

### 3. ステップ数を変えないでください

FastH3 は **4ステップ専用**に訓練されています。増やしても良くなりません。

`ManualSigmas` の値も固定です: `0.9999166, 0.9728326, 0.9230769, 0.8, 0.0`

### 4. キャラクターを似せる（R2V）

**FastH3（4ステップ）でも、R2Vの参照画像はしっかり効きます。** 髪・眼鏡・服の色まで追従しました（2026-09-02 実測）。

ただし**最大の落とし穴**が1つあります。

> **参照画像がつながっていなくても、ComfyUIはエラーを出さずに生成を完了します。**

`ref_images` は入れ物を増やせる形の入力で、**空のまま実行しても止まりません**。
このため「4ステップだと参照が効かない」と丸一日ぶん誤解した経緯があります。

**回す前の確認**:

1. `LoadImage` から `MiniMaxH3ReferenceToVideo` へ**線が伸びているか**を目で見る
2. 迷ったら、**参照を外した1本と見比べる**。見分けがつかなければ届いていません

**外見はプロンプトにも書いておく**と安定します（参照の有無にかかわらず有効な習慣）。

**まだ分かっていないこと**: 参照は何枚が最適か（上限は9枚）、20ステップ版との品質差。
2026-09-01 に書いていた「尺が短いと似ない」「1カット1.5秒以上必要」という記述は、
**参照が届いていない状態での観測だったので取り下げました**。

R2V の参照画像は、**見出し文字入りのキャラクターシートでも問題ありません**。

### それでも顔が決まらないときは I2V

**I2V（画像から動画）** なら、1フレーム目に置いた画像がそのまま出力の1フレーム目になるため、**顔が構造的に固定されます**。

**I2V の起点画像の条件**（R2V の参照画像には当てはまりません）:

- キャラクター1体だけが写っていること
- **文字・見出し・枠線・複数ポーズが入っていないこと**
  （文字入りのシートを I2V の起点にすると、見出しが最後まで画面に焼き付きます）

### 5. プロンプトの書き方

- **外見は文章でも必ず書く**。参照画像任せにしない
- **背景の装飾語がキャラクターの衣装に漏れます**。背景に `ribbons` や `chrome` と書くと、腰に金の飾りが生えたりします
- **禁止形は効きません**。`Avoid melting anatomy` ではなく `anatomy stays clean and correct` のように**肯定形で書く**
- **カットを詰め込みすぎない**。1カット 1.5秒以上を目安に

### 6. VSA が効いているかログで確認してください

生成中のログに次の行が出ていれば成功です。

```text
[sol_attn] producer path: 76588 tokens, VSA tiles (87552 padded rows, 18 prefix tiles), topk=0.100
```

`VSA tiles` と `topk=0.100` が出ていなければ、間引きが効かず通常の計算に落ちています。

---

## 速度の目安

すべてログから拾った実測値です。

**RTX 4090（VRAM 24GB・ローカル）**

| 条件 | FastH3（4ステップ） |
| --- | --- |
| 1920×1088 × 8秒 | **約3〜4分** |
| 1376×768 × 15秒 | **約3分45秒** |
| 1920×1088 × 5秒 | **約2〜3分** |

**Colab A100 80GB**

| 条件 | 1本目（コールド） | 2本目（ウォーム） |
| --- | --- | --- |
| 1088×1920（2.09MP）× 5.17秒 | **132.35秒** | **118.83秒** |
| 1376×768（1.06MP）× 15.08秒 | **235.42秒** | **223.90秒** |
| 1632×928（1.51MP）× 15.08秒 | **413.89秒** | — |
| 1888×1056（1.99MP）× 15.08秒 | **589.63秒** | — |

セッションを終了せずに連続で生成すると、**2本目以降は 5〜10% 速くなります**。

**解像度を上げたときの伸び方**（尺15秒で固定した3点）: 画素が 1.42倍 → 時間 1.96倍、
画素が 1.88倍 → 時間 2.90倍。**おおよそ「画素数の1.5乗」**で、
通常のAttention（2乗）よりは緩やかですが、線形でもありません。

参考として、同系統の条件を **Colab A100 の通常版（20ステップ）** で回したときは、1.5MP × 8秒で **25分35秒** かかりました。FastH3はその**10分の1以下**です。

---

## ライセンスと再配布について

### この Notebook で使うもの

| 対象 | ライセンス | 扱い |
| --- | --- | --- |
| MiniMax H3 の重み | [MiniMax H3 Community License](https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/LICENSE) | **日本は適用地域**。米・EU・英・韓は除外地域。商用UIには `MiniMax H3` の表示が必要 |
| ComfyUI | GPL-3.0 | Notebook 内でクローン |
| comfy-kitchen | Apache-2.0 | **本リポジトリの [Releases](https://github.com/zasuko/zasuko-fasth3-colab/releases/tag/comfy-kitchen-solattn) で再配布**しています。ライセンス全文と出所は [`third_party/comfy-kitchen/`](third_party/comfy-kitchen/) にあります |
| FastH3 の重み（[Kijai/MiniMax-H3-experimental](https://huggingface.co/Kijai/MiniMax-H3-experimental)） | **表記なし** | Hugging Face から直接ダウンロード。**再配布しません** |
| `sol_attn_minimax_v5.py` | **表記なし** | PR の添付ファイルから直接ダウンロード。**再配布しません** |
| ワークフロー JSON | **表記なし** | [sepiablue-ai/minimax_h3_workflows](https://github.com/sepiablue-ai/minimax_h3_workflows) から直接ダウンロード。**再配布しません** |

**ライセンス表記が無いものは再配布せず、Notebook から公式の配布元へ取りに行く形にしています。**

### 生成した動画について

- 生成物の権利は MiniMax が主張しませんが、**入力素材の権利と各サービスの規約は利用者の責任**です
- 収益化する動画に使う場合は、**`MiniMax H3` のクレジット表示**をおすすめします
- 年間売上 2,000万米ドルを超える商用利用には、MiniMax の書面による許可が必要です

---

## 参考にした情報

- [FastH3 導入ガイド（Kamimoto 氏 / note・2026-08-31）](https://note.com/sepiablue/n/n99905bcb9c8b) — 本 Notebook の元になった手順
- [Kijai/MiniMax-H3-experimental](https://huggingface.co/Kijai/MiniMax-H3-experimental) — FastH3 の配布元
- [Comfy-Org/comfy-kitchen PR #117](https://github.com/Comfy-Org/comfy-kitchen/pull/117) — Sol-Attention の実装
- [Comfy-Org/ComfyUI PR #15958](https://github.com/Comfy-Org/ComfyUI/pull/15958) — ComfyUI 側の VSA 対応
- [sepiablue-ai/minimax_h3_workflows](https://github.com/sepiablue-ai/minimax_h3_workflows) — サンプルワークフロー

---

## 免責

この Notebook は個人の検証記録をもとにしたものです。**動作保証はありません。**
未マージの開発版に依存しているため、上流の変更で予告なく動作しなくなる可能性があります。
正式版が公開されたら、この構成は不要になります。

普段お使いの ComfyUI 環境にこの構成をインストールしないでください。壊れます。
