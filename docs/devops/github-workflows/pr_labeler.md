# Workflow: `pr_labeler.yml` — PR Labeler

## Overview

A unified PR labeling workflow that applies multiple categories of labels in a single sequential pass to avoid race conditions. Consolidates size labels, file-based labels, title-based labels, and contributor classification into one workflow. Shared logic lives in `.github/scripts/pr-labeler.js` and `.github/scripts/pr-labeler-config.json`.

## Trigger

`pull_request_target` events: `opened`, `synchronize`, `reopened`, `edited`.

Uses `pull_request_target` (not `pull_request`) to safely access secrets for organization membership checks — it runs against the base branch, never the PR's code.

## Concurrency

- `opened` events get their own concurrency group so external/tier labels are never cancelled.
- Other events share a group per PR number and cancel in-progress runs.

## Secrets Required

| Secret | Purpose |
|---|---|
| `ORG_MEMBERSHIP_APP_ID` | GitHub App ID for org membership checks |
| `ORG_MEMBERSHIP_APP_PRIVATE_KEY` | GitHub App private key |

A GitHub App with `pull-requests: write`, `issues: write`, and `organization members: read` permissions is required.

## Jobs

### `label`

Runs on `ubuntu-latest` with `pull-requests: write` and `issues: write` permissions.

| Step | Runs When | Description |
|---|---|---|
| Checkout | Always | Checks out base branch so `require('./.github/scripts/pr-labeler.js')` resolves |
| Generate GitHub App token | `opened` only | `actions/create-github-app-token@v3` — needed for private org membership check |
| Verify App token | `opened` only | Fails fast if token generation failed |
| Check org membership | `opened` only | Calls `h.checkMembership()` via `actions/github-script@v8` — outputs `is-external` |
| Rename deepagents scope to sdk | Always | Rewrites `(deepagents)` to `(sdk)` in non-release PR titles |
| Apply PR labels | Always | Core labeling step — applies size, file, title, and internal labels |
| Apply contributor tier label | `opened` + external | Applies `new-contributor` or `trusted-contributor` based on merged PR count |
| Add external label | `opened` + external | Adds `external` label using App token (propagates events to downstream workflows) |

## Labels Applied

### Size Labels (based on added + deleted lines, excluding `uv.lock` and `docs/`)

| Lines Changed | Label |
|---|---|
| < 50 | `size: XS` |
| < 200 | `size: S` |
| < 500 | `size: M` |
| < 1000 | `size: L` |
| 1000+ | `size: XL` |

### File-Based Labels

| Files Changed | Label |
|---|---|
| `libs/deepagents/**` | `deepagents` |
| `libs/cli/**` | `cli` |
| `libs/acp/**` | `acp` |
| `libs/evals/**` | `evals` |
| `action.yml` | `cli`, `github_actions` |
| `.github/workflows/**`, `.github/actions/**` | `github_actions` |
| `**/pyproject.toml`, `uv.lock`, `**/requirements*.txt` | `dependencies` |

### Title-Based Labels (from Conventional Commit type)

| Type | Label |
|---|---|
| `feat` | `feature` |
| `fix` | `fix` |
| `docs` | `documentation` |
| `style` | `linting` |
| `refactor` | `refactor` |
| `perf` | `performance` |
| `test` | `tests` |
| `build`, `ci`, `chore` | `infra` |
| `revert` | `revert` |
| `release` | `release` |
| `hotfix` | `hotfix` |

### Scope-Based Labels

| Scope | Label |
|---|---|
| `sdk`, `deepagents` | `deepagents` |
| `cli`, `deepagents-cli`, `cli-gha` | `cli` |
| `acp` | `acp` |
| `deps` | `dependencies` |
| `evals`, `harbor` | `evals` |
| `ci`, `infra`, `build`, `chore` | `infra` |
| `daytona` | `daytona` |
| `examples` | `examples` |

### Contributor Labels

| Condition | Label |
|---|---|
| Internal contributor (org member) | `internal` |
| External, 0 merged PRs | `new-contributor` |
| External, 5+ merged PRs | `trusted-contributor` |
| Any external | `external` |
