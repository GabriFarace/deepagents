# Workflow: `_test.yml` — Unit Testing (Reusable)

## Overview

A reusable workflow that runs unit tests for a given package directory. Called by the main CI workflow (`ci.yml`) for each changed package across a matrix of Python versions. Each package now renders as a **single parent row** in the Actions UI with child rows per version/OS leg, instead of one top-level row per version.

## Trigger

`workflow_call` only. Callers pass:

| Input | Required | Default | Description |
|---|---|---|---|
| `working-directory` | Yes | — | Package directory (e.g. `libs/deepagents`) |
| `python-versions` | Yes | — | JSON array of Python versions as quoted strings (e.g. `'["3.11","3.12"]'`). Parsed via `fromJSON()` into `strategy.matrix.python-version`. |
| `os` | No | `ubuntu-latest` | Primary runner OS; paired with every `python-versions` entry. |
| `extra-configurations` | No | `[]` | JSON array of additional `{python-version, os}` legs merged via `matrix.include`. Used to add platform-specific legs (e.g. Windows) without a standalone job. |
| `coverage-python-version` | No | `""` | Python version of the leg on which to collect coverage. Empty disables coverage for all legs. |
| `coverage-os` | No | `""` | Runner OS of the coverage leg. Defaults to the primary `os` input when empty. |

> **Note:** The previous `python-version` (singular) and boolean `coverage` inputs have been replaced by `python-versions` (JSON array) and the `coverage-python-version` + `coverage-os` pair. This avoids coverage collisions when a version appears on multiple OSes.

## Jobs

### `validate-inputs`

Validates the `python-versions` and `extra-configurations` JSON arrays using `jq` before the matrix is built. Fails loudly if:
- Any entry in `python-versions` is not a quoted string.
- Any entry in `extra-configurations` is missing the `python-version` or `os` fields (both must be strings).
- The configured coverage leg is not among the real matrix legs.

### `build`

Runs on the matrix of `os × python-versions` (plus any `extra-configurations` entries) with a 20-minute timeout. Job name is `Python <ver> / <os>`. Default working directory is set to the package directory.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python + uv | `./.github/actions/uv_setup` with `test-<working-directory>` cache suffix |
| Install test dependencies | `uv sync --group test` |
| Run unit tests | `make test` (with `COV_ARGS=` to skip coverage on non-coverage legs; `PYTEST_EXTRA=-q` for quieter output) |
| Verify clean working directory | Fails if any files were modified during the test run |

Matrix values and inputs are passed through `env:` variables (`$PY`, `$COV_PY`, `$PY_OS`, `$COV_OS`) rather than inline `${{ }}` in `run:` bodies, closing a template-injection sink on a reusable workflow.

## Permissions

- `contents: read` (global)

## Environment Variables

- `UV_NO_SYNC: "true"` — prevents auto-sync
- `UV_FROZEN: "true"` — prevents lockfile modifications
- `RUN_SANDBOX_TESTS: "true"` — enables sandbox test execution in the test environment

## Coverage Strategy

Coverage is collected only on the leg matching **both** `coverage-python-version` and `coverage-os`. Callers in `ci.yml` typically set `coverage-python-version: "3.12"` with the default OS, so only one leg per package generates a coverage report.

## Post-Test Verification

After running tests, the workflow checks that `git status` reports a clean working directory (`nothing to commit, working tree clean`). This catches tests that accidentally write files.
