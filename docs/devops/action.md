# GitHub Action: `action.yml` — Deep Agents

## Overview

The root `action.yml` defines the `langchain-ai/deepagents` composite GitHub Action. It provides a ready-to-use action for running the deepagents CLI coding assistant in any GitHub workflow. It handles CLI installation, optional persistent memory via `actions/cache`, optional skills installation, agent execution, and memory persistence.

## Location

`/action.yml`

## Branding

- Icon: `cpu`
- Color: `blue`
- Author: LangChain AI

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `prompt` | Yes | — | The prompt/instruction to send to the agent |
| `model` | No | — | Model to use (`claude-*`, `gpt-*`, `gemini-*`). Provider auto-detected. |
| `anthropic_api_key` | No | — | Anthropic API key |
| `openai_api_key` | No | — | OpenAI API key |
| `google_api_key` | No | — | Google API key |
| `github_token` | No | `${{ github.token }}` | GitHub token for API access |
| `working_directory` | No | `.` | Working directory for the agent |
| `cli_version` | No | (latest) | Pin a specific `deepagents-cli` version (minimum: 0.0.31) |
| `skills_repo` | No | (none) | GitHub repo to clone for skills (`owner/repo`, `owner/repo@ref`, or full URL) |
| `enable_memory` | No | `true` | Persist agent memory across runs using `actions/cache` |
| `memory_scope` | No | `repo` | Cache scope: `pr`, `branch`, or `repo` |
| `agent_name` | No | `agent` | Agent identity name — controls memory namespace |
| `shell_allow_list` | No | `recommended,git,gh` | Shell commands the agent is allowed to execute |
| `timeout` | No | `30` | Maximum agent runtime in minutes |

## Outputs

| Output | Description |
|---|---|
| `response` | Full text response from the agent (stdout + stderr) |
| `exit_code` | Exit code from the agent process |
| `cache_hit` | Whether agent memory was restored from cache (empty if memory disabled) |

## Steps

### 1. Set up uv

Uses `astral-sh/setup-uv@0ca8f610542aa7f4acaf39e65cf4eb3c35091883` (pinned SHA for security) with caching enabled.

### 2. Resolve cache key

If `enable_memory` is `true`, computes the cache key based on `memory_scope`:

| Scope | Cache Key Pattern |
|---|---|
| `pr` | `deepagents-memory-<agent_name>-pr-<PR_NUMBER>` (falls back to `ref-<branch>`) |
| `branch` | `deepagents-memory-<agent_name>-ref-<branch>` |
| `repo` | `deepagents-memory-<agent_name>-repo` |

Unknown scopes default to `pr`-scoped with a warning.

### 3. Restore agent memory

If `enable_memory` is `true`, restores from cache:
- `~/.deepagents/<agent_name>/`
- `~/.deepagents/sessions.db`
- `<working_directory>/.deepagents/AGENTS.md`

### 4. Install deepagents-cli

Installs via `uvx`:
- Pinned version: `uvx --from "deepagents-cli==<version>" deepagents --version`
- Latest: `uvx --from deepagents-cli deepagents --version`

Enforces minimum version `0.0.31` (required for `--agent`, `--shell-allow-list`, `-n` flags).

### 5. Install skills

If `skills_repo` is set, clones the repository (with optional `@ref` spec) and copies all directories containing `SKILL.md` into `.deepagents/skills/<skill_name>/`. Fails if no skills are found.

### 6. Run Deep Agents

Constructs and runs the agent command:
```
uvx --from deepagents-cli deepagents \
  --agent <agent_name> \
  --shell-allow-list <shell_allow_list> \
  [--model <model>] \
  -n <prompt>
```

Runs with a `timeout` (converted to seconds). Captures both stdout and stderr to an output file. Sets `exit_code` and `response` outputs. Exits with the agent's exit code.

### 7. Save agent memory

If `enable_memory` is `true`, saves the memory cache using the same paths as the restore step. Runs with `always()` to persist memory even if the agent fails.

## Usage Example

```yaml
- uses: langchain-ai/deepagents@main
  with:
    prompt: "Fix the failing tests in this PR"
    model: claude-sonnet-4-6
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    skills_repo: langchain-ai/langchain-skills
```

## Notes

- The action uses `uvx` to run the CLI without installing it permanently, always fetching the latest (or pinned) version.
- Memory caching allows the agent to recall prior context from previous runs, enabling iterative workflows.
- The `skills_repo` feature allows packaging reusable agent capabilities as skills and sharing them across repos.
