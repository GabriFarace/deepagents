# Local Action: `uv_setup` — Set Up Python and uv

## Overview

A composite GitHub Action (`/.github/actions/uv_setup/action.yml`) that installs a pinned version of `uv` and the specified Python version with dependency caching. Used as a setup step in nearly all CI/CD workflows across the repository.

## Location

`.github/actions/uv_setup/action.yml`

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `python-version` | Yes | — | Python version (MAJOR.MINOR format, e.g. `3.12`) |
| `enable-cache` | No | `"true"` | Whether to enable uv dependency caching |
| `cache-suffix` | No | `""` | Custom suffix for cache key invalidation (e.g. `lint-libs/deepagents`) |
| `working-directory` | No | `"**"` | Directory scope for cache dependency glob |

## Pinned uv Version

`UV_VERSION: "0.5.25"` — pinned to ensure reproducible builds across all CI jobs.

## Steps

### Install uv and set the Python version

Uses `astral-sh/setup-uv@v7` with:
- The pinned `UV_VERSION`
- The specified Python version
- Cache enabled/disabled per input
- Cache dependency glob scoped to:
  - `<working-directory>/pyproject.toml`
  - `<working-directory>/uv.lock`
  - `<working-directory>/requirements*.txt`
- Cache suffix for segmented caching per job type

## Cache Key Strategy

The cache suffix allows different job types to maintain independent caches, preventing cache poisoning between lint, test, and benchmark jobs. Common suffixes used across workflows:

| Caller | Suffix |
|---|---|
| `_lint.yml` | `lint-<working-directory>` |
| `_test.yml` | `test-<working-directory>` |
| `_benchmark.yml` | `benchmark-<python-version>` |
| `evals.yml` | `evals`, `evals-aggregate` |
| `harbor.yml` | `harbor`, `harbor-prep` |
| `release.yml` | (none — no cache) |

## Usage Example

```yaml
- name: Set up Python + uv
  uses: ./.github/actions/uv_setup
  with:
    python-version: "3.12"
    working-directory: "libs/deepagents"
    cache-suffix: test-libs/deepagents
```

## Notes

- The `working-directory` default of `"**"` makes the cache glob match all directories when no specific directory is provided (useful for root-level operations).
- Caching is disabled for the `pre-release-checks` job in `release.yml` intentionally, to maximize sensitivity to missing dependencies.
- The `enable-cache: "false"` option is used by `check_extras_sync.yml` and `check_versions.yml` for their 2-minute timeout jobs.
