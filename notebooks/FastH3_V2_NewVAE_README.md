# FastH3 V2＋新INT8 VAE版

元のFastH3 Colab Notebookを基にした追加版です。**8ステップのV2モデル**と、映像を変換する**INT8 VAE**をセットで使います。
元の4ステップ版とは別Notebookです。元版のランタイムを流用せず、新しいランタイムで開始してください。

- ComfyUI v0.36.0を固定。暫定SolAttnMiniMaxノードと旧専用wheelを、標準BlockSparseAttention＋comfy-kitchen 0.2.34に置換。
- 動画shift 10／音声shift 3／Euler／8ステップ／VSA保持率20%。同梱4ワークフローとアプリ表示にも反映。
- GPUはL4 / A100 / H100 / G4系など。モデル合計は約42GB。モデル保存と作業領域には余裕を持たせてください。
- まず0.4MP・短尺で開始。高解像度、長尺、多数の参照は必要メモリと時間が増えます。
- ローカルRTX 4090では1344×768・8秒・参照7枚で成功。**このNotebookのColab L4/A100実機での生成テストは未実施**です。ローカルの結果はColabの速度・品質保証ではありません。
- V2は公式にはT2V（文章から動画）が中心。I2V（開始画像）／R2V（参照画像）は実験的で、画質と演出の一致は別に評価してください。
- 元NotebookのPinggy接続、Colabシークレット、Drive出力、サンプル画像、4種類の操作画面を引き継いでいます。

制作：ざすこ（道草 雑草子）／[AIみちくさch](https://www.youtube.com/channel/UC84fyKjiilxssZVxhE_RiaA)

# モデル・設定の出所

| 部品 | この版の設定 |
|---|---|
| メインモデル | `fastvideo_fasth3_8step_v2_pruned_int8_convrot.safetensors` |
| 映像VAE | `minimax_h3_video_vae_int8_convrot.safetensors` |
| 文章・参照の理解 | `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` |
| 音声VAE | `minimax_h3_audio_vae_fp32.safetensors` |
| ComfyUI | v0.36.0 / `ee71d5c4993f29086b27fde1629a945ae48425bf` |
| PyTorch | 2.9.0 / CUDA 13.0（元Notebookの組み合わせを維持） |
| 高速化 | comfy-kitchen 0.2.34、標準BlockSparseAttention、VSA保持率20% |
| ManualSigmas | `0.9998999099, 0.9857884051, 0.9675752487, 0.9431680774, 0.9090909091, 0.8571428571, 0.7692307692, 0.5882352941, 0` |

- [V2公式モデルカード](https://huggingface.co/FastVideo/FastVideo-FastH3-8-Step-V2)
- [ComfyUI用V2配布](https://huggingface.co/FastVideo/FastVideo-FastH3-Comfy)
- [VAE・テキストエンコーダー配布](https://huggingface.co/Comfy-Org/MiniMax-H3)
- [FastVideo公式設定](https://haoailab.com/FastVideo/inference/fasth3-distilled/)
- [ComfyUI v0.36.0](https://github.com/Comfy-Org/ComfyUI/tree/ee71d5c4993f29086b27fde1629a945ae48425bf)
- [モデルのMiniMax H3 Community License](https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/LICENSE)

モデルは配布元から直接ダウンロードし、このGitHubリポジトリには再配布しません。利用地域・用途など、モデル配布元の条件を確認してください。ComfyUIはGPL-3.0、comfy-kitchenはApache-2.0です。旧版の専用wheelはこの追加版では使いません。

2026-09-20：Notebook形式・Python構文・全ワークフローの接続とV2設定をローカルで確認。Colabでのインストール・トンネル接続・生成の実機試験は未実施です。

## H100・G4系への対応（2026-09-20追記）

L4/A100だけを許す名前判定を廃止し、実際のGPUの計算世代（Compute Capability 8.0以上）と搭載メモリ（約24GB以上を想定、22GiB未満は停止）で確認します。H100（9.0）とRTX PRO 6000 Blackwell（12.0、Google CloudのG4系）も、この条件を満たせば先へ進みます。T4（7.5）は停止します。

起動時に実GPU名・メモリ・CUDA情報を表示し、PyTorchの小さな行列計算とcomfy-kitchenのCUDA sparse attention計算を実行します。失敗時はモデルを大量ダウンロードする前に止めます。GPU名だけを追加して計算できたことにする実装ではありません。

24GB級では従来の省メモリ起動設定を使用し、より大きなメモリのGPUでは標準起動にします。解像度や長さを勝手に増やしません。G4/H100の割り当てはColab側の提供状況によります。

**H100/G4上でのNotebook全体の実機生成テストは未実施です。** 機種ごとのCUDAドライバや配布ライブラリの組み合わせも起動時テストで確認してください。この確認を通っても、長尺・高解像度のメモリ不足や全モデルの互換性までは保証しません。

出典：
- [NVIDIA GPU計算世代一覧](https://developer.nvidia.com/cuda/gpus)
- [Google Cloud G4の仕様](https://docs.cloud.google.com/compute/docs/accelerator-optimized-machines)
- [comfy-kitchen 0.2.34](https://github.com/Comfy-Org/comfy-kitchen/tree/v0.2.34)
- [Colab公式FAQ](https://research.google.com/colaboratory/faq.html)
