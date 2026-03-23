# Config: `pr-labeler-config.json`

## Overview

JSON configuration file that drives all PR and issue labeling logic for the repository. Shared between `pr-labeler.js`, `pr_labeler.yml`, `pr_labeler_backfill.yml`, and `tag-external-issues.yml`.

## Location

`.github/scripts/pr-labeler-config.json`

## Schema

### Top-Level Fields

| Field | Type | Description |
|---|---|---|
| `org` | `string` | GitHub organization name (`"langchain-ai"`) |
| `trustedThreshold` | `number` | Merged PR count at which an external contributor becomes `trusted-contributor` (currently `5`) |
| `labelColor` | `string` | Default hex color for auto-created labels (`"b76e79"`) |
| `sizeThresholds` | `array` | Size label bands (see below) |
| `excludedFiles` | `array` | Filenames excluded from size calculation and some file rules (e.g. `"uv.lock"`) |
| `excludedPaths` | `array` | Path prefixes excluded from size calculation (e.g. `"docs/"`) |
| `typeToLabel` | `object` | Maps Conventional Commits types to label names |
| `scopeToLabel` | `object` | Maps Conventional Commits scopes to label names |
| `fileRules` | `array` | Rules mapping file paths to labels (see below) |

### `sizeThresholds`

Array of objects with `label` (required) and `max` (optional). The last entry has no `max` and is the catch-all.

| Label | Max Changed Lines |
|---|---|
| `size: XS` | < 50 |
| `size: S` | < 200 |
| `size: M` | < 500 |
| `size: L` | < 1000 |
| `size: XL` | (catch-all) |

### `typeToLabel`

| Type | Label |
|---|---|
| `feat` | `feature` |
| `fix` | `fix` |
| `docs` | `documentation` |
| `hotfix` | `hotfix` |
| `style` | `linting` |
| `refactor` | `refactor` |
| `perf` | `performance` |
| `test` | `tests` |
| `build`, `ci`, `chore` | `infra` |
| `revert` | `revert` |
| `release` | `release` |
| `breaking` | `breaking` |

### `scopeToLabel`

| Scope | Label |
|---|---|
| `acp` | `acp` |
| `ci`, `infra` | `infra` |
| `cli`, `cli-gha`, `deepagents-cli` | `cli` |
| `daytona` | `daytona` |
| `deepagents`, `sdk` | `deepagents` |
| `deps` | `dependencies` |
| `docs` | `documentation` |
| `evals`, `harbor` | `evals` |
| `examples` | `examples` |

### `fileRules`

Array of rule objects, each with a `label` and exactly one of: `prefix`, `suffix`, `exact`, or `pattern` (regex string). Optional `skipExcludedFiles: true` prevents excluded files from triggering the label.

| Matcher | Label | Notes |
|---|---|---|
| prefix `libs/deepagents/` | `deepagents` | Skips excluded files |
| prefix `libs/cli/` | `cli` | Skips excluded files |
| prefix `libs/acp/` | `acp` | Skips excluded files |
| prefix `libs/evals/` | `evals` | Skips excluded files |
| exact `action.yml` | `cli`, `github_actions` | Two rules |
| prefix `.github/workflows/` | `github_actions` | |
| prefix `.github/actions/` | `github_actions` | |
| suffix `pyproject.toml` | `dependencies` | |
| exact `uv.lock` | `dependencies` | |
| pattern `(?:^|/)requirements[^/]*\.txt$` | `dependencies` | |
