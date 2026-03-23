# Release Please Configuration

## Overview

`release-please-config.json` configures the [release-please](https://github.com/googleapis/release-please) tool that automates release PR management for the `deepagents-cli` package. Release-please monitors commits on `main`, analyzes Conventional Commits, and creates/updates a draft release PR with an auto-generated changelog and version bump.

## Location

`/release-please-config.json`

## Key Behaviors

- **`skip-github-release: true`** — Release-please does NOT create GitHub releases directly. GitHub releases are created by `release.yml` after PyPI publication succeeds.
- **`draft-pull-request: true`** — Release PRs are created as drafts to prevent accidental merges.
- **`component-no-space: true`** — Component names in tags have no spaces.

## Pull Request Configuration

### Title Pattern

```
release(${component}): ${version}
```

Example: `release(deepagents-cli): 0.5.1`

### Pull Request Header

Release PRs open with a caution banner:
```
> [!CAUTION]
> Merging this PR will automatically publish to **PyPI** and create a **GitHub release**.
```

Followed by a link to `.github/RELEASING.md`.

### Pull Request Footer

Includes a note that a "New Contributors" section is automatically appended to the GitHub release notes at publish time.

## Changelog Sections

### Visible Sections (appear in CHANGELOG.md)

| Commit Type | Section Header |
|---|---|
| `feat` | Features |
| `fix` | Bug Fixes |
| `perf` | Performance Improvements |
| `revert` | Reverted Changes |

### Hidden Sections (tracked but not shown)

`docs`, `style`, `chore`, `refactor`, `test`, `ci`, `hotfix`

## Packages

Currently only one package is managed:

### `libs/cli` — `deepagents-cli`

| Setting | Value |
|---|---|
| `release-type` | `python` |
| `package-name` | `deepagents-cli` |
| `component` | `deepagents-cli` |
| `bump-minor-pre-major` | `true` — minor version bumps before 1.0.0 |
| `bump-patch-for-minor-pre-major` | `true` — feat commits bump patch (not minor) before 1.0.0 |
| `extra-files` | `pyproject.toml`, `deepagents_cli/_version.py` |
| `changelog-path` | `CHANGELOG.md` |

The `extra-files` setting ensures release-please updates version strings in both `pyproject.toml` and `_version.py`.

## Tag Format

Tags follow the pattern: `<package-name>==<version>` (e.g. `deepagents-cli==0.5.1`)

- `tag-separator: "=="` — uses `==` instead of `@` or `-`
- `include-component-in-tag: true` — component name is part of the tag
- `include-v-in-tag: false` — no `v` prefix on versions

## Manifest File

`.release-please-manifest.json` (tracked separately) stores the current version for each component. Release-please updates this file when creating release PRs.

## Notes

- Other packages (`deepagents` SDK, partner packages) are released manually via `workflow_dispatch` in `release.yml` — they are not managed by release-please.
- When a release PR is merged, `release-please.yml` detects the CHANGELOG.md update and automatically triggers `release.yml` for the CLI package.
