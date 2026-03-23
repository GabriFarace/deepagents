# Workflow: `release-please.yml` — Release Please (CLI Only)

## Overview

Automates release PR management for the `deepagents-cli` package using [release-please](https://github.com/googleapis/release-please). When commits land on `main`, release-please analyzes conventional commits and either creates/updates a draft release PR (with changelog and version bump) or, when a release PR is merged, triggers the full release workflow.

## Trigger

`push` to `main` branch only.

## Permissions

- `contents: write`
- `pull-requests: write`

## Jobs

### 1. `release-please`

Runs `googleapis/release-please-action@v4` with:
- `config-file: release-please-config.json`
- `manifest-file: .release-please-manifest.json`

After the action runs, checks whether the CLI's `CHANGELOG.md` was modified in the most recent commit (indicating a release PR was just merged).

**Outputs:**
- `cli-release` — `true` if this was a CLI release commit
- `pr` — the release PR object if a PR was created/updated

### 2. `update-lockfiles`

Runs when release-please creates or updates a release PR (`pr != ''`).

Checks out the release branch and regenerates all `uv.lock` files using the appropriate Python version per package (3.14 for `libs/acp`, 3.12 for everything else). Commits and pushes the result as `chore: update lockfiles`.

This is necessary because release-please updates `pyproject.toml` version numbers but does not regenerate lockfiles ([upstream issue](https://github.com/googleapis/release-please/issues/2561)).

### 3. `release-deepagents-cli`

Runs when `cli-release == 'true'` (a CLI release PR was merged). Calls `release.yml` with `package: deepagents-cli`.

Permissions passed: `contents: write`, `id-token: write`, `pull-requests: write`.

## Release PR Format

PR titles follow the pattern `release(deepagents-cli): X.Y.Z` (configured in `release-please-config.json`).

Release PRs include a caution banner: merging will publish to PyPI and create a GitHub release.

## Notes

- GitHub releases are created by `release.yml`, not by release-please directly (`skip-github-release: true` in config).
- Release PRs are created as **drafts** (`draft-pull-request: true`).
- The manifest file `.release-please-manifest.json` tracks the current version for each component.
- Currently only the `deepagents-cli` package is managed by release-please; other packages use manual `workflow_dispatch` in `release.yml`.
