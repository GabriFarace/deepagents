# DevOps Documentation

This directory documents the CI/CD infrastructure, automation scripts, and developer tooling for the deepagents monorepo.

## Contents

| Document | Description |
|---|---|
| [GitHub Workflows README](./github-workflows/README.md) | Overview of all GitHub Actions workflows |
| [GitHub Scripts README](./github-scripts/README.md) | Overview of all CI/CD scripts |
| [Makefile](./Makefile.md) | Root Makefile — monorepo-wide lock/lint/format commands |
| [Pre-commit Config](./pre-commit-config.md) | Git pre-commit hooks for local development |
| [action.yml](./action.md) | The `langchain-ai/deepagents` GitHub Action |
| [release-please-config.json](./release-please.md) | Release-please configuration for automated CLI releases |
| [.mcp.json](./mcp-config.md) | Model Context Protocol server configuration |
| [dependabot.yml](./dependabot.md) | Dependabot automated dependency update configuration |
| [CODEOWNERS](./CODEOWNERS.md) | Code ownership and required reviewer rules |
| [uv_setup action](./github-actions-uv-setup.md) | Local composite action for Python + uv setup |

## Repository Structure

```
deepagents/
├── .github/
│   ├── workflows/        # GitHub Actions workflows
│   ├── scripts/          # CI/CD support scripts
│   ├── actions/
│   │   └── uv_setup/     # Local composite action
│   ├── CODEOWNERS
│   └── dependabot.yml
├── libs/
│   ├── deepagents/       # Core SDK
│   ├── cli/              # deepagents-cli
│   ├── evals/            # Evaluation suite
│   ├── acp/              # ACP package
│   └── partners/         # Partner integrations (daytona, modal, quickjs, runloop)
├── action.yml            # GitHub Action definition
├── Makefile              # Root monorepo Makefile
├── .pre-commit-config.yaml
├── release-please-config.json
└── .mcp.json
```

## CI/CD Architecture

### The Main CI Pipeline

Every pull request and push to `main` runs through `ci.yml`:

1. **Change Detection** (`dorny/paths-filter`) — determines which packages changed
2. **Lint** (via `_lint.yml`) — runs `make lint` per changed package in parallel
3. **Test** (via `_test.yml`) — runs `make test` per changed package across Python 3.11–3.14 in parallel
4. **Benchmark** (via `_benchmark.yml`) — runs CodSpeed benchmarks for CLI changes
5. **ci_success** — aggregates all results into a single required status check

SDK changes automatically trigger CLI tests because the CLI depends on the SDK.

### Release Pipeline

The release process follows this flow:

```
Commits on main
      ↓
release-please.yml (analyzes conventional commits)
      ↓
Draft release PR created/updated (with CHANGELOG + version bump)
      ↓ (PR merged)
release-please.yml detects CHANGELOG.md change
      ↓
release.yml triggered with package=deepagents-cli
      ↓
setup → build → pre-release-checks + test-pypi-publish (parallel)
      ↓
publish (to PyPI)
      ↓
mark-release (creates GitHub release, updates PR labels)
```

For other packages (`deepagents` SDK, partner packages), releases are triggered manually via `workflow_dispatch` on `release.yml`.

### Pre-Commit Hooks

The `.pre-commit-config.yaml` runs locally before every commit:
- Prevents direct commits to `main`
- Validates YAML, TOML syntax
- Fixes file hygiene (trailing whitespace, EOF newlines, smart quotes)
- Runs `make format lint` per package (for changed packages)
- Checks lockfiles are up-to-date
- Checks extras sync and version equality

### PR Quality Gates

Every pull request goes through:

| Check | Workflow | Hard/Advisory |
|---|---|---|
| Conventional Commits title | `pr_lint.yml` | Hard (blocks merge) |
| Lockfiles up-to-date | `check_lockfiles.yml` | Hard |
| Version sync (pyproject ↔ _version.py) | `check_versions.yml` | Hard |
| Extras sync | `check_extras_sync.yml` | Hard |
| SDK pin for CLI releases | `check_sdk_pin.yml` | Advisory (PR comment) |
| External PR must reference issue | `require_issue_link.yml` | Hard (closes PR) |

### PR Automation

PRs are automatically labeled with:
- **Size labels** (`size: XS` through `size: XL`) based on changed lines
- **Package labels** (`deepagents`, `cli`, `evals`, etc.) based on changed files
- **Type labels** (`feature`, `fix`, `tests`, etc.) based on PR title
- **Contributor labels** (`internal`, `external`, `new-contributor`, `trusted-contributor`)
- **Priority labels** (`p0`–`p3`) inherited from linked issues

### Evaluation Infrastructure

Two manual workflows support model evaluation:

- **`evals.yml`** — runs the unit-test-style evaluation suite across configurable model presets. Results are aggregated into Markdown tables and radar charts.
- **`harbor.yml`** — runs the Harbor terminal-bench evaluation using various sandbox environments (Docker, Daytona, Modal, Runloop, LangSmith).

Both use `models.py` as a single source of truth for model definitions and preset groups.

## Key Design Decisions

### uv for Package Management

All Python environments use [uv](https://github.com/astral-sh/uv) for fast, reproducible dependency management. The `uv_setup` local action pins uv at `0.5.25` across all CI jobs.

### Monorepo with Per-Package Makefiles

Each package has its own `Makefile` defining `lint`, `format`, `test`, and `evals` targets. The root `Makefile` delegates to package Makefiles, enabling both per-package and monorepo-wide operations.

### OIDC Trusted Publishing

Package releases to PyPI use [OIDC trusted publishing](https://blog.pypi.org/posts/2023-04-20-introducing-trusted-publishers/) via `pypa/gh-action-pypi-publish`. No PyPI API tokens are stored as secrets — authentication happens via GitHub's OIDC provider.

### Build/Publish Separation

The `release.yml` workflow deliberately separates the `build` job (minimal permissions: `contents: read`) from the `publish` job (has `id-token: write` for OIDC). This prevents a compromised build dependency from accessing PyPI credentials.

### GitHub App for Org Membership

PR/issue contributor classification requires checking private organization membership. This uses a GitHub App (`ORG_MEMBERSHIP_APP_ID` + `ORG_MEMBERSHIP_APP_PRIVATE_KEY` secrets) rather than a PAT, to:
- Enable label events to propagate to downstream workflows (GITHUB_TOKEN events don't trigger further workflows)
- Access private org membership data

## Secrets Reference

| Secret | Used By | Purpose |
|---|---|---|
| `ORG_MEMBERSHIP_APP_ID` | `pr_labeler.yml`, `tag-external-issues.yml` | GitHub App ID for org membership |
| `ORG_MEMBERSHIP_APP_PRIVATE_KEY` | Same | GitHub App private key |
| `ANTHROPIC_API_KEY` | `evals.yml`, `harbor.yml`, `deepagents-example.yml` | Anthropic model API key |
| `OPENAI_API_KEY` | `evals.yml`, `harbor.yml` | OpenAI model API key |
| `GOOGLE_API_KEY` | `evals.yml` | Google model API key |
| `LANGSMITH_API_KEY` | `evals.yml`, `harbor.yml` | LangSmith experiment tracking |
| `DAYTONA_API_KEY` | `harbor.yml` | Daytona sandbox credentials |
| `MODAL_TOKEN_ID` + `MODAL_TOKEN_SECRET` | `harbor.yml` | Modal sandbox credentials |
| `RUNLOOP_API_KEY` | `harbor.yml` | Runloop sandbox credentials |
| `BASETEN_API_KEY` | `evals.yml`, `harbor.yml` | Baseten model hosting |
| `FIREWORKS_API_KEY` | `evals.yml` | Fireworks model hosting |
| `OPENROUTER_API_KEY` | `evals.yml` | OpenRouter model routing |
| `NVIDIA_API_KEY` | `evals.yml` | NVIDIA NIM models |
| `GROQ_API_KEY` | `evals.yml` | Groq-hosted models |
| `XAI_API_KEY` | `evals.yml` | xAI/Grok models |
| `MISTRAL_API_KEY` | `evals.yml` | Mistral models |
| `DEEPSEEK_API_KEY` | `evals.yml` | DeepSeek models |
| `OLLAMA_API_KEY` | `evals.yml` | Ollama Cloud models |
