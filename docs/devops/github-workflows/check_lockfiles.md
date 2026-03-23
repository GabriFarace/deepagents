# Workflow: `check_lockfiles.yml` — Check Lockfiles

## Overview

Verifies that all `uv.lock` files across the monorepo are up-to-date with their corresponding `pyproject.toml` files. Prevents PRs from being merged when lockfiles are out of sync.

## Trigger

- `push` to `main` branch
- `pull_request` (all)
- `merge_group` events

Concurrent runs for the same workflow/ref are cancelled (`cancel-in-progress: true`).

## Jobs

### `check-lockfiles`

Runs on `ubuntu-latest` with a 5-minute timeout.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python and uv | `./.github/actions/uv_setup` with Python 3.14 |
| Check all lockfiles | `make lock-check` (runs `uv lock --check` on every package directory) |

## Permissions

- `contents: read`

## How It Works

`make lock-check` iterates over all package directories containing a `Makefile` (found via glob) and runs `uv lock --check` for each, using Python 3.14 for `libs/acp` and 3.12 for all others. If any lockfile is stale the step fails.

## Notes

- The corresponding fix command is `make lock` — regenerates all lockfiles.
- Lockfiles are also automatically updated by the `release-please.yml` workflow when release-please creates or updates a release PR.
