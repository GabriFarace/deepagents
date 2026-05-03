# `libs/cli/deepagents_cli/agent.py`

## High-Level Purpose

`agent.py` contains `create_cli_agent()`, the function that assembles the agent graph for the CLI's LangGraph server. It is the CLI-specific analogue of `create_deep_agent()` in the SDK — it calls `create_deep_agent()` but wraps it with CLI-specific middleware, tools, interrupt configuration, and local-shell backend setup. This is where the agent's capabilities and safety guardrails are decided.

---

## Key Function

### `create_cli_agent(config, mcp_tools, subagents) → CompiledStateGraph`

Builds and returns the compiled agent graph. Called from `server_graph.py::make_graph()`.

**Parameters:**

| Parameter | Type | Purpose |
|---|---|---|
| `config` | `ServerConfig` | Server configuration (model, auto_approve, etc.) |
| `mcp_tools` | `list[BaseTool]` | Pre-loaded MCP tool objects |
| `subagents` | `list[SubAgent]` | Subagent specs from YAML files |

**Steps:**

1. **Backend selection** — always uses `LocalShellBackend` (disk + local shell). This gives the agent full filesystem and shell access on the user's machine. For sandboxed deployments, a partner backend would be used instead.

2. **Tool assembly** — combines:
   - `fetch_url` — fetches a URL and converts HTML to Markdown (from `tools.py`)
   - `web_search` — Tavily web search (from `tools.py`, only added if `TAVILY_API_KEY` is set)
   - MCP tools — passed in from `mcp_tools` parameter

3. **Interrupt configuration** — determines which tool calls require HITL approval:
   - If `auto_approve=True` or `interrupt_shell_only=True`: no interrupts (agent runs headless)
   - Default: interrupts on `execute`, `write_file`, `edit_file`, `web_search`, `fetch_url`, `task`, `start_async_task`

4. **Middleware assembly** — adds CLI-specific middleware layers (see below)

5. **Calls `create_deep_agent()`** — passes all of the above to the SDK factory. Returns the compiled graph.

---

## Middleware Stack (CLI-specific additions)

Beyond the default SDK middleware, the CLI agent adds:

| Middleware | Purpose |
|---|---|
| `ConfigurableModelMiddleware` | Allows mid-session model switching via `/model` command |
| `TokenStateMiddleware` | Tracks context window usage; feeds the status bar token counter |
| `AskUserMiddleware` | Enables the agent to call `ask_user()` — pauses execution and prompts the user for input |
| `LocalContextMiddleware` | Injects git branch, project root, and directory tree into the system prompt |
| `SkillsMiddleware` | Loads skills from `~/.deepagents/{agent}/skills/` and project `.deepagents/skills/` |
| `MemoryMiddleware` | Loads `AGENTS.md` from `~/.deepagents/{agent}/AGENTS.md` |
| `ShellAllowListMiddleware` | Validates shell commands against the allow-list (used in non-interactive mode) |
| `SummarizationMiddleware` | Compacts conversation history when token usage is high |

---

## `load_async_subagents(config) → list[AsyncSubAgent]`

Reads `[async_subagents]` entries from `~/.deepagents/config.toml`. Each entry specifies a remote LangGraph deployment (graph_id, url, headers) that the agent can invoke asynchronously via `start_async_task`.

---

## Architecture Notes

**LocalShellBackend choice:** The CLI runs on the user's own machine with their own permissions. There is no sandbox. The HITL interrupt system is the primary safety mechanism — the user reviews and approves shell commands and file writes before they execute.

**AskUserMiddleware:** This middleware adds an `ask_user(question)` tool to the agent's tool list. When the agent calls it, the CLI pauses execution and shows `AskUserMenu` in the TUI. The user's response is returned as a `ToolMessage` and execution continues. This avoids the agent making uninformed decisions or silently failing on ambiguous requests.

**ConfigurableModelMiddleware:** Adds a `configurable` field to the LangGraph state that the `/model` command can update. On the next model call, the middleware swaps in the new model. This enables mid-session model changes without restarting the server.

---

## See Also

- [server_graph.md](server_graph.md) — calls `create_cli_agent()`
- [tools.md](tools.md) — `fetch_url` and `web_search` definitions
- [mcp_tools.md](mcp_tools.md) — how MCP tools are loaded before being passed here
- [../../deepagents/deepagents/graph.md](../../deepagents/deepagents/graph.md) — the SDK `create_deep_agent()` called here
