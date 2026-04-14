# Chatbot on AI PC

Local chatbot for Intel AI PC using OpenVINO and a browser UI.

## Purpose
- Run `SakanaAI/TinySwallow-1.5B-Instruct` locally.
- Use GPU inference by default, with CPU fallback when needed.
- Keep all chat data on the local machine.

## Current Scope
- FastAPI backend
- Browser UI
- Single-response chat API
- Local model loading from `model/`
- Lazy OpenVINO model initialization on first chat request
- Clear runtime errors for missing model files or missing OpenVINO packages
- Do not display internal reasoning or tags such as `think` in the UI or API response

## One-Command Setup
`setup.bat` is part of the product surface, not a convenience script. Another AI agent should be able to recreate it from the docs without reverse-engineering shell behavior.

Required behavior:
- Run from the repository root on Windows `cmd.exe`
- Detect Python 3.10-3.12 from `py -3` or `python`
- Create `.venv` if it does not exist
- Use `.venv\Scripts\python.exe` for all later package installs and helper scripts
- Install `requirements.txt`
- Read `model.local_dir` from `config.json`
- Read `model.download_source` from `config.json`
- Succeed immediately if all required model files already exist in that directory
- If model files are missing and no argument is provided, download from the configured default source
- If model files are missing, accept one optional source argument
- Treat the source as a local directory if it exists
- Otherwise treat the source as a Hugging Face repo id and download only the required model files
- Exit with a clear usage message if model files are missing and no source is available
- Print actionable error messages and return a non-zero exit code on failure

If the model files already exist, or if they are missing and should be downloaded from the default source:
```powershell
setup.bat
```

If you want setup to copy a local OpenVINO model folder into `model/`:
```powershell
setup.bat C:\models\TinySwallow-1.5B-Instruct
```

If you want setup to download a Hugging Face snapshot into `model/`:
```powershell
setup.bat SakanaAI/TinySwallow-1.5B-Instruct
```

## Run Manually
Requirements:
- Windows 10/11
- Python 3.10-3.12
- About 10 GB free disk space

Setup:
```powershell
python -m venv .venv
.\.venv\Scripts\Activate
python -m pip install --upgrade pip wheel
pip install -r requirements.txt
```

Model:
Place the model snapshot in `model/`, or use the default `model.download_source` in `config.json`:
`SakanaAI/TinySwallow-1.5B-Instruct`

Required configuration:
- `model.name`: `SakanaAI/TinySwallow-1.5B-Instruct`
- `model.download_source`: `SakanaAI/TinySwallow-1.5B-Instruct`

Required files:
- `openvino_model.xml`
- `openvino_model.bin`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`

Run:
```powershell
run.bat
```

Inference device:
- Default: GPU
- Fallback: CPU

Main files:
- `config.json`
- `run.py`
- `app/`
- `static/`
- `scripts/print_model_dir.py`
- `scripts/print_model_download_source.py`
- `scripts/download_model.py`
- `setup.bat`
- `run.bat`

Open:
- UI: `http://127.0.0.1:8000/`
- Health: `http://127.0.0.1:8000/health`
- Chat API: `POST http://127.0.0.1:8000/api/chat`

Sample request:
```json
{"message":"Hello","max_tokens":256,"temperature":0.7,"top_p":0.9}
```

Sample response:
```json
{"response":"Hi","inference_time":2.14,"tokens_generated":120}
```

Response rule:
- Return only the user-facing answer text in `response`
- Strip internal reasoning markers or sections such as `think`, `<think>...</think>`, or similar debug output before returning text

## For The Next AI Agent
Read these files first:
- `docs/requirements_definition.md`
- `docs/technical_specification.md`
- `docs/setup.md`

For `setup.bat`, do not rely on fragile inline quoting when reading `config.json`. Prefer calling a small Python helper script from the venv instead of embedding a long `python -c` expression inside `for /f`.
