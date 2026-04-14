# Setup

This file is for human operators. It describes the current setup flow.

## Requirements
- Windows 10/11
- Python 3.10-3.12
- About 10 GB free disk space

## One-Command Setup
The purpose of `setup.bat` is to let a human or an AI agent prepare this repo from a plain Windows terminal without manually activating environments or copying files by hand.

If the model is already in the configured folder:
```powershell
setup.bat
```

If the model is missing, `setup.bat` downloads from the default Hugging Face repo in `config.json`:
```powershell
setup.bat
```

If you want to copy from a local OpenVINO model folder:
```powershell
setup.bat C:\models\DeepSeek-R1-Distill-Qwen-1.5B-int4-cw-ov
```

If you want to download from Hugging Face:
```powershell
setup.bat OpenVINO/DeepSeek-R1-Distill-Qwen-1.5B-int4-cw-ov
```

## What `setup.bat` Must Do
- Start from the repository root
- Detect Python from `py -3` or `python`
- Reject unsupported Python versions
- Create `.venv` when needed
- Use `.venv\Scripts\python.exe` for package installation and helper scripts
- Install the packages in `requirements.txt`
- Read `model.local_dir` from `config.json`
- Read `model.download_source` from `config.json`
- Check these required files in that directory:
  `openvino_model.xml`, `openvino_model.bin`, `openvino_tokenizer.xml`, `openvino_tokenizer.bin`, `openvino_detokenizer.xml`, `openvino_detokenizer.bin`
- Exit successfully if the files already exist
- If files are missing and no argument is provided:
  download from the default Hugging Face repo in `model.download_source`
- If files are missing and an argument is provided:
  treat an existing path as a local model folder, otherwise treat it as a Hugging Face repo id
- If files are missing and no argument is provided and no default source is configured:
  exit non-zero and print a short usage message

## Implementation Notes For AI Agents
- Windows batch quoting around `for /f` and `python -c` is fragile
- Prefer a helper Python script that prints one value, such as the configured model directory
- Use helper scripts for both model directory and default download source to avoid fragile quoting
- Do not require `activate`; calling the venv Python directly is the stable path
- Validate the model directory after copy or download, not only before
- Keep stdout human-readable because this script is the first thing operators will run

## Manual Steps
```powershell
python -m venv .venv
.\.venv\Scripts\Activate
python -m pip install --upgrade pip wheel
pip install -r requirements.txt
run.bat
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
- `setup.bat` reads `model.local_dir` from `config.json`.
- `setup.bat` reads `model.download_source` from `config.json`.
- `run.bat` starts the app with `.venv\Scripts\python.exe`.
- If the model files are missing and no source is provided, `setup.bat` downloads from `model.download_source`.
- User-visible responses must not display internal reasoning markers such as `think` or `<think>...</think>`.
