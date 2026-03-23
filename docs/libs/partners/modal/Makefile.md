# `libs/partners/modal/Makefile`

## Purpose

Development workflow automation for the `langchain-modal` package. Provides standardized targets for testing, linting, type checking, and formatting using `uv` with frozen dependencies.

## Targets

| Target | Description |
|--------|-------------|
| `test` / `tests` | Run unit tests from `tests/unit_tests/` using pytest with socket isolation (`--disable-socket --allow-unix-socket`) |
| `integration_test` / `integration_tests` | Run integration tests from `tests/integration_tests/` with a 30-second timeout |
| `test_watch` | Run tests in watch mode using `ptw` (pytest-watcher) |
| `benchmark` | Run tests marked with `benchmark` |
| `lint` | Run Ruff linter + formatter (diff mode) + type checker on all Python files |
| `lint_diff` | Same as lint but only on files changed relative to `main` |
| `lint_package` | Lint only the `langchain_modal` package directory |
| `type` / `typecheck` | Run `ty check langchain_modal` |
| `format` | Run Ruff formatter and auto-fix linting issues |
| `format_diff` | Same as format but only on changed files |
| `help` | Display available targets with descriptions |

## Environment

- `UV_FROZEN = true` — All uv commands use locked dependency versions.
- Default goal: `help`
- Test file override: `make test TEST_FILE=tests/unit_tests/test_specific.py`
