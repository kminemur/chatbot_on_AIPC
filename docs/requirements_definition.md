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
- `python run.py` starts the server from repo root
- `GET /health` returns status and device info
- `POST /api/chat` returns JSON with response text and timing
- Missing model files produce a clear error

## Handoff Rule
When the next AI agent changes behavior, it must update this file, `docs/technical_specification.md`, and `Readme.md` in the same task.
