# `Makefile` — deepagents-acp

## Overview

The Makefile provides development workflow targets for the `deepagents-acp` package. All targets run commands through `uv run` for isolated environment management.

## Targets

### `test`
```
uv run pytest --disable-socket --allow-unix-socket $(TEST_FILE) --timeout 10 \
  --cov=deepagents_acp --cov-report=term-missing --cov-report=xml
```
Runs the test suite with:
- Network sockets disabled (`--disable-socket`) except Unix sockets
- Per-test timeout of 10 seconds
- Coverage reporting for the `deepagents_acp` package (terminal + XML)

**Variables:**
- `TEST_FILE` (default: `tests/`) — Path to the test file or directory
- `PYTEST_EXTRA` — Additional pytest arguments

### `test_watch`
Runs `pytest-watcher` (`ptw`) in watch mode. Re-runs tests on file changes.

### `toad`
```
uv run toad acp 'bash ./run.sh'
```
Starts the ACP server using the `toad` CLI tool (ACP development server).

### `lint` / `lint_diff`
- Runs `ruff format --diff` (check only) and `ruff check` on the source
- Also runs the `type` target

**`lint_diff`** restricts to files changed relative to `main`.

### `type` / `typecheck`
```
uv run --group test ty check deepagents_acp
```
Runs the `ty` type checker on the `deepagents_acp` package.

### `format` / `format_diff`
- Runs `ruff format` and `ruff check --fix` to auto-format and fix lint issues.
- `format_diff` restricts to files changed relative to `main`.

### `help`
Prints a formatted list of available targets and their descriptions.
