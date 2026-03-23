# `Makefile`

## High-Level Purpose

The Makefile provides developer workflow automation for the `deepagents` library. It covers testing, coverage, benchmarking, linting, formatting, type checking, and package execution. All commands use `uv` as the package manager and task runner.

## Global Settings

```makefile
.DEFAULT_GOAL := help       # Running `make` with no args shows help
.EXPORT_ALL_VARIABLES:
UV_FROZEN = true            # Pin dependency versions via uv.lock
```

## Targets

### Testing

| Target | Command | Description |
|---|---|---|
| `test` / `tests` | `uv run --group test pytest -n auto -vvv --disable-socket --allow-unix-socket <TEST_FILE>` | Run unit tests in parallel with network isolation and coverage reporting |
| `integration_test` / `integration_tests` | Same but `TEST_FILE=tests/integration_tests/` | Run integration tests (network enabled, 30s timeout) |
| `coverage` | `pytest --cov --cov-report xml ...` | Generate XML and terminal coverage reports |
| `test_watch` | `ptw --now . -- -vv <TEST_FILE>` | Run tests in watch mode (auto-rerun on file changes) |
| `benchmark` | `pytest ./tests -m benchmark` | Run only benchmark-marked tests |
| `update-snapshots` | `pytest --update-snapshots <SMOKE_TESTS>` | Update snapshot files for smoke tests |

**Variables:**
- `TEST_FILE` — Defaults to `tests/unit_tests/`. Override: `make test TEST_FILE=tests/unit_tests/test_foo.py`
- `PYTEST_EXTRA` — Extra pytest flags: `make test PYTEST_EXTRA="--timeout=60"`
- `SMOKE_TESTS` — Defaults to `tests/unit_tests/smoke_tests/`

**Network isolation:** Unit tests use `--disable-socket --allow-unix-socket` to block network calls while allowing Unix domain sockets (needed for some async I/O). Integration tests do not restrict networking.

### Linting and Formatting

| Target | Description |
|---|---|
| `lint` | Run ruff check + ruff format check + type check (on all `.py` files) |
| `lint_diff` | Same but only on files changed relative to `main` branch |
| `lint_package` | Lint only the `deepagents/` source package |
| `lint_tests` | Lint only the `tests/` directory |
| `format` | Auto-format with ruff (fix + format) |
| `format_diff` | Format only changed files |
| `type` / `typecheck` | Run `ty check deepagents` |
| `check_imports` | Run `scripts/check_imports.py` on all package Python files |

**Ruff commands used:**
- `ruff check <files>` — linting
- `ruff format <files> --diff` — format check (no changes)
- `ruff format <files>` — format in place
- `ruff check --fix <files>` — auto-fix lint issues

**`PYTHON_FILES` variable:** Controls which files are linted/formatted. Computed by `git diff` for `_diff` targets.

### Other

| Target | Command | Description |
|---|---|---|
| `run` | `uvx --no-cache --reinstall .` | Reinstall package from source and run |
| `help` | `awk` on Makefile | Show all documented targets with descriptions |

## Usage Examples

```bash
# Run all unit tests
make test

# Run a specific test file
make test TEST_FILE=tests/unit_tests/backends/test_state.py

# Run integration tests
make integration_tests

# Check and fix all linting issues
make format

# Check types only
make type

# Run tests continuously while developing
make test_watch
```
