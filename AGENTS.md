# AGENTS.md

This file contains working instructions for AI agents and development-assistance tools that operate in this repository.

## Project Context

This project is a small local chatbot application for Windows AI PCs. It runs an OpenVINO model locally and provides a browser UI.

Core assumptions:
- Target Windows 10/11.
- Use Python 3.12+ and `uv`.
- Run inference locally. Do not send chat content to the cloud.
- Run `OpenVINO/gemma-4-E4B-it-int8-ov` through Optimum Intel and Transformers.
- Prefer GPU, then fall back to CPU when GPU is unavailable.
- Do not expose internal reasoning markers such as `think`, `<think>...</think>`, or `reasoning:` in the API or UI.

## Read First

Before making changes, read the relevant documents for the task.

- `Readme.md`: User-facing overview, startup flow, and API summary.
- `docs/requirements_definition.md`: Required behavior, out-of-scope items, and acceptance checks.
- `docs/technical_specification.md`: Architecture, API, model contract, and change rules.
- `docs/setup.md`: Detailed behavior for `uv sync` and `download.py`.
- `docs/libraries.md`: Dependency policy for OpenVINO, Optimum Intel, and Transformers.
- `DESIGN.md`: Product design, architecture intent, and UI design direction.

## Commands

Standard setup:
```bat
uv sync
```

Prepare the model:
```bat
uv run python download.py
```

Use a local OpenVINO model:
```bat
uv run python download.py C:\models\gemma-4-E4B-it-int8-ov
```

Run the app:
```bat
uv run python run.py
```

Validate model files:
```bat
uv run python scripts\check_model_files.py model
```

## Implementation Rules

- Keep the app small. Add files only when they remove real complexity.
- Use `uv run python ...`; do not depend on manually activating a virtual environment.
- Treat relative paths as relative to the repository root.
- Preserve the `config.json` contract.
- Treat `server.port` as the starting port. If it is already in use, try the next available port.
- Do not eagerly load the model during server startup. Load it lazily on the first chat request.
- Validate required OpenVINO files after model preparation.
- Use `optimum-cli export openvino --task image-text-to-text` for OpenVINO export.
- Do not replace the runtime path with `openvino-genai` or `LLMPipeline`-based code.
- The API `response` field must contain only user-facing answer text.

## Library Rules

The standard runtime imports are:

```python
from optimum.intel.openvino import OVModelForVisualCausalLM
from transformers import AutoProcessor
```

Important dependency constraints:
- `openvino>=2026.1.0`
- `optimum-intel @ git+https://github.com/rkazants/optimum-intel.git@support_gemma_4`
- `transformers==5.5.0`

Keep this policy until the model card confirms that Gemma 4 support has landed in released public versions of `optimum-intel` and `transformers`.

## API Rules

`GET /health` returns at least:

```json
{"status":"ok","model_loaded":false,"device":"AUTO:GPU,CPU"}
```

`POST /api/chat` accepts:

```json
{"message":"Hello","max_tokens":512,"temperature":0.7,"top_p":0.9}
```

The response is:

```json
{"response":"Hi","inference_time":2.14,"tokens_generated":120}
```

## UI Rules

- Do not add a complex frontend framework.
- Keep the browser UI small and reliable for sending one message and displaying one response.
- Match colors, spacing, and component styling to `DESIGN.md`.
- Do not show internal reasoning, raw debug output, or UI language that implies cloud transmission.

## Keep Out

The following are out of scope:

- User authentication
- Cloud sync
- Streaming responses
- Persistent multi-conversation history
- Complex frontend frameworks

## Change Rule

When behavior or specifications change, update the required documentation in the same task.

- `Readme.md`
- `docs/requirements_definition.md`
- `docs/technical_specification.md`
- `docs/setup.md`, when setup behavior changes
- `DESIGN.md`, when UI or experience design changes
