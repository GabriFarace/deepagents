# Workflow: `harbor.yml` — Harbor Benchmark

## Overview

Runs the [Harbor](https://github.com/av/harbor) coding agent benchmark against the `terminal-bench` dataset. Harbor evaluates coding agents on terminal-based tasks and tracks results in LangSmith. Supports multiple sandbox environments (Docker, Daytona, LangSmith, Modal, Runloop) and both SDK and CLI agent modes.

## Trigger

`workflow_dispatch` only. Inputs:

| Input | Default | Description |
|---|---|---|
| `models` | `all` | Model preset dropdown (`all`, `anthropic`, `openai`, `baseten`, or individual models) |
| `models_override` | (empty) | Comma-separated `provider:model` override |
| `sandbox_env` | `docker` | Sandbox environment: `docker`, `daytona`, `langsmith`, `modal`, `runloop` |
| `n_tasks` | `0` | Max tasks to run (`0` = all) |
| `concurrency` | `1` | Parallel sandbox slots |
| `agent_mode` | `sdk` | Agent implementation: `sdk` or `cli` |

## Secrets Required

| Secret | When Required |
|---|---|
| `LANGSMITH_API_KEY` | Always (experiment tracking) |
| `ANTHROPIC_API_KEY` | Anthropic models |
| `OPENAI_API_KEY` | OpenAI models |
| `BASETEN_API_KEY` | Baseten models |
| `DAYTONA_API_KEY` | `daytona` sandbox |
| `MODAL_TOKEN_ID` + `MODAL_TOKEN_SECRET` | `modal` sandbox |
| `RUNLOOP_API_KEY` | `runloop` sandbox |

## Environment Variables (Global)

- `HARBOR_DATASET_NAME: "terminal-bench"`
- `HARBOR_DATASET_VERSION: "2.0"`
- `UV_NO_SYNC: "true"`

## Jobs

### 1. `prep` — Prepare Matrix

- Logs dispatch inputs to GitHub step summary
- Computes model matrix via `python .github/scripts/models.py harbor` with `HARBOR_MODELS` env var
- Sets up Python 3.12 + installs evals dependencies
- Ensures the LangSmith dataset exists: `python scripts/harbor_langsmith.py ensure-dataset terminal-bench --version 2.0`

### 2. `harbor` — Run Harbor (Matrix, up to 6 hours)

Runs once per model. `fail-fast: false`. Working directory: `libs/evals`.

| Step | Description |
|---|---|
| Verify sandbox credentials | Checks that required secrets are set for the chosen sandbox and model provider |
| Checkout | `actions/checkout@v6` |
| Set up Python 3.12 + uv | `./.github/actions/uv_setup` with `harbor` cache suffix |
| Install dependencies | `uv sync --group test --locked` |
| Create LangSmith experiment | `python scripts/harbor_langsmith.py create-experiment terminal-bench` — outputs experiment name |
| Suppress Harbor tips | Writes a notification cache file to silence first-run tips |
| Run Harbor | `uv run harbor run --agent-import-path deepagents_harbor:DeepAgentsWrapper --dataset terminal-bench@2.0 ...` with concurrency and agent mode flags |
| Find latest Harbor job | Python snippet that locates the newest job directory under `jobs/terminal-bench/` |
| Add Harbor rewards to LangSmith | `python scripts/harbor_langsmith.py add-feedback <job_dir>` — posts pass/fail rewards to the experiment |
| Write workflow summary | Appends run metadata (model, dataset, sandbox, concurrency, etc.) to the step summary |
| Upload Harbor artifacts | Uploads `libs/evals/jobs/terminal-bench` as `harbor-<index>` artifact |

## Notes

- Harbor is a terminal-bench evaluation harness for AI coding agents.
- The `deepagents_harbor:DeepAgentsWrapper` is an integration shim that adapts the deepagents agent to Harbor's interface.
- The `--agent-kwarg use_cli_agent=true` flag switches between SDK mode and CLI mode.
- Jobs time out after 6 hours (`timeout-minutes: 360`), accommodating large task sets.
