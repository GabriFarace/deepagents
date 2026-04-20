# GitHub Workflows

All workflows live in `.github/workflows/`. The repository uses a layered approach: reusable base workflows (prefixed with `_`) called by the main CI orchestrator, plus standalone workflows for specialized concerns.

## Workflow Index

### Core CI

| Workflow | Trigger | Purpose |
|---|---|---|
| [`ci.yml`](./ci.md) | push to main, PR, merge_group | Main CI — change detection, lint, test, benchmark for all packages |
| [`_lint.yml`](./_lint.md) | workflow_call | Reusable: runs `make lint` for a package |
| [`_test.yml`](./_test.md) | workflow_call | Reusable: runs `make test` for a package across Python versions |
| [`_benchmark.yml`](./_benchmark.md) | workflow_call | Reusable: runs CodSpeed wall-time benchmarks for a package |

### Release Pipeline

| Workflow | Trigger | Purpose |
|---|---|---|
| [`release-please.yml`](./release-please.md) | push to main | Creates/updates draft release PRs for `deepagents-cli`; triggers release on merge |
| [`release.yml`](./release.md) | workflow_call, workflow_dispatch | Builds, tests, and publishes packages to PyPI; creates GitHub releases |

### Integrity Checks

| Workflow | Trigger | Purpose |
|---|---|---|
| [`check_lockfiles.yml`](./check_lockfiles.md) | push to main, PR, merge_group | Verifies all `uv.lock` files are up-to-date |
| [`check_extras_sync.yml`](./check_extras_sync.md) | PR/push touching `libs/cli/pyproject.toml` | Verifies optional extras match required deps |
| [`check_versions.yml`](./check_versions.md) | PR touching version files | Verifies `pyproject.toml` and `_version.py` versions match |
| [`check_sdk_pin.yml`](./check_sdk_pin.md) | PR touching `pyproject.toml` files | Advisory: warns when CLI SDK pin mismatches SDK version |

### Pull Request Management

| Workflow | Trigger | Purpose |
|---|---|---|
| [`pr_lint.yml`](./pr_lint.md) | PR opened/edited/synchronized | Validates PR title follows Conventional Commits |
| [`pr_labeler.yml`](./pr_labeler.md) | pull_request_target | Applies size, file, title, and contributor labels to PRs |
| [`pr_labeler_backfill.yml`](./pr_labeler_backfill.md) | workflow_dispatch | Retroactively applies labels to all open PRs |
| [`require_issue_link.yml`](./require_issue_link.md) | pull_request_target | Enforces external PRs reference an approved, assigned issue |
| [`sync_priority_labels.yml`](./sync_priority_labels.md) | PR opened/edited, issue labeled, workflow_dispatch | Syncs p0–p3 priority labels from issues to linked PRs |

### Issue Management

| Workflow | Trigger | Purpose |
|---|---|---|
| [`auto-label-by-package.yml`](./auto-label-by-package.md) | issue opened/edited | Applies package labels based on "Area" section in issue body |
| [`tag-external-issues.yml`](./tag-external-issues.md) | issue opened, workflow_dispatch | Tags issues as external/internal; applies contributor tier labels |
| [`close_unchecked_issues.yml`](./close_unchecked_issues.md) | issue opened | Auto-closes issues that bypass the template or omit required checkboxes |

### Evaluation & Benchmarking

| Workflow | Trigger | Purpose |
|---|---|---|
| [`evals.yml`](./evals.md) | workflow_dispatch | Runs evaluation suite across multiple AI models |
| [`harbor.yml`](./harbor.md) | workflow_dispatch | Runs Harbor terminal-bench evaluation across models and sandbox environments |

### Example/Reference

| Workflow | Trigger | Purpose |
|---|---|---|
| [`deepagents-example.yml`](./deepagents-example.md) | PR/issue comment mentioning `@deepagents`, workflow_dispatch | Demonstrates using the Deep Agents action to respond to PR comments with AI |

## Reusable Workflow Pattern

The `_lint.yml`, `_test.yml`, and `_benchmark.yml` workflows are prefixed with `_` to signal they are reusable base workflows. They accept `working-directory` as the primary input. `_test.yml` accepts `python-versions` (JSON array) and `extra-configurations` so the full test matrix — including cross-OS legs — is owned by the reusable workflow rather than each caller, making it trivial to add CI for new packages in `ci.yml` without duplicating logic.

## Common Local Action

All workflows that need Python + uv use `./.github/actions/uv_setup` — a local composite action that installs a pinned version of uv (`0.5.25`) with segmented caching. See [uv_setup documentation](../github-actions-uv-setup.md).

## Concurrency Strategy

Most workflows cancel in-progress runs for the same ref/PR when a new run starts. Exceptions:
- `pr_labeler.yml`: keeps `opened` events, cancels subsequent ones (to preserve external/tier labels)
- `sync_priority_labels.yml`: serializes per-PR or per-issue, doesn't cancel (last-writer-wins convergence)
- `ci_success`: always runs (`if: always()`) to provide a consistent status signal
