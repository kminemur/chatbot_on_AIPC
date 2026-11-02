# DESIGN.md

This file describes the design intent, system shape, and UI direction for Chatbot on AI PC. The detailed implementation contracts live under `docs/`; this document focuses on the overall design and the reasoning behind it.

## Product Goal

Chatbot on AI PC provides a small browser-based chat application that runs an OpenVINO model locally on a Windows AI PC. Users send messages through the browser UI and receive locally generated answers without sending chat content to the cloud.

The core value is:

- It runs locally on Intel AI PC GPU/CPU hardware.
- Setup and startup are handled by short `uv` commands.
- The implementation and UI stay small, inspectable, and easy to verify.

## Target Environment

- OS: Windows 10/11
- Python: 3.12+
- Package/runtime command: `uv`
- Model runtime: OpenVINO + Optimum Intel + Transformers
- Standard model: `OpenVINO/gemma-4-E4B-it-int8-ov`
- Server: FastAPI + Uvicorn
- UI: Static browser UI

## Architecture

Runtime flow:

1. The user runs `uv run python run.py`.
2. `run.py` loads `config.json`.
3. Startup begins from `server.port`; if the port is in use, the next available port is used.
4. The FastAPI app starts and serves both static UI assets and API routes.
5. The first chat request validates the configured model directory.
6. The Optimum Intel OpenVINO model and processor are loaded lazily.
7. Generated output is cleaned before being returned to the API/UI.

Main components:

- `run.py`: Loads configuration, finds an available port, and starts Uvicorn.
- `app/main.py`: Creates the FastAPI app, mounts static files, and registers API routes.
- `app/api/chat.py`: Provides chat API endpoints.
- `app/models/model_manager.py`: Resolves the model path, validates files, and manages lazy loading.
- `app/models/ov_inference.py`: Wraps Optimum Intel OpenVINO inference.
- `download.py`: Prepares the configured model.
- `scripts/`: Helper scripts for model preparation and validation.
- `static/`: Browser UI.

## Runtime Design

The model is loaded on the first chat request instead of during server startup. This keeps server startup, health checks, and missing-model errors easier to separate and diagnose.

Device selection prefers GPU and falls back to CPU when needed. `GET /health` returns `status`, `model_loaded`, and `device` so users can inspect the current runtime state.

`POST /api/chat` returns one complete JSON response rather than streaming output. The `response` field contains only user-facing text. Internal reasoning markers such as `think`, `<think>...</think>`, `reasoning:`, `analysis:`, `answer:`, and `final:` are removed before the response reaches the API or UI.

## Setup Design

Dependencies are prepared with `uv sync`. The model is prepared with `uv run python download.py`. The setup flow must not depend on manually activating a virtual environment or copying individual files by hand.

Model preparation responsibilities:

- Read `model.local_dir` and `model.download_source` from `config.json`.
- Skip setup if an existing model directory is already complete.
- Accept either a local OpenVINO directory or a Hugging Face repo ID as the source.
- If the Hugging Face repo does not contain the required OpenVINO files, try OpenVINO export.
- Validate the required model files at the end of every preparation path.

## API Design

`GET /health`:

```json
{"status":"ok","model_loaded":false,"device":"AUTO:GPU,CPU"}
```

`POST /api/chat` request:

```json
{"message":"Hello","max_tokens":512,"temperature":0.7,"top_p":0.9}
```

`POST /api/chat` response:

```json
{"response":"Hi","inference_time":2.14,"tokens_generated":120}
```

Errors should help the user understand what to fix next. Missing model file errors should include the missing file names.

## UI Design Principles

The UI should feel like a quiet local AI PC chat tool: lightweight, calm, and direct. It should prioritize chat input, response display, and runtime state over marketing-style hero sections or complex navigation.

Core principles:

- Keep the screen small, readable, and comfortable for repeated use.
- Let the user reach the chat interaction immediately.
- Show runtime state such as `model_loaded` and device information in a restrained way.
- Make setup-related errors actionable.
- Do not expose internal reasoning or raw debug output.
- Avoid UI language that suggests cloud sync or remote message transmission.

## Color Balance

The UI color direction is based on pear tones: soft greens, warm yellow-greens, pale flesh colors, and small golden accents. The goal is to soften the technical feel of an AI inference app while keeping it readable and tool-like.

Recommended color balance:

- Base background: Pale cream-green inspired by pear flesh. About 55% of the screen.
- Surface: Near-white pear tint for inputs, response areas, and panels. About 25%.
- Primary: Yellow-green inspired by pear skin for send buttons, focus states, and primary actions. About 10%.
- Secondary: Deep olive green inspired by leaves and stems for headings, status text, and emphasis. About 5%.
- Accent: A small amount of ripe pear gold or warm blush for warnings, badges, and hover states. 5% or less.

Recommended tokens:

```css
:root {
  --color-bg: #f4f6df;
  --color-surface: #fffdf1;
  --color-surface-muted: #edf1cf;
  --color-primary: #a8bf3f;
  --color-primary-strong: #7f9828;
  --color-secondary: #4f6428;
  --color-accent: #d69b3a;
  --color-danger: #b65b42;
  --color-text: #243018;
  --color-muted: #68734b;
  --color-border: #d8ddb4;
}
```

Color usage:

- Use surface and border colors to create hierarchy instead of flooding the screen with one background color.
- Reserve primary color for actions so the UI does not become a single yellow-green wash.
- Use accent color for meaningful states, not decoration.
- Use dark olive text on pale backgrounds for adequate contrast.
- Keep danger color calm and compatible with the pear palette.

## Typography And Layout

- Use a system font stack that reads well with mixed Japanese and English text.
- Use roughly 14px to 16px as the baseline body size.
- Give chat messages enough line height for long generated answers.
- Keep buttons and input fields large enough for comfortable use in Windows browsers.
- Use modest border radii and avoid overly decorative card layouts.

## Scope Boundaries

The following are out of scope:

- User authentication
- Cloud sync
- Streaming responses
- Persistent multi-conversation history
- Complex frontend frameworks

## Documentation Rule

When design or behavior changes, update the relevant documents in the same change.

- User-facing instructions: `Readme.md`
- Requirements and acceptance checks: `docs/requirements_definition.md`
- Technical contracts: `docs/technical_specification.md`
- Setup details: `docs/setup.md`
- UI and experience design: `DESIGN.md`
- Agent working rules: `AGENTS.md`
