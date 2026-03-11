# Setup

This file is for human operators. It describes the current manual setup only.

## Requirements
- Windows 10/11
- Python 3.10-3.12
- About 10 GB free disk space

## Steps
```powershell
python -m venv .venv
.\.venv\Scripts\Activate
python -m pip install --upgrade pip wheel
pip install -r requirements.txt
python run.py
```

## Model Folder
The app expects the model under `model/` unless `config.json` says otherwise.

Required files:
- `openvino_model.xml`
- `openvino_model.bin`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`

## URLs
- `http://127.0.0.1:8000/`
- `http://127.0.0.1:8000/health`

## Notes
- Run from the repository root.
- Save `config.json` as UTF-8 without BOM.
- If a future agent adds one-command setup, update this file after the implementation exists.
