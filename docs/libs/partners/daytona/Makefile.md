# `libs/partners/daytona/Makefile`

## Purpose

Development workflow automation for the `langchain-daytona` package. Identical in structure to the `modal` and `runloop` partner Makefiles, differing only in the package name (`langchain_daytona`).

## Targets

| Target | Description |
|--------|-------------|
| `test` / `tests` | Run unit tests from `tests/unit_tests/` with socket isolation (`--disable-socket --allow-unix-socket`) |
| `integration_test` / `integration_tests` | Run integration tests from `tests/integration_tests/` with a 30-second timeout |
| `test_watch` | Run tests in watch mode using `ptw` |
| `benchmark` | Run tests marked with `benchmark` |
| `lint` | Run Ruff linter + formatter (diff mode) + type checker on all Python files |
| `lint_diff` | Same as lint but only on files changed relative to `main` |
| `lint_package` | Lint only the `langchain_daytona` package directory |
| `type` / `typecheck` | Run `ty check langchain_daytona` |
| `format` | Run Ruff formatter and auto-fix linting issues |
| `format_diff` | Same as format but only on changed files |
| `help` | Display available targets |

## Environment

- `UV_FROZEN = true` — All uv commands use locked dependency versions.
- Default goal: `help`
