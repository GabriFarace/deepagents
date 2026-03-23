# Workflow: `pr_labeler_backfill.yml` — PR Labeler Backfill

## Overview

A manual-only workflow that retroactively applies the same labels as `pr_labeler.yml` (size, file, title, contributor classification) to all currently open PRs. Uses the same shared logic from `.github/scripts/pr-labeler.js`.

## Trigger

`workflow_dispatch` only. Input:

| Input | Default | Description |
|---|---|---|
| `max_items` | `100` | Maximum number of open PRs to process |

## Secrets Required

| Secret | Purpose |
|---|---|
| `ORG_MEMBERSHIP_APP_ID` | GitHub App ID |
| `ORG_MEMBERSHIP_APP_PRIVATE_KEY` | GitHub App private key |

## Jobs

### `backfill`

Runs on `ubuntu-latest` with `pull-requests: write` and `issues: write` permissions.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Generate GitHub App token | `actions/create-github-app-token@v3` |
| Backfill labels on open PRs | `actions/github-script@v8` — processes up to `max_items` open PRs |

## Backfill Logic

For each open PR (up to `max_items`):
1. Checks org membership of the PR author (with cache to avoid repeated API calls)
2. Applies `external` or `internal` label
3. Applies contributor tier label (`trusted-contributor` if 5+ merged PRs, `new-contributor` if 0)
4. Computes PR size and applies the appropriate `size: *` label
5. Applies file-based labels from `pr-labeler-config.json` rules
6. Applies title-based labels from the PR title (Conventional Commits format)
7. Removes stale managed labels (size, tier, type) that no longer apply

Failures on individual PRs are logged as warnings but do not stop the backfill.

## Notes

- Use this workflow after making changes to labeling rules to retroactively sync existing PRs.
- The contributor cache is shared across all PRs in the run to minimize API calls.
