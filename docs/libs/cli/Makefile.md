# `Makefile` — Build and Development Commands

## High-Level Purpose

This Makefile defines development workflow commands for the `deepagents-cli` package. It uses `uv` as the package manager and runner, `ruff` for linting and formatting, `ty` for type checking, and `pytest` for testing.

The default target is `help`.

## Global Settings

```makefile
.EXPORT_ALL_VARIABLES:
UV_FROZEN = true
```

`UV_FROZEN = true` is exported to all subprocesses, ensuring that `uv run` uses the locked dependency versions from `uv.lock` and doesn't auto-update.

## Testing Targets

| Target | Command | Description |
|---|---|---|
| `make test` or `make tests` | `uv run --group test pytest -n auto --disable-socket --allow-unix-socket $(PYTEST_EXTRA) tests/unit_tests/ $(COV_ARGS)` | Run unit tests with coverage (parallel, network disabled) |
| `make coverage` | `uv run --group test pytest --cov --cov-config=.coveragerc --cov-report xml --cov-report term-missing:skip-covered` | Run unit tests with XML+terminal coverage reports |
| `make integration_test` | `uv run --group test pytest -n auto -vvv --timeout 30 tests/integration_tests/` | Run integration tests with 30s timeout |
| `make test_watch` | `uv run --group test ptw --now . -- -vv tests/unit_tests/` | Run tests in watch mode (re-runs on file changes) |
| `make benchmark` | `uv run --group test pytest ./tests -m benchmark` | Run benchmark-marked tests only |

### Test Variables

| Variable | Default | Description |
|---|---|---|
| `TEST_FILE` | `tests/unit_tests/` | Path to test directory/file |
| `COV_ARGS` | `--cov=deepagents_cli --cov-report=term-missing` | Coverage arguments |
| `PYTEST_EXTRA` | (empty) | Additional pytest arguments |

**Override examples:**
```bash
make test TEST_FILE=tests/unit_tests/test_sessions.py
make test PYTEST_EXTRA=-v
```

## Linting and Formatting Targets

| Target | Description |
|---|---|
| `make lint` | Run ruff check + format diff + type check on all files |
| `make lint_diff` | Lint only files changed relative to `main` branch |
| `make lint_package` | Lint only `deepagents_cli/` (not tests) |
| `make lint_tests` | Lint only `tests/` |
| `make format` | Auto-fix: run ruff format + ruff check --fix |
| `make format_diff` | Auto-fix only changed files |
| `make type` or `make typecheck` | Run `ty check` (type checker) |
| `make check_imports` | Verify imports across all Python files via `scripts/check_imports.py` |

**`lint_diff` and `format_diff`** use `git diff --relative=libs/cli --name-only --diff-filter=d main` to find only modified Python files, making CI fast on PRs.

## Other Targets

| Target | Command | Description |
|---|---|---|
| `make run` | `uvx --no-cache --reinstall .` | Reinstall the package and run it |
| `make help` | (built-in) | Show all available targets with descriptions |

## Toolchain

| Tool | Purpose |
|---|---|
| `uv` | Package management and script running |
| `ruff` | Fast Python linter and formatter |
| `ty` | Type checker |
| `pytest` | Test runner |
| `pytest-xdist` | Parallel test execution (`-n auto`) |
| `pytest-watch` | Watch mode for tests |
| `pytest-cov` | Coverage reporting |
