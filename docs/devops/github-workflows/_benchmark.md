# Workflow: `_benchmark.yml` — CodSpeed Benchmark (Reusable)

## Overview

A reusable workflow that runs `pytest-benchmark` tests under [CodSpeed](https://codspeed.io/) wall-time instrumentation. Results are tracked across commits on the CodSpeed dashboard to catch performance regressions.

## Trigger

`workflow_call` only — called by other workflows (not triggered directly). Callers pass:

| Input | Required | Default | Description |
|---|---|---|---|
| `working-directory` | Yes | — | Package directory, e.g. `libs/deepagents` |
| `python-version` | No | `3.13.11` | Python version (pinned to avoid CodSpeed segfaults on 3.13.12+) |

## Jobs

### `benchmark` — CodSpeed

Runs on `codspeed-macro` runners (required by CodSpeed).

| Step | Action |
|---|---|
| Checkout | `actions/checkout@v6` |
| Set up Python + uv | `./.github/actions/uv_setup` with benchmark cache suffix |
| Install dependencies | `uv sync --group test` |
| Run benchmarks | `CodSpeedHQ/action@v4` in `walltime` mode, running `pytest ./tests -m benchmark --codspeed` |

## Permissions

- `contents: read`
- `id-token: write` — used for OIDC authentication with CodSpeed (no repository secret needed)

## Environment Variables

- `UV_NO_SYNC: "true"` — prevents uv from auto-syncing during the run

## Notes

- Python 3.13.11 is pinned due to a known CodSpeed segfault on 3.13.12+ ([upstream issue](https://github.com/CodSpeedHQ/pytest-codspeed/issues/106))
- The `benchmark-deepagents` call in `ci.yml` is currently disabled (`if: false`) pending CodSpeed integration readiness
- The `benchmark-cli` call in `ci.yml` is active and triggered on CLI changes
