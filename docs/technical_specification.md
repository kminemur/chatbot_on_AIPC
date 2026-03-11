# Technical Specification

## Objective
Provide a local chat app that serves a simple browser UI and runs OpenVINO inference on Intel AI PC hardware.

## Main Components
- `run.py`: reads `config.json`, resolves the device, starts Uvicorn
- `app/main.py`: creates the FastAPI app and serves static files
- `app/api/chat.py`: exposes `POST /api/chat`
- `app/models/model_manager.py`: resolves the model path and caches state
- `app/models/ov_inference.py`: performs OpenVINO generation
- `static/`: browser client

## Expected Startup Flow
1. Load `config.json`
2. Resolve device priority as NPU, then GPU, then CPU
3. Start FastAPI with Uvicorn
4. Load model lazily on first inference request

## API Contract
`GET /health`
```json
{"status":"ok","model_loaded":true,"device":"AUTO:NPU,GPU,CPU"}
```

`POST /api/chat`
```json
{"message":"Hello","max_tokens":512,"temperature":0.7,"top_p":0.9}
```

Response:
```json
{"response":"Hi","inference_time":2.14,"tokens_generated":120}
```

## Config Contract
`config.json` should contain:
- `model.name`
- `model.local_dir`
- `model.max_context_length`
- `inference.max_tokens`
- `inference.temperature`
- `inference.top_p`
- `inference.top_k`
- `inference.repetition_penalty`
- `server.host`
- `server.port`
- `server.log_level`
- `hardware.preferred_device`
- `hardware.fallback_order`

## Constraints
- Windows-first
- Local-only execution
- No streaming is required
- Keep the frontend simple

## Guidance For The Next AI Agent
- Do not assume setup automation exists unless it is implemented in the repo.
- Prefer updating `run.py`, `config.json`, and `docs/` together when behavior changes.
- Keep the API stable unless the docs are updated in the same change.
