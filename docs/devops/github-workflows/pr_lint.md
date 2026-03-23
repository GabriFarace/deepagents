# Workflow: `pr_lint.yml` — PR Title Lint

## Overview

Validates that PR titles follow the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/) specification. Enforces a consistent, machine-readable commit history that release-please can parse for changelog generation and version bumping.

## Trigger

`pull_request` events: `opened`, `edited`, `synchronize`.

## Permissions

- `pull-requests: read`

## Allowed Format

```
<type>[optional scope]: <description>
```

Examples:
- `feat(sdk): add multi-agent support`
- `fix(cli): resolve flag parsing error`
- `docs: update API usage examples`

## Jobs

### `lint-pr-title`

Runs on `ubuntu-latest`.

| Step | Description |
|---|---|
| Reject empty scope | Fails if title has empty parentheses (e.g. `fix(): ...`) |
| Validate Conventional Commits format | Uses `amannn/action-semantic-pull-request@v6` |

## Allowed Types

| Type | Description |
|---|---|
| `feat` | New feature (MINOR bump) |
| `fix` | Bug fix (PATCH bump) |
| `docs` | Documentation changes |
| `style` | Formatting/linting (no code change) |
| `refactor` | Code change without feature/fix |
| `perf` | Performance improvement |
| `test` | Adding/correcting tests |
| `build` | Build system/external dependencies |
| `ci` | CI configuration changes |
| `chore` | Other non-source changes |
| `revert` | Reverts a previous commit |
| `release` | Prepare a new release |
| `hotfix` | Urgent fix that won't trigger a release |

## Allowed Scopes (optional)

`acp`, `ci`, `cli`, `cli-gha`, `daytona`, `deepagents`, `deepagents-cli`, `deps`, `evals`, `examples`, `harbor`, `infra`, `quickjs`, `sdk`

Multiple scopes are allowed separated by commas.

## Special Rules

- **Breaking changes**: append `!` after type/scope (e.g. `feat!: drop support for X`)
- **Release commits**: format `release(scope): x.y.z` (e.g. `release(deepagents): 1.2.0`)
- **Disallowed scopes**: `release` as a scope (different from `release` as a type) and uppercase scopes
- **Override**: PRs labeled `ignore-lint-pr-title` skip this check

## Notes

- Scope is optional (`requireScope: false`)
- The `release` type in PR titles is for release-please generated PRs; it is distinct from the disallowed `release` scope
