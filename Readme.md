# Chatbot on AI PC

Local chatbot for Intel AI PC using OpenVINO and a browser UI.

## Purpose
- Run `DeepSeek-R1-Distill-Qwen-1.5B-int4-cw-ov` locally.
- Prefer NPU, but allow GPU or CPU fallback.
- Keep all chat data on the local machine.

## Current Scope
- FastAPI backend
- Browser UI
- Single-response chat API
- Local model loading from `model/`

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
Place the OpenVINO model snapshot in `model/`.
Required files:
- `openvino_model.xml`
- `openvino_model.bin`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`

Run:
```powershell
python run.py
```

Open:
- UI: `http://127.0.0.1:8000/`
- Health: `http://127.0.0.1:8000/health`

## For The Next AI Agent
Read these files first:
- `docs/requirements_definition.md`
- `docs/technical_specification.md`
- `docs/setup.md`

The docs are intentionally short and implementation-focused so another agent can continue work without re-deriving requirements.
