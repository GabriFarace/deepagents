# Workflow: `require_issue_link.yml` — Require Issue Link

## Overview

Enforces that external contributors reference an approved GitHub issue in their PR body and are assigned to that issue. PRs that fail this check are labeled `missing-issue-link`, commented on with instructions, and automatically closed. Maintainers can bypass the check by reopening the PR or removing the `missing-issue-link` label.

## Trigger

`pull_request_target` events: `edited`, `reopened`, `labeled`, `unlabeled`.

Does **not** trigger on `opened` — new PRs have no labels yet, so the `external` label applied by `pr_labeler.yml` triggers this check on the subsequent `labeled` event.

## Enforcement Gate

Controlled by the `ENFORCE_ISSUE_LINK: "true"` environment variable. When set to `false`, the check logic runs (for dry-run visibility) but will not label, comment, close, or fail PRs.

## Job Conditions

The `check-issue-link` job runs when:
- The `external` label was just added to the PR, OR
- The `missing-issue-link` label was just removed (maintainer override check), OR
- The PR was edited/reopened and already has the `external` label

The job is **skipped** when the PR has `trusted-contributor` or `bypass-issue-check` labels.

## Jobs

### `check-issue-link`

Runs on `ubuntu-latest` with `pull-requests: write` and `actions: write`.

| Step | Description |
|---|---|
| Check for issue link and assignee | Core check (see below) |
| Add missing-issue-link label | If check failed (enforcement on) |
| Remove missing-issue-link label and reopen | If check passed and PR was previously closed |
| Post comment, close PR, and fail | If check failed (enforcement on) |

## Check Logic

The core check step:

1. **Maintainer override (label removed):** If a maintainer (org member) removed `missing-issue-link`, bypasses enforcement — removes the label, reopens the PR if closed, and adds `bypass-issue-check`.
2. **Maintainer override (PR reopened with label):** Same bypass if an org member reopens a PR that has `missing-issue-link`.
3. **Live label race guard:** Re-fetches labels to catch concurrent updates; exits early if `trusted-contributor` or `bypass-issue-check` are present.
4. **Issue link check:** Scans the PR body for patterns like `Fixes #NNN`, `Closes #NNN`, `Resolves #NNN` (case-insensitive, all variants).
5. **Assignee check:** For each linked issue (up to 5), verifies the PR author is assigned to at least one.

## Failure Actions

When a PR fails:
- Adds `missing-issue-link` label
- Posts (or updates) a comment explaining the requirement with instructions
- Closes the PR
- Cancels all other in-progress and queued workflow runs for the same commit

The comment includes an HTML marker `<!-- require-issue-link -->` for idempotent management.

## Maintainer Override Options

| Action | Effect |
|---|---|
| Reopen a closed PR with `missing-issue-link` | Applies `bypass-issue-check`, reopens PR |
| Remove `missing-issue-link` label | Same — applies `bypass-issue-check`, reopens if closed |
| Label PR `trusted-contributor` | Skips check on future triggers |
| Label PR `bypass-issue-check` | Skips check on future triggers |

## Notes

- Depends on `pr_labeler.yml` having applied the `external` label first (required for the labeling event chain to work).
- The `bypass-issue-check` label persists — once set, future edits/reopens won't re-trigger enforcement.
