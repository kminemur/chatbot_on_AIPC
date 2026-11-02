# Chatbot on AI PC

Windows AI PC 上で動く、ローカル実行の小さなチャットアプリです。ブラウザからメッセージを送り、OpenVINO で動かした `OpenVINO/gemma-4-E4B-it-int8-ov` が回答します。チャット内容をクラウドへ送信しないことを前提にしています。

## 特徴

- Windows 10/11 向け
- Python 3.12+ と `uv` でセットアップ
- FastAPI + Uvicorn によるローカル Web サーバー
- 静的なブラウザ UI
- OpenVINO + Optimum Intel + Transformers によるローカル推論
- GPU を優先し、利用できない場合は CPU にフォールバック
- API と UI には内部推論マーカーを表示しない

## 必要なもの

- Windows 10/11
- Python 3.12 以降
- `uv`
- モデル保存用の空き容量

初回セットアップでは依存関係とモデルファイルを取得します。ネットワーク接続が必要になるのはセットアップ時です。通常のチャット推論はローカルで実行します。

## クイックスタート

依存関係をインストールします。

```bat
uv sync
```

モデルを準備します。

```bat
uv run python download.py
```

アプリを起動します。

```bat
uv run python run.py
```

起動ログに表示された URL をブラウザで開きます。標準設定では `http://127.0.0.1:8000/` から開始します。`8000` が使用中の場合は、次の空きポートを探して起動します。

## モデルの準備

標準設定では `config.json` の内容に従い、`OpenVINO/gemma-4-E4B-it-int8-ov` を `model/` に準備します。

```bat
uv run python download.py
```

変換済みのローカル OpenVINO モデルを使う場合は、モデルディレクトリを指定します。

```bat
uv run python download.py C:\models\gemma-4-E4B-it-int8-ov
```

別の Hugging Face リポジトリを指定することもできます。

```bat
uv run python download.py OpenVINO/gemma-4-E4B-it-int8-ov
```

モデル準備では、OpenVINO 形式に必要なファイルがそろっているかを確認します。Hugging Face 側に OpenVINO ファイルがない場合は、`optimum-cli export openvino --task image-text-to-text` による変換を行う方針です。

## 設定

主な設定は `config.json` にあります。

```json
{
  "model": {
    "name": "OpenVINO/gemma-4-E4B-it-int8-ov",
    "local_dir": "model",
    "download_source": "OpenVINO/gemma-4-E4B-it-int8-ov",
    "max_context_length": 4096
  },
  "inference": {
    "max_tokens": 512,
    "temperature": 0.7,
    "top_p": 0.9
  },
  "server": {
    "host": "127.0.0.1",
    "port": 8000
  }
}
```

相対パスはリポジトリルートからの相対パスとして扱います。`server.port` は開始ポートであり、使用中の場合は次の空きポートを使います。

## API

### `GET /health`

サーバーとモデルの状態を返します。

```json
{
  "status": "ok",
  "model_loaded": false,
  "device": "AUTO:GPU,CPU"
}
```

モデルはサーバー起動時には読み込まず、最初のチャット要求で遅延ロードします。

### `POST /api/chat`

リクエスト:

```json
{
  "message": "Hello",
  "max_tokens": 512,
  "temperature": 0.7,
  "top_p": 0.9
}
```

レスポンス:

```json
{
  "response": "Hi",
  "inference_time": 2.14,
  "tokens_generated": 120
}
```

`response` にはユーザーに見せる回答だけを入れます。`think`、`<think>...</think>`、`reasoning:`、`analysis:` などの内部推論マーカーは API と UI に出しません。

## 依存関係

Gemma 4 対応のため、依存関係は意図的に固定しています。2026 年 7 月 17 日に公式モデルカードと公開パッケージを再確認し、`uv.lock` を最新の互換版で更新しています。

```toml
openvino>=2026.1.0
optimum-intel @ git+https://github.com/rkazants/optimum-intel.git@support_gemma_4
transformers==5.5.0
```

標準のランタイム import は次の方針です。

```python
from optimum.intel.openvino import OVModelForVisualCausalLM
from transformers import AutoProcessor
```

現在のロックでは、主なパッケージは `openvino==2026.2.1`、対応ブランチの `optimum-intel==1.27.0.dev0+eac3893`、`transformers==5.5.0`、`torch==2.13.0` です。

公開版 `optimum-intel==2.0.0` に Gemma 4 対応は入りましたが、配布メタデータが要求する `transformers>=4.45,<5.1` は、Gemma 4 が要求する `transformers>=5.5.0` と両立しません。公式モデルカードも引き続き上記のカスタムブランチと `transformers==5.5.0` を指定しているため、公開版だけへの移行はまだ行いません。この upstream の制約により、`uv pip check` は `optimum-intel` と `transformers` の1件の不一致を報告しますが、`uv` の override と同じ意図によるものです。

## 開発メモ

- アプリは小さく保つ
- 複雑なフロントエンドフレームワークを追加しない
- `uv run python ...` を使い、仮想環境の手動 activate に依存しない
- モデルはサーバー起動時に読み込まず、最初のチャット要求で読み込む
- チャット内容をクラウドへ送信する実装にしない
- 仕様や挙動を変えた場合は、関連ドキュメントも同じ変更で更新する

設計意図と UI 方針は `DESIGN.md` を参照してください。AI エージェント向けの作業ルールは `AGENTS.md` にあります。

## スコープ外

このプロジェクトでは、次の機能は扱いません。

- ユーザー認証
- クラウド同期
- ストリーミング応答
- 複数会話の永続履歴
- 複雑なフロントエンドフレームワーク
