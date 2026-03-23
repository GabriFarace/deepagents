# Pre-Commit Configuration

## Overview

The `.pre-commit-config.yaml` file defines git pre-commit hooks that enforce code quality, file hygiene, and consistency checks before every local commit. These hooks mirror many of the same checks run in CI, catching issues early in the developer workflow.

## Location

`/.pre-commit-config.yaml`

## Installation

```bash
pip install pre-commit
pre-commit install
```

## Hooks

### From `pre-commit/pre-commit-hooks` (v4.3.0)

| Hook ID | Description |
|---|---|
| `no-commit-to-branch` | Prevents direct commits to `main` |
| `check-yaml` | Validates YAML syntax (with `--unsafe` to allow custom tags) |
| `check-toml` | Validates TOML syntax |
| `end-of-file-fixer` | Ensures all files end with a newline (excludes `libs/evals/tests/evals/tau2_airline/data/`) |
| `trailing-whitespace` | Removes trailing whitespace (excludes `.ambr` files and tau2 airline data) |

### From `sirosen/texthooks` (v0.6.8)

| Hook ID | Description |
|---|---|
| `fix-smartquotes` | Replaces curly/smart quotes with straight ASCII quotes (excludes tau2 airline data) |
| `fix-spaces` | Replaces non-standard Unicode spaces with regular spaces (excludes tau2 airline data) |

### Local Hooks

#### `deepagents` — Format and lint deepagents SDK

- **Entry:** `make -C libs/deepagents format lint`
- **Trigger:** Changes to files under `libs/deepagents/`
- Runs both format and lint to keep code consistent

#### `deepagents-cli` — Format and lint CLI

- **Entry:** `make -C libs/cli format lint`
- **Trigger:** Changes to files under `libs/cli/`

#### `evals` — Format and lint evals

- **Entry:** `make -C libs/evals format lint`
- **Trigger:** Changes to files under `libs/evals/`

#### `acp` — Format and lint ACP

- **Entry:** `make -C libs/acp format lint`
- **Trigger:** Changes to files under `libs/acp/`

#### `lock-check` — Check lockfiles are up-to-date

- **Entry:** `make lock-check`
- **Trigger:** Changes to any `libs/*/pyproject.toml` or `libs/*/uv.lock`
- Fails if any lockfile is stale (run `make lock` to fix)

#### `extras-sync` — Check extras sync with required deps

- **Entry:** `python3 .github/scripts/check_extras_sync.py libs/cli/pyproject.toml`
- **Trigger:** Changes to `libs/cli/pyproject.toml`
- Ensures optional extras version constraints match required deps

#### `version-equality` — Check pyproject.toml and _version.py match

- **Entry:** `python3 .github/scripts/check_version_equality.py`
- **Trigger:** Changes to version files in `libs/deepagents/` or `libs/cli/`
- Fails if `pyproject.toml` version differs from `_version.py`

## Notes

- All local hooks use `pass_filenames: false` — they run the full command regardless of which specific files changed in the trigger set.
- The `language: system` setting means hooks use the system Python/tools rather than a pre-commit managed environment, so developer tools (`uv`, `make`) must be installed.
- Tau2 airline test data is excluded from text-fixing hooks because it may intentionally contain non-standard characters.
