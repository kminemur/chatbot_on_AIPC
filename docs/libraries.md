# Libraries

このプロジェクトの標準モデルは `OpenVINO/gemma-4-E4B-it-int8-ov` です。このモデルは OpenVINO IR 形式の INT8 圧縮済みモデルですが、実行経路は `openvino-genai` ではなく `Optimum Intel` と `transformers` を使います。

## Runtime Stack
`OpenVINO/gemma-4-E4B-it-int8-ov` は Hugging Face のモデルカード上で experimental model とされています。2026-06-02 時点のモデルカードでは、OpenVINO 2026.1.0 以上、Optimum Intel 1.27.0 以上に互換とされていますが、Gemma 4 対応には `rkazants/optimum-intel` の `support_gemma_4` ブランチと `transformers==5.5.0` が指定されています。

このため、`pyproject.toml` では次を前提にします。

- `openvino>=2026.1.0`
- `optimum-intel @ git+https://github.com/rkazants/optimum-intel.git@support_gemma_4`
- `transformers==5.5.0`
- `torch`
- `torchvision`
- `Pillow`
- `huggingface_hub>=1.5.0,<2.0.0`

`support_gemma_4` ブランチの package metadata は `transformers <5.1` のままですが、モデルカードの実行手順は `transformers==5.5.0` を指定しています。その差分を uv に明示するため、`pyproject.toml` の `[tool.uv]` で `override-dependencies = ["transformers==5.5.0"]` を設定します。

`openvino-genai` はこのモデルの標準実行経路には使いません。`LLMPipeline` 前提のコードやドキュメントは追加しないでください。

## Library Roles
`openvino`:
OpenVINO runtime 本体です。OpenVINO IR のロードと Intel GPU/CPU での実行に使います。

`optimum-intel`:
OpenVINO 形式の Hugging Face モデルを Transformers 互換 API で扱うために使います。このモデルでは `OVModelForVisualCausalLM` を使う想定です。

`transformers`:
`AutoProcessor`、chat template、tokenize/decode、`generate()` の周辺 API に使います。Gemma 4 対応のため `5.5.0` に固定します。

`torch`, `torchvision`, `Pillow`:
processor 入出力と visual causal LM 系の依存として必要です。今の UI はテキストチャットを入口にしますが、モデル種別は image-text-to-text なので画像処理系ライブラリを外さないでください。

`huggingface_hub`:
`uv run python download.py` 経由で Hugging Face repo から OpenVINO ファイルを取得するために使います。

## Inference Contract
推論ラッパは次の形を基本にします。

```python
from optimum.intel.openvino import OVModelForVisualCausalLM
from transformers import AutoProcessor

model_id_or_dir = "model"
processor = AutoProcessor.from_pretrained(model_id_or_dir)
model = OVModelForVisualCausalLM.from_pretrained(model_id_or_dir)
```

テキストチャットでは、ユーザーメッセージを chat template に通してから `processor` と `model.generate()` に渡します。API/UI には `processor.decode(..., skip_special_tokens=True)` で得たユーザー向けテキストだけを返します。

## Setup Notes
`uv sync` は GitHub から `optimum-intel` の対応ブランチを取得します。ネットワークに出られない環境では依存同期に失敗します。

PyTorch の取得で環境によって CPU wheel の index が必要になる場合があります。その場合は一度だけ次を試します。

```bat
uv pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/cpu
uv sync
```

モデルカードの experimental 指定が外れ、公開版 `optimum-intel` と `transformers` で Gemma 4 が動くようになった場合は、`pyproject.toml` とこの文書を同じ変更で更新します。
