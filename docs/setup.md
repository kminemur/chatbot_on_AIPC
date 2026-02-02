# Setup (Windows)

This app runs a local OpenVINO LLM server (FastAPI) and a simple browser UI.

## 1) Prerequisites
- Windows 10/11
- Python 3.12 (3.10+ should work, but 3.12 is recommended)
- Disk: ~10GB free (model files are large)

## 2) Create venv + install deps
```powershell
python -m venv .venv
.\.venv\Scripts\Activate
python -m pip install --upgrade pip wheel
pip install -r requirements.txt
```

## 3) Download model files
This project expects the Hugging Face snapshot under `./model/`.

```powershell
pip install "huggingface_hub[cli]"
hf download OpenVINO/DeepSeek-R1-Distill-Qwen-1.5B-int4-cw-ov --local-dir model
```

The folder must contain at least:
- `openvino_model.xml` / `openvino_model.bin`
- `openvino_tokenizer.xml` / `openvino_tokenizer.bin`
- `openvino_detokenizer.xml` / `openvino_detokenizer.bin`

## 4) Run
```powershell
python run.py
```

Open:
- UI: `http://127.0.0.1:8000/`
- Health: `http://127.0.0.1:8000/health`

## 5) Troubleshooting
- `openvino-genai is not available` / import errors: run `pip install -r requirements.txt` inside the venv.
- `Model files are missing`: confirm `config.json -> model.local_dir` points to your model folder and that the files above exist.
- If you run from another working directory: use `python run.py` from the repo root (recommended).
