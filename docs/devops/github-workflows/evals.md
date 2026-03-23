# Workflow: `evals.yml` — Evaluations

## Overview

Runs the deepagents evaluation suite against one or more AI models. Evaluations measure correctness, solve rate, step efficiency, tool call efficiency, and per-category scores. Results are aggregated into a summary table in the workflow run and optionally rendered as radar charts published to the `eval-assets` branch.

## Trigger

`workflow_dispatch` only (manual trigger). Inputs:

| Input | Default | Description |
|---|---|---|
| `models` | `all` | Model preset to evaluate. Dropdown with individual model options and named sets (`all`, `set0`, `set1`, `set2`, `open`). |
| `models_override` | (empty) | Comma-separated `provider:model` specs that override the dropdown. |
| `eval_categories` | (empty) | Comma-separated category names to filter (e.g. `memory,hitl,tool_usage`). Empty = all categories. |

The effective model selection is: `models_override` if set, otherwise `models` dropdown, otherwise `all`.

## Secrets Required

| Secret | Provider |
|---|---|
| `LANGSMITH_API_KEY` | LangSmith tracing |
| `ANTHROPIC_API_KEY` | Anthropic models |
| `OPENAI_API_KEY` | OpenAI models |
| `GOOGLE_API_KEY` | Google models |
| `XAI_API_KEY` | xAI/Grok models |
| `MISTRAL_API_KEY` | Mistral models |
| `DEEPSEEK_API_KEY` | DeepSeek models |
| `GROQ_API_KEY` | Groq-hosted models |
| `OLLAMA_API_KEY` | Ollama Cloud models |
| `NVIDIA_API_KEY` | NVIDIA NIM models |
| `BASETEN_API_KEY` | Baseten-hosted models |
| `FIREWORKS_API_KEY` | Fireworks-hosted models |
| `OPENROUTER_API_KEY` | OpenRouter-hosted models |

## Jobs

### 1. `prep` — Prepare Matrix

- Logs dispatch inputs to the GitHub step summary
- Computes the eval model matrix by running `python .github/scripts/models.py eval` with `EVAL_MODELS` set from the resolved input

Output: `matrix` JSON used by the `eval` job's strategy.

### 2. `eval` — Run Evals (Matrix)

Runs once per model in the matrix. Up to 120 minutes per job. `fail-fast: false` so all models complete even if some fail.

Working directory: `libs/evals`.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python 3.12 + uv | `./.github/actions/uv_setup` with `evals` cache suffix |
| Install dependencies | `uv sync --group test` |
| Apply category filter | Adds `--eval-category <cat>` flags to `PYTEST_ADDOPTS` when categories specified |
| Run evals | `make evals` |
| Upload eval report | Uploads `evals_report.json` as a workflow artifact (`evals-report-<index>`) |

Environment variables set for each eval job include all provider API keys, LangSmith tracing config, and the model being evaluated.

### 3. `aggregate` — Aggregate Evals

Runs after all `eval` jobs (even if some fail). Downloads all `evals-report-*` artifacts and:

1. Runs `python .github/scripts/aggregate_evals.py` — generates summary tables and `evals_summary.json`
2. If summary exists, generates radar charts via `python scripts/generate_radar.py`
3. Publishes charts to the `eval-assets` branch at `runs/<run_id>/`
4. Appends chart images to the GitHub step summary

## Environment Variables

- `UV_NO_SYNC: "true"`, `UV_FROZEN: "true"` — freeze environment
- `PYTEST_ADDOPTS` — base flags including `--model <model>` and `--evals-report-file evals_report.json`
- `LANGSMITH_TRACING_V2: "true"` — enables LangSmith experiment tracing

## Concurrency

Grouped by workflow + ref + resolved model selection, with `cancel-in-progress: true`.

## Permissions

- `contents: write` — needed to push chart artifacts to `eval-assets` branch
