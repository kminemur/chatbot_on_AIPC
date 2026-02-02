# Chatbot on AI PC

This repository provides a small local chatbot that runs the DeepSeek-R1-Distill-Qwen-1.5B-int4-cw-ov model with OpenVINO on an Intel AI PC (NPU). The app stays entirely on your machine and serves a simple browser UI.

## Features
- Local inference accelerated by OpenVINO with automatic fallback to GPU or CPU when no NPU is available
- Browser-based chat interface that returns the full answer after each request (no streaming requirement)
- Works fully offline once the model files are downloaded

## Requirements
- Windows 10 or 11
- Python 3.10-3.12 (3.12 recommended)
- Intel AI PC with NPU (recommended), GPU or CPU fallback is supported
- At least 8 GB RAM and ~10 GB of free disk space (model files are large)

## Quick Start
For copy/paste setup commands, see `docs/setup.md`.

1. Clone and enter the repository:
   ```powershell
   git clone https://github.com/kazukiminemura/chatbot_on_AIPC.git
   cd chatbot_on_AIPC
   ```
2. Create and activate a virtual environment:
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate
   python -m pip install --upgrade pip wheel
   ```
3. Install dependencies:
   ```powershell
   pip install -r requirements.txt
   ```
4. (First run only) Download the OpenVINO model snapshot with the Hugging Face CLI:
   ```powershell
   pip install "huggingface_hub[cli]"
   hf download OpenVINO/DeepSeek-R1-Distill-Qwen-1.5B-int4-cw-ov --local-dir model
   ```
5. Start the application:
   ```powershell
   python run.py
   ```
6. Open `http://127.0.0.1:8000/` in your browser to start chatting.
7. Optional health check: open `http://127.0.0.1:8000/health` to confirm the model loads.

## Model Files
When you run the server for the first time, download the OpenVINO model snapshot into the local `model/` directory using:
```powershell
hf download OpenVINO/DeepSeek-R1-Distill-Qwen-1.5B-int4-cw-ov --local-dir model
```
Once the download finishes, the application will load the model directly from `model/`.

Required files include:
- `openvino_model.xml` / `openvino_model.bin`
- `openvino_tokenizer.xml` / `openvino_tokenizer.bin`
- `openvino_detokenizer.xml` / `openvino_detokenizer.bin`

> Windows Note: If you see errors about long paths, run `git config --system core.longpaths true` or clone the repository into a shorter directory name.

## Troubleshooting
- **Dependency issues**: Use a fresh venv (`python -m venv .venv`) and reinstall with `pip install -r requirements.txt`.
- **`ImportError` related to OpenVINO GenAI pipelines**: Ensure you installed the pinned versions from `requirements.txt`.
- **Missing `huggingface_hub.errors`**: Update the hub client with `pip install --upgrade "huggingface-hub>=0.23.0"`.
- **Model cannot be opened / files missing**: Ensure the OpenVINO snapshot exists in `model/` (or update `config.json -> model.local_dir`) and the required files above exist.
- **`Unexpected UTF-8 BOM` when loading `config.json`**: Save `config.json` with plain UTF-8 encoding (no BOM). In PowerShell you can run `python -c "import pathlib; p = pathlib.Path('config.json'); p.write_text(p.read_text(encoding='utf-8-sig'), encoding='utf-8')"` to strip the BOM automatically.

## Project Layout
```
chatbot_on_AIPC/
|-- app/                # FastAPI backend and model helpers
|-- static/             # HTML, CSS, JavaScript for the web client
|-- model/              # Hugging Face snapshot with the OpenVINO model files
|-- docs/               # Simplified documentation
|-- config.json         # Runtime configuration
`-- requirements.txt    # Python dependencies
```

## Documentation
- `docs/setup.md` (recommended)
- `docs/requirements_definition.md`
- `docs/technical_specification.md`
