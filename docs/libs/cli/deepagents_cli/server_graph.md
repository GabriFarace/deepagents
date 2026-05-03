# `libs/cli/deepagents_cli/server_graph.py`

## High-Level Purpose

`server_graph.py` is the module that runs **inside the LangGraph server subprocess**. It contains `make_graph()`, the factory function that LangGraph calls at server startup to obtain the compiled agent graph. This file is copied verbatim into the temporary workspace directory (see [server_manager.md](server_manager.md)), so it must be self-contained: all imports are relative to the CLI package, not to the workspace.

---

## Key Function

### `make_graph() → CompiledStateGraph`

Called once by `langgraph dev` at module import time. Returns the compiled LangGraph state graph that serves all incoming requests.

**Steps:**

1. **Read server config** — Calls `ServerConfig.from_env()` which reads `DEEPAGENTS_CLI_SERVER_*` environment variables set by `server_manager.py`:
   - `model` — model spec string
   - `mcp_config_path` — path to MCP config (if any)
   - `agent_name` — selected agent (if any)
   - `auto_approve` — whether HITL is disabled
   - `interrupt_shell_only` — whether only shell commands require approval

2. **Load MCP tools** — calls `resolve_and_load_mcp_tools(config.mcp_config_path)` to discover and load any configured MCP server tools.

3. **Load subagent specs** — calls `load_subagents(agent_name)` to read subagent YAML files from `~/.deepagents/{agent_name}/agents/`.

4. **Create the agent** — calls `create_cli_agent(...)` (from `agent.py`) with the resolved config, MCP tools, and subagent specs.

5. **Return the compiled graph** — the graph is compiled with an `AsyncSqliteSaver` checkpointer that reads the DB path from `DEEPAGENTS_CLI_DB_PATH`.

---

## `ServerConfig`

A simple dataclass read from environment variables. All fields have defaults.

| Field | Env var | Default |
|---|---|---|
| `model` | `DEEPAGENTS_CLI_SERVER_MODEL` | `"claude-sonnet-4-6"` |
| `mcp_config_path` | `DEEPAGENTS_CLI_SERVER_MCP_CONFIG` | `None` |
| `agent_name` | `DEEPAGENTS_CLI_SERVER_AGENT` | `None` |
| `auto_approve` | `DEEPAGENTS_CLI_SERVER_AUTO_APPROVE` | `False` |
| `interrupt_shell_only` | `DEEPAGENTS_CLI_SERVER_INTERRUPT_SHELL_ONLY` | `False` |

---

## Architecture Notes

**Separation of concerns:** This file is deliberately thin — it reads config, calls the three loader functions, and returns the result. All logic lives in `agent.py`, `mcp_tools.py`, and `subagents.py`. This keeps the "what to wire up" logic in `server_graph.py` and the "how to wire it" logic in the respective modules.

**Module boundary:** `server_graph.py` is copied into a generated workspace, so it runs in a context where `deepagents_cli` is installed as a package. Any refactoring that changes the module's public interface will break the copy.

---

## See Also

- [agent.md](agent.md) — `create_cli_agent()` called from here
- [server_manager.md](server_manager.md) — copies this file and sets env vars
- [mcp_tools.md](mcp_tools.md) — `resolve_and_load_mcp_tools()`
- [subagents.md](subagents.md) — `load_subagents()`
