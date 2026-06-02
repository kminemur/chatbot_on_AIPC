# Chatbot on AI PC

Windows AI PC 向けのローカルチャットアプリです。FastAPI でブラウザ UI を出し、OpenVINO で `OpenVINO/gemma-4-E4B-it-int8-ov` を実行します。

## Quick Start
Requirements:
- Windows 10/11
- Python 3.12+
- `uv`
- 空き容量 10 GB 以上

Library note:
- `OpenVINO/gemma-4-E4B-it-int8-ov` は experimental model のため、Gemma 4 対応ブランチの `optimum-intel` と `transformers==5.5.0` を使います。
- ライブラリ方針は `docs/libraries.md` を参照してください。

Setup:
```bat
setup.bat
```

Run:
```bat
run.bat
```

Open:
- UI: 起動ログに表示された URL。通常は `http://127.0.0.1:8000/`
- Health: 起動ログに表示された URL + `/health`

`8000` が使用中の場合は、`8001` 以降の空きポートで自動起動します。

## Model Setup
通常は `setup.bat` だけで十分です。

`setup.bat` は次を行います:
- `uv sync` で `.venv` を作成/更新
- `config.json` の `model.local_dir` を読む
- `model/` に OpenVINO 形式の必須ファイルがあるか確認
- なければ `model.download_source` から準備
- Hugging Face 側に OpenVINO ファイルがなければ `optimum-cli export openvino` でローカル変換

ローカルに変換済みモデルがある場合:
```bat
setup.bat C:\models\gemma-4-E4B-it-int8-ov
```

別の Hugging Face repo を使う場合:
```bat
setup.bat OpenVINO/gemma-4-E4B-it-int8-ov
```

標準設定では `OpenVINO/gemma-4-E4B-it-int8-ov` を使います。

必須ファイル:
- `openvino_model.xml`
- `openvino_model.bin`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`

## API
`POST /api/chat`
```json
{"message":"Hello","max_tokens":256,"temperature":0.7,"top_p":0.9}
```

Response:
```json
{"response":"Hi","inference_time":2.14,"tokens_generated":120}
```

Rules:
- `response` はユーザー向けの回答だけを返す
- `think`, `<think>...</think>`, `reasoning:` などの内部推論表示は UI/API に出さない

## Project Map
- `setup.bat`: 依存関係とモデルを準備
- `run.bat`: アプリ起動
- `config.json`: モデル、推論、サーバー、デバイス設定
- `app/`: FastAPI backend
- `static/`: browser UI
- `scripts/`: setup 用ヘルパー
- `docs/`: 実装ルールと詳細仕様

## For Implementers
最初に読む順番:
1. `docs/requirements_definition.md`
2. `docs/technical_specification.md`
3. `docs/libraries.md`
4. `docs/setup.md`

実装方針:
- 速く作るため、構成を増やさない
- Windows batch の複雑な quoting は避け、Python helper script を使う
- 変更した仕様は同じタスクで docs と README に反映する
