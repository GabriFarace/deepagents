# Workflow: `tag-external-issues.yml` — Tag External Issues

## Overview

Automatically classifies newly opened GitHub issues as `external` or `internal` based on whether the author is an active member of the `langchain-ai` GitHub organization. For external contributors, also applies contributor tier labels based on their merged PR history. Includes a manual backfill job for retroactively labeling open issues.

## Trigger

- `issues` events: `opened`
- `workflow_dispatch` with `max_items` input (default 100) — for backfilling open issues

## Secrets Required

| Secret | Purpose |
|---|---|
| `ORG_MEMBERSHIP_APP_ID` | GitHub App ID |
| `ORG_MEMBERSHIP_APP_PRIVATE_KEY` | GitHub App private key |

A GitHub App with `issues: write` and `organization members: read` permissions is required to check private org membership.

## Jobs

### `tag-external` (Issue Opened)

Runs when `event_name == 'issues'` with `issues: write` permission.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Generate GitHub App token | `actions/create-github-app-token@v3` |
| Check if contributor is external | Calls `h.checkMembership()` from `pr-labeler.js` |
| Apply contributor tier label | For external contributors: applies `trusted-contributor` (5+ merged PRs). Skips `new-contributor` (not meaningful on issues). |
| Add external/internal label | Adds `external` or `internal` label |

### `backfill` (Manual)

Runs when `event_name == 'workflow_dispatch'`.

Paginates through open issues (skipping PRs), and for each:
- Checks org membership (with cache)
- Applies `external`/`internal` label
- Applies `trusted-contributor` if applicable
- Removes stale tier labels

## Contributor Tiers

| Condition | Label |
|---|---|
| External, 5+ merged PRs | `trusted-contributor` |
| Internal (org member) | `internal` |
| External | `external` |

Note: `new-contributor` is intentionally skipped for issues (only applied on PRs by `pr_labeler.yml`).

## Relationship to `pr_labeler.yml`

- Issues are classified here; PRs are classified in `pr_labeler.yml`
- Both workflows use the same `pr-labeler.js` helper functions
- This workflow does **not** classify PRs to avoid race conditions with concurrent label mutations
