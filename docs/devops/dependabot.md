# Dependabot Configuration

## Overview

`.github/dependabot.yml` configures GitHub Dependabot to automatically monitor and open pull requests for dependency updates. Two package ecosystems are monitored: GitHub Actions and Python (via uv).

## Location

`/.github/dependabot.yml`

## Schedule

Both ecosystems are checked **weekly on Mondays**.

## Ecosystems

### GitHub Actions

- **Ecosystem:** `github-actions`
- **Directory:** `/` (root — scans all workflow files)
- **Grouping:** All GitHub Actions updates are grouped into a single PR per cycle (`patterns: ["*"]`)

This updates action version pins in `.github/workflows/*.yml` and `.github/actions/**`.

### Python (uv)

- **Ecosystem:** `uv`
- **Directories monitored:**
  - `/libs/deepagents`
  - `/libs/cli`
  - `/libs/evals`
  - `/libs/acp`
  - `/libs/partners/daytona`
  - `/examples/content-builder-agent`
  - `/examples/deep_research`
  - `/examples/text-to-sql-agent`
- **Grouping:** All pip/uv dependency updates are grouped into a single PR per directory per cycle (`patterns: ["*"]`)

Each monitored directory receives its own grouped PR for Python dependency updates.

## Notes

- Partner packages `langchain-modal`, `langchain-quickjs`, and `langchain-runloop` are not listed — they may need to be added manually if dependency automation is desired.
- Grouped updates reduce PR noise by batching all updates from a cycle into one PR per ecosystem/directory combination.
- The `uv` ecosystem support requires Dependabot to understand `pyproject.toml` and `uv.lock` files.
