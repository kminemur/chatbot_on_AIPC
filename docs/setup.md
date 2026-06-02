# Setup

この文書は、現在の `setup.bat` を再実装できる粒度の仕様です。

`setup.bat` の責務は、Windows 上で Python 環境を同期し、`config.json` が指す OpenVINO モデル一式を `model.local_dir` に用意することです。手作業の venv activation や個別ファイルコピーには依存しません。

## User Commands
標準セットアップ:
```bat
setup.bat
```

ローカルの変換済み OpenVINO モデルを使う:
```bat
setup.bat C:\models\gemma-4-E4B-it-int8-ov
```

任意の Hugging Face repo を使う:
```bat
setup.bat OpenVINO/gemma-4-E4B-it-int8-ov
```

## Batch Contract
`setup.bat` は repo root 以外から呼ばれても動くように、最初に `%~dp0` へ移動します。

実装順序:
1. `where uv` で `uv` の存在を確認する。なければエラー表示して exit code `1`。
2. `where py` で Python Launcher を探す。あれば `py -3`、なければ `python` を使う。
3. 選んだ Python で `sys.version_info >= (3, 12)` を確認する。満たさなければ exit code `1`。
4. `uv sync` を実行して `.venv` を作成または更新する。
5. 以降の Python 実行は `.venv\Scripts\python.exe` を使う。activation はしない。
6. `scripts\print_model_dir.py` の stdout を `MODEL_DIR` に入れる。
7. `scripts\check_model_files.py "%MODEL_DIR%"` を実行する。
8. 必須ファイルがすでにそろっていれば `Model is ready in ...` を表示して成功終了する。
9. 第 1 引数があればそれを `SOURCE` にする。
10. 第 1 引数がなければ `scripts\print_model_download_source.py` の stdout を `SOURCE` にする。
11. `SOURCE` が空ならエラー表示して exit code `1`。
12. `scripts\prepare_model.py "%SOURCE%" "%MODEL_DIR%"` を実行する。
13. `prepare_model.py` が失敗したら、その exit code で失敗終了する。
14. 最後に `scripts\check_model_files.py "%MODEL_DIR%"` を再実行する。
15. 成功したら `Setup complete.` を表示する。

Batch 側では複雑な JSON 読み取り、Hugging Face 操作、ファイルコピー、OpenVINO export を実装しません。Windows batch の quoting を単純に保つため、それらは Python helper に寄せます。

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
- ローカル変換では既存の `target_dir` を削除して作り直し、`.venv\Scripts\optimum-cli.exe export openvino --model <source> --task text-generation-with-past <target_dir>` を実行する。
- `optimum-cli.exe` が見つからない場合だけ `optimum-cli` を PATH から呼ぶ。
- export 後に必須ファイルを確認する。
- 不足があれば、target 直下の出力ファイル一覧を stderr に出してから exit code `1`。

## Required Model Files
`check_model_files.py` と `prepare_model.py` は同じ必須ファイル定義を使います。

- `openvino_model.xml`
- `openvino_model.bin`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`

`openvino-genai` の `LLMPipeline` が model directory をそのまま読むため、これらのファイルは `model.local_dir` 直下に置きます。

## Config Inputs
`setup.bat` が読む設定は `config.json` の `model` セクションだけです。

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
- 現在の helper は値をそのまま batch に返すため、通常は repo root からの相対パスを使う。

`model.download_source`:
- `setup.bat` に引数がない場合の source。
- Hugging Face repo ID を想定する。
- 空の場合、引数なしの `setup.bat` は失敗する。

## Expected Output
初回セットアップで Hugging Face repo から OpenVINO モデルを取得する場合:
```text
Syncing Python environment...
Preparing model from OpenVINO/gemma-4-E4B-it-int8-ov...
Exported OpenVINO model files to model
Model files OK: model
Setup complete.
```

すでにモデルがそろっている場合:
```text
Syncing Python environment...
Model is ready in model.
```

Optimum や Transformers が warning を出すことがあります。最終的に `Model files OK` と `Setup complete.` が出て exit code `0` なら成功です。

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
- `setup.bat` は環境確認、変数受け渡し、helper 起動だけに留める。
- Python helper は repo root 基準で `config.json` を読む。
- `.venv\Scripts\python.exe` を使い、shell activation に依存しない。
- OpenVINO export は `python -m optimum.exporters.openvino` ではなく `optimum-cli export openvino` を使う。
- `prepare_model.py` は既存 target を削除するのは export 直前だけにする。
- 必須ファイルの定義は `check_model_files.py` の `REQUIRED_MODEL_FILES` を source of truth にする。
- stdout は通常の進行状況、stderr は失敗理由に使う。
