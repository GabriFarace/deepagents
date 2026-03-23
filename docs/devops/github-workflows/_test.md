# Workflow: `_test.yml` — Unit Testing (Reusable)

## Overview

A reusable workflow that runs unit tests for a given package directory. Called by the main CI workflow (`ci.yml`) for each changed package, often across a Python version matrix.

## Trigger

`workflow_call` only. Callers pass:

| Input | Required | Default | Description |
|---|---|---|---|
| `working-directory` | Yes | — | Package directory (e.g. `libs/deepagents`) |
| `python-version` | Yes | — | Python version to use |
| `coverage` | No | `true` | Whether to collect coverage. Disabled for non-primary matrix legs to speed up CI. |

## Jobs

### `build`

Runs on `ubuntu-latest` with a 20-minute timeout. Default working directory is set to the package directory.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python + uv | `./.github/actions/uv_setup` with `test-<working-directory>` cache suffix |
| Install test dependencies | `uv sync --group test` |
| Run unit tests | `make test` (optionally with `COV_ARGS=` to skip coverage, `PYTEST_EXTRA=-q` for quieter output) |
| Verify clean working directory | Fails if any files were modified during the test run |

## Permissions

- `contents: read` (global)

## Environment Variables

- `UV_NO_SYNC: "true"` — prevents auto-sync
- `UV_FROZEN: "true"` — prevents lockfile modifications
- `RUN_SANDBOX_TESTS: "true"` — enables sandbox test execution in the test environment

## Coverage Strategy

Coverage is only collected when `coverage` input is `true`. In `ci.yml`, coverage is enabled only for Python 3.12 to avoid redundant overhead across the full version matrix.

## Post-Test Verification

After running tests, the workflow checks that `git status` reports a clean working directory (`nothing to commit, working tree clean`). This catches tests that accidentally write files.
