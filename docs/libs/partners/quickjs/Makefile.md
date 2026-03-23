# `libs/partners/quickjs/Makefile`

## Purpose

Development workflow automation for the `langchain-quickjs` package. Similar structure to other partner Makefiles but adds coverage reporting, an `update_snapshots` target for smoke tests, and runs integration tests in parallel.

## Targets

| Target | Description |
|--------|-------------|
| `test` / `tests` | Run unit tests with coverage (`--cov=langchain_quickjs --cov-report=term-missing`), socket isolation (`--disable-socket --allow-unix-socket`) |
| `coverage` | Run full coverage report with XML output and term-missing:skip-covered format |
| `integration_test` / `integration_tests` | Run integration tests in parallel (`-n auto`) with a 30-second timeout |
| `test_watch` | Run tests in watch mode using `ptw` |
| `update_snapshots` | Update smoke test prompt snapshots (`tests/unit_tests/smoke_tests/test_system_prompt.py --update-snapshots`) |
| `lint` | Run Ruff linter + formatter (diff mode) + type checker on all Python files |
| `lint_diff` | Same as lint but only on files changed relative to `main` |
| `lint_package` | Lint only the `langchain_quickjs` package directory |
| `type` / `typecheck` | Run `ty check langchain_quickjs` |
| `format` | Run Ruff formatter and auto-fix linting issues |
| `format_diff` | Same as format but only on changed files |
| `help` | Display available targets with descriptions |

## Key Differences from Other Partner Makefiles

- Unit tests always run with coverage enabled.
- Integration tests use `-n auto` for parallel execution.
- Has a `coverage` target for detailed XML + terminal coverage reports.
- Has `update_snapshots` for maintaining smoke test baselines of system prompt generation.
- No `benchmark` target.

## Environment

- `UV_FROZEN = true` — All uv commands use locked dependency versions.
- Default goal: `help`
