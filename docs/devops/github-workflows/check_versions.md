# Workflow: `check_versions.yml` — Check Version Equality

## Overview

Ensures that version numbers in `pyproject.toml` and `_version.py` remain in sync for both the SDK and CLI packages. Prevents releases with mismatched version numbers.

## Trigger

`pull_request` events touching any of:
- `libs/deepagents/pyproject.toml`
- `libs/deepagents/deepagents/_version.py`
- `libs/cli/pyproject.toml`
- `libs/cli/deepagents_cli/_version.py`

Concurrent runs for the same workflow/ref are cancelled.

## Jobs

### `check_version_equality`

Runs on `ubuntu-latest` with a 2-minute timeout.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python and uv | `./.github/actions/uv_setup` with Python 3.14, no cache |
| Verify pyproject.toml and _version.py match | `python .github/scripts/check_version_equality.py` |

## Permissions

- `contents: read`

## Checked Packages

| `pyproject.toml` | `_version.py` |
|---|---|
| `libs/deepagents/pyproject.toml` | `libs/deepagents/deepagents/_version.py` |
| `libs/cli/pyproject.toml` | `libs/cli/deepagents_cli/_version.py` |

## Dependencies

Calls `.github/scripts/check_version_equality.py`. See [check_version_equality.py documentation](../github-scripts/check_version_equality.md) for details.

## Notes

- This check is also run as a pre-commit hook via `.pre-commit-config.yaml`.
- Only triggers when version-bearing files are modified, keeping CI fast.
