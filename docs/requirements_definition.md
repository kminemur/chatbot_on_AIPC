# Requirements Definition

## Goal
Intel AI PC 上で、OpenVINO モデルをローカル実行する小さなチャットアプリを作る。

## Must Have
- Windows 10/11 で動く
- Python 3.12+ と `uv` を使う
- `uv sync` で `.venv` を準備できる
- `uv run python download.py` でモデルを準備できる
- `uv run python run.py` でサーバーを起動でき、`8000` が使用中なら次の空きポートを使う
- ブラウザ UI から 1 メッセージを送信できる
- `POST /api/chat` が 1 回の完全な回答を JSON で返す
- `GET /health` が状態と device を返す
- 推論はローカルで行い、チャット内容をクラウドへ送らない
- GPU を優先し、使えない場合は CPU に fallback する
- `think`, `<think>...</think>`, `reasoning:` などの内部推論表示を API/UI に出さない

## Keep Out
- ユーザー認証
- クラウド同期
- 複雑な frontend framework
- streaming response
- 複数会話履歴の永続化

## Acceptance Checks
- `uv run python download.py` が repo root から成功する
- `uv run python download.py <local_dir>` がローカル OpenVINO モデルを `model.local_dir` にコピーできる
- `uv run python download.py <hf_repo>` が Hugging Face repo からモデルを準備できる
- Hugging Face repo に OpenVINO ファイルがない場合、ローカル変換で必須ファイルを生成できる
- `uv run python scripts\check_model_files.py model` が成功する
- `uv run python run.py` でサーバーが起動し、設定ポートが使用中でも空きポートで起動する
- `GET /health` が `status`, `model_loaded`, `device` を返す
- `POST /api/chat` が `response`, `inference_time`, `tokens_generated` を返す
- モデルファイル不足時は不足ファイル名を含む明確なエラーを返す

## Handoff Rule
仕様や挙動を変えたら、同じタスクで次を更新する。
- `Readme.md`
- `docs/requirements_definition.md`
- `docs/technical_specification.md`
- 必要なら `docs/setup.md`
