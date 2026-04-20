# `agent.py`

## High-Level Purpose

This module handles agent management and creation for the CLI. It is responsible for:

- Loading async subagent definitions from `config.toml`
- Listing available agents from the user's `~/.deepagents/` directory
- Resetting agents (clearing their prompt/state)
- Creating the main LangGraph agent with all middleware applied

The agent creation process wires together models, backends, MCP tools, subagents, skills, memory, HITL, unicode security, and sandbox integrations.

## Module-Level Constants

| Constant | Type | Description |
|---|---|---|
| `DEFAULT_AGENT_NAME` | `str` | `"agent"` — default agent used when no `-a` flag is provided |
| `REQUIRE_COMPACT_TOOL_APPROVAL` | `bool` | `True` — `compact_conversation` requires HITL approval |

## Functions

### `load_async_subagents(config_path: Path | None = None) -> list[AsyncSubAgent]`

Loads async subagent definitions from `config.toml`'s `[async_subagents]` section. Each sub-table defines a remote LangGraph deployment.

**Example config:**
```toml
[async_subagents.researcher]
description = "Research agent"
url = "https://my-deployment.langsmith.dev"
graph_id = "agent"
```

**Parameters:**
- `config_path`: Path to config file. Defaults to `~/.deepagents/config.toml`.

**Returns:** List of `AsyncSubAgent` specs. Returns empty list if section is absent, missing, or invalid.

**Required fields per entry:** `description`, `graph_id`.
**Optional fields:** `url`, `headers`.

### `get_available_agent_names() -> list[str]`

Returns a sorted list of available agent names by scanning `~/.deepagents/` for real subdirectories. Symlinks are excluded so dangling links do not masquerade as agents. Filesystem errors (missing parent, permission denied, broken entries) are logged and surfaced as an empty list rather than raised — callers show an empty modal instead of crashing.

**Returns:** Sorted list of agent name strings. Empty when no agents exist or the directory is unreadable.

### `list_agents(*, output_format: OutputFormat = "text") -> None`

Lists all available agents found in `settings.user_deepagents_dir`. Prints a Rich table or JSON output.

**Parameters:**
- `output_format`: `'text'` for Rich console output, `'json'` for machine-readable JSON.

### `create_agent(...) -> Pregel`

The main agent factory. Creates a fully configured LangGraph agent with all CLI middleware layers applied.

**Key steps:**
1. Resolves the backend (local, sandbox, or remote).
2. Creates the LLM model via `create_model`.
3. Loads MCP tools via `resolve_and_load_mcp_tools`.
4. Loads subagents and async subagents.
5. Loads skills.
6. Builds `CompositeBackend` with local shell, filesystem, and optional sandbox backends.
7. Wraps the agent with middleware: `MemoryMiddleware`, `SkillsMiddleware`, `ConfigurableModelMiddleware`, `LocalContextMiddleware`.
8. Applies unicode security hooks.
9. Compiles and returns the LangGraph `Pregel` graph.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `create_deep_agent` | `deepagents` | Core agent factory |
| `CompositeBackend`, `LocalShellBackend` | `deepagents.backends` | Execution backends |
| `FilesystemBackend` | `deepagents.backends.filesystem` | File system access |
| `MemoryMiddleware`, `SkillsMiddleware` | `deepagents.middleware` | Agent middleware |
| `config`, `console`, `settings` | `deepagents_cli.config` | App configuration |
| `ConfigurableModelMiddleware` | `deepagents_cli.configurable_model` | Model switching middleware |
| `get_default_working_dir` | `deepagents_cli.integrations.sandbox_factory` | Sandbox working dir |
| `LocalContextMiddleware` | `deepagents_cli.local_context` | Local execution context |
| `ProjectContext` | `deepagents_cli.project_utils` | Project context detection |
| `list_subagents` | `deepagents_cli.subagents` | Custom subagent loader |
| `detect_dangerous_unicode`, etc. | `deepagents_cli.unicode_security` | Security checks |
