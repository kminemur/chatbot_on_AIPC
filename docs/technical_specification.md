# Technical Specification

## Objective
Provide a local chat app that serves a simple browser UI and runs OpenVINO inference on Intel AI PC hardware.

## Main Components
- `setup.bat`: creates `.venv`, installs packages, and prepares the model directory
- `scripts/print_model_dir.py`: prints `model.local_dir` from `config.json` for Windows-safe batch integration
- `run.py`: reads `config.json`, resolves the device, starts Uvicorn
- `app/main.py`: creates the FastAPI app and serves static files
- `app/api/chat.py`: exposes `POST /api/chat`
- `app/models/model_manager.py`: resolves the model path and caches state
- `app/models/ov_inference.py`: performs OpenVINO generation
- `scripts/download_model.py`: copies a local model folder or downloads a Hugging Face snapshot
- `static/`: browser client
- `config.json`: stores model, inference, server, and hardware defaults

## Expected Startup Flow
1. Load `config.json`
2. Resolve device priority as NPU, then GPU, then CPU
3. Start FastAPI with Uvicorn
4. Load model lazily on first inference request
5. Return a 500 error with a clear message if the model directory or OpenVINO runtime is not ready

## `setup.bat` Contract
The setup script is Windows-first and must be implementable by another AI agent from this file alone.

Required sequence:
1. `cd` to the repository root using `%~dp0`
2. Detect Python using `py -3` first, then `python`
3. Validate that the detected interpreter is between Python 3.10 and 3.12
4. Create `.venv` if `.venv\Scripts\python.exe` does not exist
5. Use `.venv\Scripts\python.exe` for `pip install` and all helper script invocations
6. Install `requirements.txt`
7. Resolve `model.local_dir` from `config.json`
8. Check whether the model directory already contains all six required OpenVINO files
9. If complete, exit successfully
10. If incomplete and no source argument was passed, print usage and exit non-zero
11. If a source argument was passed:
12. Copy from that directory when it exists locally
13. Otherwise download from Hugging Face into the configured model directory
14. Validate the required files again and fail clearly if they are still missing

Implementation guidance:
- Avoid shell activation. The script should work without `call .venv\Scripts\activate`
- Avoid long inline `python -c` commands inside `for /f`; Windows quoting is error-prone
- Prefer a helper such as `scripts/print_model_dir.py` to return one value to batch
- Return non-zero on every failure path
- Keep console output short and actionable

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
- `setup.bat` is required behavior, not optional tooling.
- If you change setup behavior, update `setup.bat`, `Readme.md`, and `docs/setup.md` together.
- On Windows batch, prefer helper Python scripts over nested quoting tricks.
- Prefer updating `run.py`, `config.json`, and `docs/` together when behavior changes.
- Keep the API stable unless the docs are updated in the same change.
