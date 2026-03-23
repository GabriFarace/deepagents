# `libs/evals/Makefile`

## Purpose

Development workflow automation for the `deepagents-evals` package. Covers unit testing, Harbor eval execution across multiple sandbox providers, radar chart generation, linting, formatting, and type checking.

## Targets

### Testing

| Target | Description |
|--------|-------------|
| `test` | Run unit tests from `tests/unit_tests/` with socket isolation (`--disable-socket --allow-unix-socket`) |
| `evals` | Run eval tests from `tests/evals/` with `LANGSMITH_TEST_SUITE=deepagents-evals` |
| `test_watch` | Run tests in watch mode using `ptw` |

### Harbor Eval Runs

All Harbor targets use `--agent-import-path deepagents_harbor:DeepAgentsWrapper`. The `-n` flag controls concurrent sandbox slots (parallel trials), not task count. `AGENT_MODE` (default `cli`) controls whether to use the CLI agent or SDK agent via `--agent-kwarg use_cli_agent=true/false`.

| Target | Dataset | Concurrency | Environment |
|--------|---------|-------------|-------------|
| `run-hello-world` | `hello-world` | 1 | docker |
| `run-terminal-bench-modal` | `terminal-bench@2.0` | 4 | modal |
| `run-terminal-bench-daytona` | `terminal-bench@2.0` | 40 | daytona |
| `run-terminal-bench-docker` | `terminal-bench@2.0` | 1 | docker |
| `run-terminal-bench-runloop` | `terminal-bench@2.0` | 10 | runloop |

### Charts

| Target | Description |
|--------|-------------|
| `radar` | Generate radar chart with toy data; output to `RADAR_OUTPUT` (default `charts/radar.png`) |
| `radar-from-summary` | Generate radar chart from `SUMMARY_JSON` (default `evals_summary.json`) |

### Linting and Formatting

| Target | Description |
|--------|-------------|
| `lint` | Ruff format (diff) + Ruff lint (diff) + type checker on `deepagents_evals/`, `deepagents_harbor/`, `tests/` |
| `lint_diff` | Same as lint but only on files changed relative to `main` |
| `type` / `typecheck` | Run `ty check` on source packages and unit tests |
| `format` | Run Ruff formatter + auto-fix linting issues |
| `format_diff` | Same as format but only on changed files |
| `format_unsafe` | Run Ruff formatter with unsafe fixes |
| `help` | Display available targets |

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TEST_FILE` | `tests/unit_tests` | Override test path for `test` target |
| `PYTEST_EXTRA` | (empty) | Additional pytest arguments |
| `AGENT_MODE` | `cli` | Harbor agent mode: `cli` (local) or `sdk` (CI) |
| `RADAR_OUTPUT` | `charts/radar.png` | Output path for radar chart |
| `SUMMARY_JSON` | `evals_summary.json` | Input summary JSON for `radar-from-summary` |
| `LINT` | (empty) | Set to `minimal` to skip Ruff lint check in lint target |
