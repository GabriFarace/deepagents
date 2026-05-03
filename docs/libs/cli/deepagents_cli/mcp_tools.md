# `libs/cli/deepagents_cli/mcp_tools.py`

## High-Level Purpose

`mcp_tools.py` handles everything related to Model Context Protocol (MCP) servers: discovering config files, validating them, establishing connections to MCP servers, loading their tool lists, and converting MCP tools into LangChain `BaseTool` objects that the agent can call. It runs both in the TUI process (for pre-flight display) and in the server subprocess (for actual agent use).

---

## Key Functions

### `resolve_and_load_mcp_tools(config_path=None) → tuple[list[BaseTool], MCPSessionManager | None, list[MCPServerInfo]]`

The main entry point. Discovers the MCP config file (if no explicit path is given), validates it, connects to all configured servers, and returns tools + session manager + server info.

**Config discovery order** (highest to lowest priority):

1. Explicit `config_path` argument (from `--mcp-config` flag)
2. Project `.deepagents/.mcp.json` (in the current working directory tree)
3. Project `.agents/.mcp.json`
4. User-level `~/.deepagents/.mcp.json`

Returns `([], None, [])` if no config is found — MCP is purely optional.

### `_preflight_validate_mcp_config(config_path) → list[str]`

Runs in the **TUI process** before the server starts. Returns a list of error strings. Catches:

- Malformed JSON
- Missing required fields (`command`, `args` for stdio; `url` for sse/http)
- Unknown transport types
- Invalid server names

Validation errors are shown in the TUI's startup overlay before the server even launches, giving fast feedback.

### `_check_mcp_project_trust(config_path) → bool`

Project-level MCP configs (`.deepagents/.mcp.json`) require explicit user approval before their servers are started — they could run arbitrary commands. Trust is recorded in `~/.deepagents/mcp_trust.db` (keyed by config file hash). Returns `True` if trusted, `False` if not. When `False`, the TUI shows a trust prompt.

---

## MCP Config Format

`~/.deepagents/.mcp.json` or `.deepagents/.mcp.json`:

```json
{
  "mcpServers": {
    "github": {
      "transport": "stdio",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {"GITHUB_TOKEN": "ghp_..."}
    },
    "my-api": {
      "transport": "http",
      "url": "http://localhost:8000/mcp"
    },
    "analytics": {
      "transport": "sse",
      "url": "https://analytics.example.com/mcp/sse"
    }
  }
}
```

**Transport types:**

| Transport | Connection method | Use case |
|---|---|---|
| `stdio` | Spawns a subprocess, communicates over stdin/stdout | Local tools (npm packages, Python scripts) |
| `sse` | HTTP GET with Server-Sent Events streaming | Remote services with streaming |
| `http` | HTTP POST (request/response) | Remote services without streaming |

---

## `MCPSessionManager`

Manages the lifecycle of persistent MCP server connections in the LangGraph server subprocess. Created once at server startup, used for all agent calls in the session.

**Key methods:**

- `async connect_all()` — Starts all configured MCP servers and establishes sessions
- `async list_tools() → list[MCPToolInfo]` — Returns aggregated tool list from all connected servers
- `async cleanup()` — Closes all sessions and stops stdio subprocesses

MCP tools are wrapped as LangChain `StructuredTool` objects with the tool's schema from the MCP server's tool registration.

---

## `MCPServerInfo`

A dataclass with display-friendly metadata about a configured server:

| Field | Purpose |
|---|---|
| `name` | Server name from config |
| `transport` | `"stdio"`, `"sse"`, or `"http"` |
| `tool_count` | Number of tools loaded |
| `status` | `"connected"`, `"error"`, `"not_started"` |
| `error` | Error message if `status == "error"` |

Used by the `/mcp` slash command to show server status in the `MCPViewer` modal.

---

## Architecture Notes

**Two-phase loading:** MCP tools are loaded twice. First in the TUI process (stateless, for pre-flight display and validation). Then in the server subprocess (persistent, for actual agent use). The TUI uses `stateless=True` mode which connects, reads tool lists, and immediately disconnects — it doesn't keep sessions alive.

**Tool name scoping:** If two MCP servers define a tool with the same name, the second one wins (last server in config wins). Consider using unique server names to avoid conflicts.

**stdio security:** Stdio MCP servers run as child processes of the LangGraph server subprocess. They inherit the server's environment. The trust check for project configs guards against untrusted `.mcp.json` files from a cloned repo running arbitrary commands.

---

## See Also

- [server_graph.md](server_graph.md) — calls `resolve_and_load_mcp_tools()` at server startup
- [agent.md](agent.md) — receives MCP tools and adds them to the agent's tool list
- [mcp_trust.md](mcp_trust.md) — trust database management (separate file)
- [widgets/README.md](widgets/README.md) — `MCPViewer` modal for `/mcp` command
