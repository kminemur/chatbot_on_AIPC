# Setup

この文書は、現在の `uv sync` と `uv run python download.py` を再実装できる粒度の仕様です。

依存関係の同期は `uv sync`、モデル準備は `download.py` が担当します。`config.json` が指す OpenVINO モデル一式を `model.local_dir` に用意し、手作業の venv activation や個別ファイルコピーには依存しません。

ライブラリ要件と Gemma 4 対応ブランチの扱いは `docs/libraries.md` を参照してください。

## User Commands
標準セットアップ:
```bat
uv sync
uv run python download.py
```

ローカルの変換済み OpenVINO モデルを使う:
```bat
uv run python download.py C:\models\gemma-4-E4B-it-int8-ov
```

任意の Hugging Face repo を使う:
```bat
uv run python download.py OpenVINO/gemma-4-E4B-it-int8-ov
```

アプリ起動:
```bat
uv run python run.py
```

## Download Contract
`download.py` は repo root から `uv run python download.py` で呼び出します。

実装順序:
1. `config.json` から `model.local_dir` を読む。
2. `scripts\check_model_files.py <model_dir>` と同じ必須ファイル定義でモデルを確認する。
3. 必須ファイルがすでにそろっていれば `Model is ready in ...` を表示して成功終了する。
4. 第 1 引数があればそれを `SOURCE` にする。
5. 第 1 引数がなければ `config.json` の `model.download_source` を `SOURCE` にする。
6. `SOURCE` が空ならエラー表示して exit code `1`。
7. `SOURCE` が既存ディレクトリなら必須ファイル確認後に `model.local_dir` へコピーする。
8. `SOURCE` がディレクトリでなければ Hugging Face repo ID として扱う。
9. Hugging Face repo から必須 OpenVINO ファイルを直接取得する。
10. 直接取得できない、またはファイルがそろわない場合は `optimum-cli export openvino --task image-text-to-text` でローカル変換する。
11. 最後に必須ファイルを再確認する。
12. 成功したら `Download complete.` を表示する。

## Helper Scripts
`scripts\print_model_dir.py`:
- repo root の `config.json` を読む。
- `model.local_dir` を stdout に 1 行だけ出す。
- 相対パスの解決はしない。呼び出し側と後続処理は repo root 基準で動く。

`scripts\print_model_download_source.py`:
- repo root の `config.json` を読む。
- `model.download_source` を stdout に 1 行だけ出す。
- 値がなければ空文字を出す。

`scripts\check_model_files.py <model_dir>`:
- `<model_dir>` に必須 OpenVINO ファイルがすべてあるか確認する。
- 不足があれば不足ファイル名を stderr に含め、exit code `1`。
- そろっていれば `Model files OK: <model_dir>` を stdout に出し、exit code `0`。
- 引数数が違う場合は usage を stderr に出し、exit code `2`。

`scripts\prepare_model.py <source> <target_dir>`:
- source が既存ディレクトリなら、必須ファイルがそろっていることを確認して target にコピーする。
- source がディレクトリで必須ファイル不足なら、コピーせず exit code `1`。
- source がディレクトリでなければ Hugging Face repo ID として扱う。
- まず `huggingface_hub.snapshot_download()` で必須 OpenVINO ファイルだけを `target_dir` に直接取得する。
- 取得後に必須ファイルがそろえば成功。
- 直接取得できない、またはファイルがそろわない場合はローカル変換に進む。
- ローカル変換では既存の `target_dir` を削除して作り直し、`.venv\Scripts\optimum-cli.exe export openvino --model <source> --task image-text-to-text <target_dir>` を実行する。
- `optimum-cli.exe` が見つからない場合だけ `optimum-cli` を PATH から呼ぶ。
- export 後に必須ファイルを確認する。
- 不足があれば、target 直下の出力ファイル一覧を stderr に出してから exit code `1`。

## Required Model Files
`check_model_files.py` と `prepare_model.py` は同じ必須ファイル定義を使います。

- `chat_template.jinja`
- `config.json`
- `generation_config.json`
- `openvino_config.json`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`
- `openvino_language_model.xml`
- `openvino_language_model.bin`
- `openvino_text_embeddings_model.xml`
- `openvino_text_embeddings_model.bin`
- `openvino_text_embeddings_per_layer_model.xml`
- `openvino_text_embeddings_per_layer_model.bin`
- `openvino_vision_embeddings_model.xml`
- `openvino_vision_embeddings_model.bin`
- `preprocessor_config.json`
- `processor_config.json`
- `tokenizer.json`
- `tokenizer_config.json`

Optimum Intel の `OVModelForVisualCausalLM.from_pretrained()` と `AutoProcessor.from_pretrained()` が model directory を読むため、これらのファイルは `model.local_dir` 直下に置きます。

## Config Inputs
`download.py` が読む設定は `config.json` の `model` セクションだけです。

```json
{
  "model": {
    "local_dir": "model",
    "download_source": "OpenVINO/gemma-4-E4B-it-int8-ov"
  }
}
```

`model.local_dir`:
- 必須。
- 既定値として `model` を想定する。
- 通常は repo root からの相対パスを使う。

`model.download_source`:
- `download.py` に引数がない場合の source。
- Hugging Face repo ID を想定する。
- 空の場合、引数なしの `download.py` は失敗する。

## Expected Output
初回セットアップで Hugging Face repo から OpenVINO モデルを取得する場合:
```text
Preparing model from OpenVINO/gemma-4-E4B-it-int8-ov...
Exported OpenVINO model files to model
Model files OK: model
Download complete.
```

すでにモデルがそろっている場合:
```text
Model is ready in model.
```

Optimum Intel や Transformers が warning を出すことがあります。最終的に `Model files OK` と `Download complete.` が出て exit code `0` なら成功です。

## Failure Behavior
失敗時は non-zero exit code を返します。

代表的な失敗:
- `uv` がない。
- Python 3.12 以上が見つからない。
- `SOURCE` が空。
- ローカル source directory に必須 OpenVINO ファイルが足りない。
- Hugging Face download と `optimum-cli export openvino` の両方で必須ファイルを用意できない。

モデルファイル不足時のエラーには、不足ファイル名か export 出力ファイル一覧を含めます。次に直すべき対象が分かることを優先します。

## Implementation Rules
- `download.py` は config 読み取り、source 選択、copy/download/export、検証だけに留める。
- Python helper は repo root 基準で `config.json` を読む。
- `uv run python ...` を使い、shell activation に依存しない。
- Gemma 4 対応の依存関係は `pyproject.toml` と `docs/libraries.md` を一致させる。
- OpenVINO export は `python -m optimum.exporters.openvino` ではなく `optimum-cli export openvino --task image-text-to-text` を使う。
- `prepare_model.py` は既存 target を削除するのは export 直前だけにする。
- 必須ファイルの定義は `check_model_files.py` の `REQUIRED_MODEL_FILES` を source of truth にする。
- stdout は通常の進行状況、stderr は失敗理由に使う。
