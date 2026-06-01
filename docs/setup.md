# Setup

`setup.bat` はこのプロジェクトの標準セットアップ手順です。手作業で venv を activate したり、モデルファイルを個別にコピーしたりしなくてよい状態を目指します。

## User Flow
標準:
```bat
setup.bat
run.bat
```

ローカルの OpenVINO モデルを使う:
```bat
setup.bat C:\models\TinySwallow-1.5B-Instruct
```

任意の Hugging Face repo を使う:
```bat
setup.bat SakanaAI/TinySwallow-1.5B-Instruct
```

## Setup Contract
`setup.bat` は repo root から実行される前提で、次の順に処理します。

1. `%~dp0` に `cd`
2. `py -3`、なければ `python` で Python 3.12+ を探す
3. `uv` がなければ失敗
4. `uv sync`
5. `.venv\Scripts\python.exe` を以後の Python として使う
6. `scripts\print_model_dir.py` で `model.local_dir` を読む
7. `scripts\check_model_files.py` でモデル必須ファイルを確認
8. そろっていれば成功終了
9. 引数があればそれを source にする
10. 引数がなければ `scripts\print_model_download_source.py` で default source を読む
11. source が空なら usage を出して失敗
12. source が既存ディレクトリなら必須ファイルをコピー
13. それ以外は Hugging Face repo として扱う
14. repo に OpenVINO ファイルがあれば取得
15. なければ `optimum-cli export openvino` でローカル変換
16. 最後に必須ファイルを再確認し、足りなければ失敗

## Required Model Files
- `openvino_model.xml`
- `openvino_model.bin`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`

## Implementation Notes
- `activate` に依存しない
- batch 内の長い `python -c` は避ける
- config 読み取りは小さな Python helper に寄せる
- stdout は短く、次に何をすればよいか分かる文にする
- 失敗時は non-zero exit code を返す
