# Workflow: `close_unchecked_issues.yml` — Close Unchecked Issues

## Overview

Automatically closes issues that bypass or ignore the issue template. GitHub issue forms enforce `required: true` checkboxes in the web UI, but the API bypasses form validation entirely, allowing bots/scripts to open issues with every box unchecked or without a template. This workflow blocks programmatic submission to keep humans in the loop and reduce low-quality/bot-generated noise.

## Trigger

- `issues` events: `opened`

## Permissions

- `contents: read` (global)
- `issues: write` (job-level)

## Secrets Required

| Secret | Purpose |
|---|---|
| `ORG_MEMBERSHIP_APP_ID` | GitHub App ID (shared with `tag-external-issues.yml`) |
| `ORG_MEMBERSHIP_APP_PRIVATE_KEY` | GitHub App private key |

## Closure Rules

The workflow evaluates rules in order, closing the issue on the first match:

| Rule | Condition | Exception |
|---|---|---|
| 0 | No issue `type` field (indicates API/CLI submission — web UI templates set it automatically; external users cannot set it via API) | Skipped if author is an org member |
| 1 | No checkboxes at all in the body | Skipped if author is an org member or bot |
| 2 | Checkboxes present but none checked | — |
| 3 | `Submission checklist` section has unchecked boxes | — |
| 4 | `Area (Required)` section has no selection | — |

When an issue is closed, a comment is posted first explaining the reason and linking to the issue templates, then the issue is closed as `not_planned`.

## Jobs

### `check-boxes`

Runs on `ubuntu-latest` with `issues: write` permission.

| Step | Description |
|---|---|
| Checkout | `actions/checkout@v6` |
| Generate GitHub App token | `actions/create-github-app-token@v3` |
| Validate issue checkboxes | `actions/github-script@v8` — evaluates all 5 rules |

The validation script uses `parseSection(heading)` to extract checkbox counts under specific Markdown H2/H3 headings and `isOrgMember()` (backed by `pr-labeler.js`) for membership checks. Membership results are cached within the run to avoid redundant API calls.

## Relationship to Other Workflows

- `tag-external-issues.yml` — tags issues as `external`/`internal` and applies contributor tier labels; handles *classification*
- `close_unchecked_issues.yml` — handles *quality gating* based on template compliance
- Both share the same GitHub App and `pr-labeler.js` helper for org membership lookups
