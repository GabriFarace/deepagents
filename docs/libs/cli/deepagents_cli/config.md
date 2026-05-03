# `libs/cli/deepagents_cli/config.py`

## High-Level Purpose

`config.py` manages all runtime configuration for the CLI. It provides the `settings` singleton — a lazily-initialized object that aggregates CLI flags, environment variables, dotenv files, and the TOML config file into a single coherent view. Other modules import `settings` and read properties from it; they never parse env vars or config files themselves.

---

## Key Objects

### `settings` (module-level singleton)

A `Settings` instance. Initialized lazily on first property access via `_ensure_bootstrap()`.

**Key properties:**

| Property | Source | Default | Description |
|---|---|---|---|
| `model_name` | `--model` flag or `config.toml [models].default` | `"claude-sonnet-4-6"` | Active model spec |
| `has_tavily` | `TAVILY_API_KEY` env var | `False` | Whether web search is available |
| `shell_allow_list` | `--shell-allow-list` flag | `[]` | Allowed shell commands (non-interactive) |
| `auto_approve` | `--auto-approve` flag | `False` | Disable HITL approvals |
| `theme` | `config.toml [ui].theme` | `"default"` | TUI color theme |
| `langsmith_project` | `LANGSMITH_PROJECT` env var | `None` | LangSmith project for traces |
| `mcp_config_path` | `--mcp-config` flag | `None` | Path to MCP config file |
| `agent_name` | `--agent` flag | `None` | Selected agent name |

### `_ensure_bootstrap() → None`

Called once on first `settings` access. Loads configuration in this order:

1. Project `.env` (in `$DEEPAGENTS_PROJECT_ROOT/.env` or current directory)
2. Global `~/.deepagents/.env`
3. CLI flags already parsed into `args` namespace
4. Reads `~/.deepagents/config.toml` for TOML settings

Captures the original `LANGSMITH_PROJECT` before overwriting it with the CLI's project name, so traces are routed correctly.

---

## Config File: `~/.deepagents/config.toml`

```toml
[models]
default = "anthropic:claude-sonnet-4-6"

[agents]
recent = "my-agent"

[ui]
theme = "monokai"

[warnings]
suppress = ["update_available"]

[async_subagents]
my-researcher = {graph_id = "researcher", url = "https://deploy.langchain.com/..."}
```

**Sections:**

| Section | Purpose |
|---|---|
| `[models]` | Default model and any provider overrides |
| `[agents]` | Which agent was most recently selected |
| `[ui]` | TUI theme preference |
| `[warnings]` | Warnings to suppress (e.g., update notifications) |
| `[async_subagents]` | Remote LangGraph deployments for async delegation |

---

## `ProjectContext`

A dataclass capturing the project-level context at CLI startup:

| Field | Description |
|---|---|
| `cwd` | Current working directory |
| `project_root` | Nearest ancestor dir containing `.git`, `pyproject.toml`, etc. |
| `git_branch` | Active git branch, or `None` |

Populated by `ProjectContext.from_user_cwd()`. Passed to the server subprocess via env vars so the agent's `LocalContextMiddleware` can inject accurate git/directory info.

---

## Architecture Notes

**Lazy initialization:** `settings` is not initialized at import time. This keeps `from deepagents_cli.config import settings` fast (used in many places). The actual bootstrap (dotenv loading, TOML parsing) runs only when a property is first accessed.

**Thread safety:** Bootstrap is guarded by a `threading.Lock`. Multiple threads accessing `settings` simultaneously won't duplicate initialization.

**Dotenv precedence:** The project `.env` is loaded first. The global `~/.deepagents/.env` is loaded second. Later loads don't overwrite already-set values (using `dotenv`'s `override=False` mode). This means project-level env vars take precedence over user-level ones.

---

## See Also

- [main.md](main.md) — reads `settings` for model and mode configuration
- [agent.md](agent.md) — reads `settings` for tool configuration
- [mcp_tools.md](mcp_tools.md) — reads `settings` for MCP config path
