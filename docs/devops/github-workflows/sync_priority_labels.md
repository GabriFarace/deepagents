# Workflow: `sync_priority_labels.yml` — Sync Priority Labels

## Overview

Synchronizes priority labels (`p0`, `p1`, `p2`, `p3`) from linked issues to PRs. When a PR body references an issue with a priority label (via `Fixes #NNN`, `Closes #NNN`, etc.), the corresponding priority is copied to the PR. Priority labels are mutually exclusive — when a PR links to multiple issues with different priorities, the highest priority wins (`p0 > p1 > p2 > p3`).

## Trigger

- `pull_request_target` events: `opened`, `edited` — syncs priority from linked issue(s) to the PR
- `issues` events: `labeled`, `unlabeled` — propagates priority changes to all open PRs that link to the issue (only for `p0`–`p3` labels)
- `workflow_dispatch` — manual backfill of all open PRs (up to `max_items`, default 200)

## Concurrency

Serialized per PR (on PR events), per issue (on issue events), or globally (backfill). `cancel-in-progress: true` except for `workflow_dispatch`.

## Jobs

### 1. `sync-from-issue` (PR opened/edited)

Runs when `event_name == 'pull_request_target'`.

1. Parses issue numbers from PR body using regex for closing keywords
2. Fetches labels for each linked issue
3. Determines highest priority (`p0` wins over `p1`, etc.)
4. Removes stale priority labels from the PR
5. Applies the winning priority label (if any)

### 2. `sync-to-prs` (Issue labeled/unlabeled)

Runs when `event_name == 'issues'` and the changed label is `p0`–`p3`.

1. Searches GitHub for open PRs referencing the issue (using `is:pr is:open "<issue_number>"`)
2. Filters results to PRs with actual closing keyword references (to avoid false positives)
3. For each matching PR, re-derives the full correct priority by checking **all** linked issues
4. Applies the highest-priority label, removing stale ones

### 3. `backfill` (Manual)

Runs when triggered via `workflow_dispatch`.

Paginates through all open PRs (up to `max_items`) and reconciles priority labels. Reports counts of processed, updated, and failed PRs.

## Priority Label Values

| Label | Priority |
|---|---|
| `p0` | Highest |
| `p1` | High |
| `p2` | Medium |
| `p3` | Low |

## Notes

- Priority labels are created automatically (color `b76e79`) if they don't exist.
- Uses `pull_request_target` (not `pull_request`) to safely write labels without executing PR code.
- The search-based approach for issue→PR sync may return false positives for low issue numbers; the regex filter prunes them.
