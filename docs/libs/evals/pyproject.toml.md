# `libs/evals/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `deepagents-evals` |
| Version | `0.0.1` |
| Description | Evaluation suite and Harbor integration for Deep Agents |
| Python requirement | `>=3.12` |
| Build backend | `setuptools>=75.0` |
| Packages | `deepagents_harbor*`, `deepagents_evals*` |
| Package data | `deepagents_evals/categories.json` |

## Runtime Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | `>=0.5.0` | Core agent framework (local editable in dev) |
| `langchain` | `>=1.2.13` | LLM orchestration |
| `deepagents-cli` | (local path in dev) | CLI tools for running evals |
| `harbor` | `>=0.1.12` | External eval harness |
| `langsmith` | `>=0.4.0` | Observability and dataset management |
| `modal` | `>=0.64.0` | Modal sandbox runtime |
| **LangChain model providers** | | |
| `langchain-anthropic` | `>=1.0.0` | Claude |
| `langchain-baseten` | `>=0.2.0` | Baseten |
| `langchain-deepseek` | `>=1.0.0` | DeepSeek |
| `langchain-fireworks` | `>=1.0.0` | Fireworks |
| `langchain-google-genai` | `>=4.0.0` | Gemini |
| `langchain-groq` | `>=1.0.0` | Groq |
| `langchain-mistralai` | `>=1.0.0` | Mistral |
| `langchain-nvidia-ai-endpoints` | `>=1.0.0` | NVIDIA NIM |
| `langchain-ollama` | `>=1.0.0` | Ollama |
| `langchain-openai` | `>=1.0.0` | OpenAI |
| `langchain-openrouter` | `>=0.1.0` | OpenRouter |
| `langchain-xai` | `>=1.0.0` | xAI |
| **Eval-specific** | | |
| `datasets` | `>=3.0.0` | HuggingFace datasets |
| `nltk` | `>=3.9.0` | Text processing |
| `openevals` | `>=0.1.3` | Open evaluation utilities |
| `tiktoken` | `>=0.8.0` | Token counting |

## Optional Dependencies

- `charts`: `matplotlib>=3.9.0` — Required for radar chart generation via `scripts/generate_radar.py`.

## Dependency Overrides (Security/Compatibility)

| Package | Override | Reason |
|---------|----------|--------|
| `openai` | `>=1.109.1,<2.0.0` | Compatibility pin |
| `e2b` | `==2.4.3` | Exact version pin |
| `python-multipart` | `>=0.0.22` | CVE-2026-24486: path traversal vulnerability |
| `protobuf` | `>=6.33.5` | CVE-2026-0994: JSON recursion depth bypass DoS |

## Linting Configuration

- Line length: 100
- Target version: Python 3.12
- Excludes: `tests/evals/data/bfcl_apis` (vendored benchmark files)
- Selects all rules; ignores: `C90` (complexity), `COM812`, `ISC001`, `E501`, `FBT`, `FIX002`, `PLR09`, `TD002`, `TD003`
- Docstring convention: Google style
- `ban-relative-imports = "all"`
- Known first-party: `deepagents_evals`, `deepagents_harbor`

## pytest Configuration

- `asyncio_mode = "auto"`
- `testpaths = ["tests"]`
