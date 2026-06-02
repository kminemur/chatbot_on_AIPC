# Technical Specification

## Architecture
- `uv run python run.py` starts the app
- `run.py` loads `config.json`, finds an available port, and starts Uvicorn
- `app/main.py` creates FastAPI, mounts `static/`, and registers API routes
- `app/api/chat.py` exposes chat endpoints
- `app/models/model_manager.py` resolves model path, validates files, and lazy-loads inference
- `app/models/ov_inference.py` wraps Optimum Intel OpenVINO inference
- `download.py` prepares the configured model
- `scripts/` contains model preparation helpers used by `download.py`

Keep the app small. Add files only when they remove real complexity.

## Runtime Flow
1. User runs `uv run python run.py`
2. App loads `config.json`
3. App starts from `server.port` and uses the next available port if needed
4. Device selection prefers GPU, then fallback devices
5. Server starts before loading the model
6. First chat request validates `model.local_dir`
7. Optimum Intel OpenVINO model and processor are created lazily
8. Generated text is cleaned before returning to the user

## Setup Flow
Detailed setup behavior lives in `docs/setup.md`.

Implementation requirements:
- `uv sync` is the supported dependency setup command
- `uv run python download.py` is the supported model setup entry point
- Use helper scripts for config reads and model validation
- Use `uv run python ...` after `uv sync`
- Do not depend on shell activation
- Validate model files after every copy/download/export path
- Keep command-line entry points shallow; JSON reads, Hugging Face download, file copying, and OpenVINO export belong in Python helpers
- Export missing Hugging Face models with `optimum-cli export openvino --task image-text-to-text`
- Runtime library details live in `docs/libraries.md`

Download helper responsibilities:
- `download.py`: read config, choose source, skip existing complete models, copy/download/export the model, and validate output
- `scripts/print_model_dir.py`: print `model.local_dir` from `config.json`
- `scripts/print_model_download_source.py`: print `model.download_source` from `config.json`
- `scripts/check_model_files.py`: validate the required OpenVINO files and report missing names
- `scripts/prepare_model.py`: copy a local OpenVINO directory, download matching OpenVINO files from Hugging Face, or export with `optimum-cli`

## Config Contract
`config.json`:
```json
{
  "model": {
    "name": "OpenVINO/gemma-4-E4B-it-int8-ov",
    "local_dir": "model",
    "download_source": "OpenVINO/gemma-4-E4B-it-int8-ov",
    "max_context_length": 4096
  },
  "inference": {
    "max_tokens": 512,
    "temperature": 0.7,
    "top_p": 0.9,
    "top_k": 50,
    "repetition_penalty": 1.1
  },
  "server": {
    "host": "127.0.0.1",
    "port": 8000,
    "log_level": "info"
  },
  "hardware": {
    "preferred_device": "GPU",
    "fallback_order": ["CPU"]
  }
}
```

Relative paths in config are resolved from repo root.

`server.port` is the preferred starting port. If it is already in use, startup tries the next ports in order.

## API Contract
`GET /health`
```json
{"status":"ok","model_loaded":false,"device":"AUTO:GPU,CPU"}
```

`POST /api/chat`
```json
{"message":"Hello","max_tokens":512,"temperature":0.7,"top_p":0.9}
```

Response:
```json
{"response":"Hi","inference_time":2.14,"tokens_generated":120}
```

Rules:
- `response` contains only user-facing text
- Strip internal reasoning markers such as `think`, `<think>...</think>`, `reasoning:`, `analysis:`, `answer:`, and `final:`
- Missing model/runtime errors should be clear enough for the user to fix setup

## Model Contract
The configured model directory must contain:
- `chat_template.jinja`
- `config.json`
- `generation_config.json`
- `openvino_config.json`
- `openvino_tokenizer.xml`
- `openvino_tokenizer.bin`
- `openvino_detokenizer.xml`
- `openvino_detokenizer.bin`
- `openvino_language_model.xml`
- `openvino_language_model.bin`
- `openvino_text_embeddings_model.xml`
- `openvino_text_embeddings_model.bin`
- `openvino_text_embeddings_per_layer_model.xml`
- `openvino_text_embeddings_per_layer_model.bin`
- `openvino_vision_embeddings_model.xml`
- `openvino_vision_embeddings_model.bin`
- `preprocessor_config.json`
- `processor_config.json`
- `tokenizer.json`
- `tokenizer_config.json`

## Library Contract
`OpenVINO/gemma-4-E4B-it-int8-ov` is loaded through Optimum Intel, not `openvino-genai`.

The supported runtime imports are:
- `from optimum.intel.openvino import OVModelForVisualCausalLM`
- `from transformers import AutoProcessor`

The project depends on the Gemma 4 support branch of `optimum-intel` and pins `transformers==5.5.0`. Do not replace this with the public PyPI `optimum-intel` constraint unless the model card confirms that Gemma 4 support has landed in a released version.

## Change Rule
When behavior changes:
- update code
- update `Readme.md`
- update the relevant doc in `docs/`
- keep duplicated prose minimal
