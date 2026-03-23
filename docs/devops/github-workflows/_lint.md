# Workflow: `_lint.yml` — Linting (Reusable)

## Overview

A reusable workflow that runs linters for a given package directory. Called by the main CI workflow (`ci.yml`) for each changed package.

## Trigger

`workflow_call` only. Callers pass:

| Input | Required | Description |
|---|---|---|
| `working-directory` | Yes | Package directory to lint (e.g. `libs/deepagents`) |
| `python-version` | Yes | Python version to use |

## Jobs

### `build`

Runs on `ubuntu-latest` with a 20-minute timeout.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python + uv | `./.github/actions/uv_setup` with `lint-<working-directory>` cache suffix |
| Install dependencies | `uv sync --group test` in the working directory |
| Run linters | `make lint` in the working directory |

## Permissions

- `contents: read` (global)

## Environment Variables

- `WORKDIR` — resolved working directory (defaults to `.` if empty)
- `RUFF_OUTPUT_FORMAT: github` — formats Ruff output as GitHub annotations
- `LINT: minimal` — sets lint level
- `UV_FROZEN: "true"` — prevents uv from modifying the lockfile

## Notes

- Linting is performed by invoking `make lint` in the package directory. The actual lint commands (typically Ruff) are defined in each package's own `Makefile`.
- This workflow is called once per changed package in a CI run.
