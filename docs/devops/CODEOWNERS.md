# CODEOWNERS

## Overview

`.github/CODEOWNERS` defines code ownership rules for the repository. GitHub uses this file to automatically request reviews from the designated owners when pull requests modify files in their ownership areas.

## Location

`/.github/CODEOWNERS`

## Ownership Rules

| Path Pattern | Owner(s) | Description |
|---|---|---|
| `/libs/cli/` | `@mdrxy` | All files in the CLI package |

## How It Works

- When a PR modifies any file under `/libs/cli/`, GitHub automatically adds `@mdrxy` as a required reviewer.
- The owner must approve the PR before it can be merged (if branch protection rules require CODEOWNERS approval).
- CODEOWNERS rules are evaluated from bottom to top — the last matching rule takes precedence. Currently there is only one rule.

## Notes

- The `libs/deepagents/` (SDK) directory does not have a designated CODEOWNER, meaning any contributor with write access can merge SDK changes without a specific required reviewer.
- For more information on CODEOWNERS syntax: [GitHub CODEOWNERS documentation](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
