# Workflow: `ci.yml` — Main CI

## Overview

The primary CI workflow for the deepagents monorepo. On every pull request and push to `main`, it detects which packages have changed and runs linting, unit tests, and benchmarks only for the affected packages. This keeps CI fast without sacrificing coverage.

## Trigger

- `push` to `main` branch
- `pull_request` (all)
- `merge_group` events

Concurrent runs for the same workflow/ref are cancelled.

## Environment Variables

- `UV_NO_SYNC: "true"`

## Permissions

- `contents: read`
- `id-token: write` — required for CodSpeed OIDC authentication in `_benchmark.yml`

## Jobs

### 1. `changes` — Detect Changes

Uses `dorny/paths-filter@v4` (with `fetch-depth: 0` for full history) to determine which packages have changed.

**Outputs:** Boolean flags for each package: `deepagents`, `cli`, `evals`, `daytona`, `modal`, `runloop`, `quickjs`.

**Path filters (each package also matches `.github/workflows/ci.yml`, `_lint.yml`, `_test.yml`, and `.github/actions/**` to ensure CI infra changes run full CI):**

| Package | Paths |
|---|---|
| `deepagents` | `libs/deepagents/**` + CI infra paths |
| `cli` | `libs/cli/**`, `libs/deepagents/**` + CI infra + benchmark paths |
| `evals` | `libs/evals/**` + CI infra paths |
| `daytona` | `libs/partners/daytona/**` + CI infra paths |
| `modal` | `libs/partners/modal/**` + CI infra paths |
| `runloop` | `libs/partners/runloop/**` + CI infra paths |
| `quickjs` | `libs/partners/quickjs/**` + CI infra paths |

### 2. Lint Jobs

Each runs `_lint.yml` for the corresponding package directory when that package has changed (or always on `push` to `main`):

| Job | Directory | Python |
|---|---|---|
| `lint-deepagents` | `libs/deepagents` | 3.11 |
| `lint-cli` | `libs/cli` | 3.11 |
| `lint-evals` | `libs/evals` | 3.14 |
| `lint-daytona` | `libs/partners/daytona` | 3.11 |
| `lint-modal` | `libs/partners/modal` | 3.11 |
| `lint-runloop` | `libs/partners/runloop` | 3.11 |
| `lint-quickjs` | `libs/partners/quickjs` | 3.11 |

### 3. Test Jobs

Each runs `_test.yml` over a Python version matrix when that package has changed. Callers pass a JSON array to `python-versions` and the matrix is expanded inside `_test.yml`; each job renders as a **single parent row** in the Actions UI with child rows per version/OS leg.

| Job | Directory | Python Matrix | Extra Legs | Coverage leg |
|---|---|---|---|---|
| `test-deepagents` | `libs/deepagents` | 3.11, 3.12, 3.13, 3.14 on ubuntu | Windows 3.13 via `extra-configurations` | 3.12 / ubuntu |
| `test-cli` | `libs/cli` | 3.11, 3.12, 3.13, 3.14 | — | 3.12 / ubuntu |
| `test-evals` | `libs/evals` | 3.12, 3.13, 3.14 | — | 3.12 / ubuntu |
| `test-daytona` | `libs/partners/daytona` | 3.11, 3.12, 3.13, 3.14 | — | 3.12 / ubuntu |
| `test-modal` | `libs/partners/modal` | 3.11, 3.12, 3.13, 3.14 | — | 3.12 / ubuntu |
| `test-runloop` | `libs/partners/runloop` | 3.11, 3.12, 3.13, 3.14 | — | 3.12 / ubuntu |

The former standalone `test-deepagents-windows` job has been removed; Windows coverage is now folded into `test-deepagents` via the `extra-configurations` input. All test matrices use `fail-fast: false` so failures in one Python version don't cancel the others.

### 4. Benchmark Jobs

| Job | Status | Directory |
|---|---|---|
| `benchmark-deepagents` | Disabled (`if: false`) — pending CodSpeed integration | `libs/deepagents` |
| `benchmark-cli` | Active | `libs/cli` |

Both delegate to `_benchmark.yml` with `secrets: inherit`.

### 5. `ci_success` — Final Status Check

Aggregates results of all jobs. Fails if any job returned `failure` or `cancelled`. Runs `if: always()` so it always provides a definitive pass/fail signal usable as a branch protection required check.

## Notes

- The path filter approach avoids `!` (negation) patterns which cause all-or-nothing matching issues with `dorny/paths-filter`.
- SDK changes also trigger CLI tests because `cli` filter includes `libs/deepagents/**`.
- The `ci_success` job is the recommended required status check for branch protection rules.
