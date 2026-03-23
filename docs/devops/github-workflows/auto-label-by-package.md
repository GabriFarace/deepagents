# Workflow: `auto-label-by-package.yml` — Auto Label Issues by Package

## Overview

Automatically applies package-specific labels to GitHub issues based on the "Area" section in the issue body. Supports both checkbox (multi-select) and dropdown (single-select) formats.

## Trigger

`issues` events: `opened` and `edited`.

## Jobs

### `label-by-package`

Runs on `ubuntu-latest` with `issues: write` permission.

| Step | Description |
|---|---|
| Sync package labels | Inline `actions/github-script@v8` that parses the issue body and applies labels |

## Logic

1. Extracts the text under the `### Area` section of the issue body using a regex.
2. Maps area names to labels:
   - `"deepagents (SDK)"` → `deepagents`
   - `"cli"` → `cli`
3. Handles both checkbox format (`- [x] item`) and plain-text dropdown format.
4. Compares desired labels to current labels and applies additions and removals atomically.

## Managed Labels

| Area Option | Label Applied |
|---|---|
| `deepagents (SDK)` | `deepagents` |
| `cli` | `cli` |

## Notes

- Labels not selected in the issue are removed if previously applied by this workflow, keeping labels accurate as issues are edited.
- This workflow reads from issue templates that present the `### Area` section.
