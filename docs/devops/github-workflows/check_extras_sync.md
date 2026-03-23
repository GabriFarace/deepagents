# Workflow: `check_extras_sync.yml` — Check Extras Sync

## Overview

Ensures that optional extras in `[project.optional-dependencies]` stay in sync with required dependencies in `[project.dependencies]` for the CLI package. When a package appears in both sections, the version constraints must match to prevent silent version drift.

## Trigger

- `pull_request` events targeting `libs/cli/pyproject.toml`
- `push` to `main` branch touching `libs/cli/pyproject.toml`

Concurrent runs for the same workflow/ref are cancelled (`cancel-in-progress: true`).

## Jobs

### `check-extras-sync`

Runs on `ubuntu-latest` with a 2-minute timeout.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python and uv | `./.github/actions/uv_setup` with Python 3.14, no cache |
| Check extras sync | `python .github/scripts/check_extras_sync.py libs/cli/pyproject.toml` |

## Permissions

- `contents: read`

## Dependencies

Calls `.github/scripts/check_extras_sync.py`. See [check_extras_sync.py documentation](../github-scripts/check_extras_sync.md) for details on the script.

## Notes

- Only triggers when `libs/cli/pyproject.toml` is modified, making it an efficient targeted check.
- Caching is disabled (`enable-cache: "false"`) to keep this fast (2-minute timeout).
- This same check is also run as a pre-commit hook via `.pre-commit-config.yaml`.
