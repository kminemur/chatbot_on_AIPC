# Requirements Definition

## Product Goal
Build a local chatbot for Intel AI PC that runs an OpenVINO model and exposes a small browser UI.

## Functional Requirements
- Serve the UI on `http://127.0.0.1:8000/`
- Accept one chat message per request
- Return one complete response per request
- Expose a health endpoint
- Load model files from a local directory
- Read inference defaults from `config.json`
- Provide a Windows `setup.bat` that can prepare both the Python environment and the model directory

## Non-Functional Requirements
- Run on Windows 10/11
- Support Python 3.10-3.12
- Prefer NPU, allow GPU and CPU fallback
- Avoid cloud dependencies during inference
- Keep docs short and implementation-directed

## Non-Goals
- Multi-user auth
- Cloud sync
- Complex frontend framework
- Streaming tokens

## Acceptance Criteria
- `setup.bat` prepares the Python environment from the repo root
- `setup.bat <source>` can populate the configured model directory from a local folder or Hugging Face repo
- `setup.bat` without arguments auto-downloads from `config.json` when model files are missing
- `setup.bat` detects Python from `py -3` or `python` and rejects versions outside 3.10-3.12
- `setup.bat` uses `.venv\Scripts\python.exe` after virtualenv creation instead of relying on shell activation
- `setup.bat` reads `model.local_dir` from `config.json` through a helper script or another implementation that is robust on Windows batch
- `setup.bat` prints usage help only when model files are missing and neither an argument nor `model.download_source` is available
- `python run.py` starts the server from repo root
- `GET /health` returns status and device info
- `POST /api/chat` returns JSON with response text and timing
- Missing model files produce a clear error
- The browser UI can submit a message to `POST /api/chat`

## Handoff Rule
When the next AI agent changes behavior, it must update this file, `docs/technical_specification.md`, and `Readme.md` in the same task.
