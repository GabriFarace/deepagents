# Workflow: `check_sdk_pin.yml` — Check SDK Pin

## Overview

An advisory check that posts a warning comment on CLI release PRs when the `deepagents` SDK pin in `libs/cli/pyproject.toml` does not match the actual SDK version in `libs/deepagents/pyproject.toml`. This workflow does **not** block merging — the hard gate is enforced at publish time in `release.yml`. The comment is automatically removed once the versions are reconciled.

## Trigger

`pull_request` events touching:
- `libs/deepagents/pyproject.toml`
- `libs/cli/pyproject.toml`

Only runs for PRs whose branch name starts with `release-please--branches--main--components--deepagents-cli`.

Concurrent runs for the same workflow/ref are cancelled.

## Jobs

### `check-sdk-pin`

Runs on `ubuntu-latest` with a 2-minute timeout.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Compare SDK version to CLI pin | Bash script extracts SDK version from `libs/deepagents/pyproject.toml` and CLI pin from `libs/cli/pyproject.toml`, emits outputs `sdk_version`, `cli_pin`, `match` |
| Manage PR comment | `actions/github-script@v8` posts a warning comment (or removes it if versions match) |

## Permissions

- `contents: read`
- `pull-requests: write` — required to post/update/delete PR comments

## Comment Behavior

| Scenario | Action |
|---|---|
| Versions match, no existing comment | No action |
| Versions match, stale comment exists | Deletes the stale comment |
| Version mismatch, no comment | Posts warning comment with mismatch table |
| Version mismatch, comment exists | Updates the existing comment |

The comment contains an HTML marker `<!-- sdk-pin-check -->` to allow idempotent management across re-pushes.

## Warning Comment Contents

When a mismatch is detected, the comment shows:
- A table comparing the SDK version vs. CLI pin
- Instructions to fix: update `libs/cli/pyproject.toml` to pin `deepagents==<sdk_version>`, then run `cd libs/cli && uv lock`
- A bypass option via the `dangerous-skip-sdk-pin-check` flag on `release.yml`
- A link to `.github/RELEASING.md` for the full recovery procedure

## Notes

- The advisory nature means the PR can still be merged even with a mismatch, but `release.yml` will fail at the "Verify CLI pins latest SDK version" step when attempting to publish.
- See `release.yml` for the hard enforcement gate.
