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

Install dependencies:
```bat
uv sync
```

Download model:
```bat
uv run python download.py
```

Run app:
```bat
uv run python run.py
```

Open:
- UI: 起動ログに表示された URL。通常は `http://127.0.0.1:8000/`
- Health: 起動ログに表示された URL + `/health`

`8000` が使用中の場合は、`8001` 以降の空きポートで自動起動します。

## Model Download
通常は `uv run python download.py` だけで十分です。

`download.py` は次を行います:
- `config.json` の `model.local_dir` を読む
- `model/` に OpenVINO 形式の必須ファイルがあるか確認
- なければ `model.download_source` から準備
- Hugging Face 側に OpenVINO ファイルがなければ `optimum-cli export openvino` でローカル変換

ローカルに変換済みモデルがある場合:
```bat
uv run python download.py C:\models\gemma-4-E4B-it-int8-ov
```

別の Hugging Face repo を使う場合:
```bat
uv run python download.py OpenVINO/gemma-4-E4B-it-int8-ov
```

標準設定では `OpenVINO/gemma-4-E4B-it-int8-ov` を使います。

必須ファイルの詳細は `docs/setup.md` を参照してください。Gemma 4 OpenVINO repo では `openvino_language_model.*`、text/vision embeddings、processor/tokenizer 設定をまとめて取得します。

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
- `download.py`: モデルを準備
- `run.py`: アプリ起動
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
- Python helper script でモデル準備の処理を小さく保つ
- 変更した仕様は同じタスクで docs と README に反映する
