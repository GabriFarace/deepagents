# Script: `pr-labeler.js`

## Overview

Shared JavaScript helper module providing all labeling logic for the PR labeler workflows. Used by `pr_labeler.yml`, `pr_labeler_backfill.yml`, and `tag-external-issues.yml` via `actions/github-script`. Reads configuration from `pr-labeler-config.json`.

## Location

`.github/scripts/pr-labeler.js`

## Usage

```javascript
const { config, h } = require('./.github/scripts/pr-labeler.js').loadAndInit(github, owner, repo, core);
```

Requires `actions/checkout` to have run first so the file is available.

## Exports

### `loadConfig() -> object`

Reads and parses `pr-labeler-config.json` from the same directory. Validates that all required keys are present (`labelColor`, `sizeThresholds`, `fileRules`, `typeToLabel`, `scopeToLabel`, `trustedThreshold`, `excludedFiles`, `excludedPaths`). Throws on missing file, JSON parse error, or missing keys.

### `init(github, owner, repo, config, core) -> object`

Initializes all helper functions with the provided GitHub API client, repository context, configuration, and core logger. Returns the `h` helpers object.

### `loadAndInit(github, owner, repo, core) -> {config, h}`

Convenience function that calls `loadConfig()` then `init()`. This is the primary entry point.

## Helper Functions (`h`)

### `ensureLabel(name, color?)`

Creates a label if it doesn't exist (using `labelColor` from config as default color). Handles 422 (concurrent creation) gracefully.

### `getSizeLabel(totalChanged) -> string`

Returns the appropriate size label string for a given number of changed lines, based on `sizeThresholds` in config.

### `computeSize(files) -> {totalChanged, sizeLabel}`

Computes total changed lines from a list of PR file objects, excluding files in `excludedFiles` (by basename) and paths under `excludedPaths`. Returns total count and size label.

### `buildFileRules() -> Rule[]`

Compiles `fileRules` from config into executable matcher objects. Each rule has a `test(path)` function based on `prefix`, `suffix`, `exact`, or `pattern` (regex). Throws for rules with no recognized matcher.

### `matchFileLabels(files, fileRules?) -> Set<string>`

Returns a set of labels matching the changed files. When `skipExcludedFiles` is set on a rule, files in `excludedFiles` are ignored for that rule (prevents lockfile-only changes from triggering package labels).

### `matchTitleLabels(title) -> {labels, type, typeLabel, scopes, breaking}`

Parses a Conventional Commits title. Returns:
- `labels` — set of labels to apply (type label + scope labels + `breaking` if `!` present)
- `type` — extracted commit type
- `typeLabel` — resolved label for the type (from `typeToLabel` config)
- `scopes` — array of scope strings
- `breaking` — boolean

### `checkMembership(author, userType) -> {isExternal}`

Checks whether `author` is an active member of the `langchain-ai` org via the GitHub API. Bots are treated as internal. Throws (does not silently default to external) on non-404 API errors.

### `getContributorInfo(contributorCache, author, userType) -> {isExternal, mergedCount}`

Checks membership with caching. For external contributors, also queries total merged PR count in the repo via GitHub search API. Returns `mergedCount: null` if the search API returns 422.

### `applyTierLabel(issueNumber, author, options?) -> string|undefined`

Determines and applies the contributor tier label to an issue or PR:
- 5+ merged PRs → `trusted-contributor`
- 0 merged PRs → `new-contributor` (unless `skipNewContributor: true`)
- Otherwise → no tier label

Returns the applied label name or `undefined`.

## Exposed Properties

| Property | Type | Description |
|---|---|---|
| `sizeLabels` | `string[]` | All configured size label names |
| `allTypeLabels` | `string[]` | All unique labels derivable from `typeToLabel` |
| `tierLabels` | `string[]` | `['new-contributor', 'trusted-contributor']` |
| `trustedThreshold` | `number` | Merged PR count threshold for trusted-contributor |
| `labelColor` | `string` | Default label hex color |

## Configuration

Configuration lives in `pr-labeler-config.json`. See [PR Labeler Config documentation](./pr-labeler-config.md) for the full schema.

## Dependencies

- Node.js built-ins: `fs`, `path`
- GitHub Actions context: `github`, `core` (injected via `actions/github-script`)
