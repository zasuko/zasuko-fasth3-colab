# FastH3 V2＋新VAE アプリ版ワークフロー

`FastH3_V2_NewVAE_R2V_App.json`をComfyUIへドラッグ＆ドロップするか、Workflowの「開く」から読み込みます。API形式ではなく、画面から読み込むUI形式です。

## 操作

1. V2＋新VAEのColab Notebookを起動します。
2. このJSONをComfyUIに読み込みます。アプリ表示が出ない場合はアプリモードへ切り替えます。
3. 縦横比、解像度、長さ、参照画像1、プロンプトを指定して実行します。
4. 初期値は0.4MP・5秒。1.0MPも選べます。参照素材に権利を持つ画像を使用してください。

解像度は概算で32の倍数へ丸めます。16:9の1.0MPは1376×768です。先のT288実験の1344×768とは少し異なります。フレーム数の丸めにより5秒指定は約5.17秒になる場合があります。

## 引き継いだ機能と修正

- 元NotebookのR2Vアプリ版をベースに、レイアウト・縦横比・長さ上限15秒・保存名の自動作成を維持。
- メインモデルをV2、映像VAEをINT8版、8ステップ・shift動画10/音声3・VSA保持率20%へ更新。
- 名前付きの保存設定も同期し、旧設定の復活を防止。
- 文字化けしたラベルを修正。アプリ入力欄の存在しない旧ノード番号を除去。
- 解像度選択に1.0MPを追加。初期値の選択文字と内部番号を一致させた。
- アプリ画面には実際に接続されている参照画像1を表示。予備画像3枚・動画・音声はノード画面に残し、「未接続」と明記。使用時だけ接続する。
- V1と取り違えないよう、この版に固有のワークフローIDを割り当て。

## 必要なもの

ComfyUI v0.36.0、comfy-kitchen 0.2.34。暫定SolAttnMiniMaxノードは不要です。

- diffusion_models: `fastvideo_fasth3_8step_v2_pruned_int8_convrot.safetensors`
- text_encoders: `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`
- vae: `minimax_h3_video_vae_int8_convrot.safetensors` と `minimax_h3_audio_vae_fp32.safetensors`

[Notebookと起動リンク](https://github.com/zasuko/zasuko-fasth3-colab)
[モデル配布元](https://huggingface.co/FastVideo/FastVideo-FastH3-Comfy)

同じV2＋新VAEのAPI構成はローカルRTX 4090とColab G4で生成済みです。本アプリ版はColabのComfyUIで読み込み、アプリ表示・入力6項目・解像度5種類を確認済みです。この見本プロンプトでの生成試験は未実施です。モデルの地域・用途等の条件は[MiniMax H3ライセンス](https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/LICENSE)を参照してください。
