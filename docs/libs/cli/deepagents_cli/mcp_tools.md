# `mcp_tools.py`

## High-Level Purpose

This module provides MCP (Model Context Protocol) tool loading and management for the deepagents CLI. It:

- Loads and validates MCP server configurations from JSON files
- Supports automatic discovery of `.mcp.json` files from user-level and project-level locations
- Connects to MCP servers (stdio, SSE, HTTP transports) and collects tool metadata
- Supports **OAuth login** for remote servers (`auth: "oauth"`)
- Supports **tool filtering** via `allowedTools` and `disabledTools` per-server config
- Manages persistent MCP sessions via `MCPSessionManager`

## Classes

### `MCPToolInfo`

**Type:** `dataclass`

Metadata for a single MCP tool.

| Attribute | Type | Description |
|---|---|---|
| `name` | `str` | Tool name (may include server name prefix) |
| `description` | `str` | Human-readable description |

### `MCPServerInfo`

**Type:** `dataclass`

Metadata for a connected MCP server and its tools.

| Attribute | Type | Description |
|---|---|---|
| `name` | `str` | Server name from the MCP configuration |
| `transport` | `str` | Transport type (`"stdio"`, `"sse"`, `"http"`, or `"config"` for bad config files) |
| `tools` | `tuple[MCPToolInfo, ...]` | Tools exposed by this server (empty if `status != "ok"`) |
| `status` | `MCPServerStatus` | `"ok"`, `"unauthenticated"`, or `"error"` |
| `error` | `str \| None` | Human-readable reason when status is not `"ok"` |

### `MCPSessionManager`

Lazy, per-server cache of persistent MCP sessions.

- **Discovery** always uses throwaway sessions (no caching for one-shot tool listing).
- **Runtime tools** bind to a caller-managed `MCPSessionManager` (server mode) or create a local manager, or stay stateless.
- Callers must call `session_manager.cleanup()` when done.

## Module-Level Constants

| Constant | Value | Description |
|---|---|---|
| `_SUPPORTED_REMOTE_TYPES` | `{"sse", "http"}` | Supported remote transport types |

## Functions

### `_resolve_server_type(server_config: dict) -> str`

Determines the transport type for a server config. Supports both `type` and `transport` field names, defaulting to `"stdio"`.

### `_validate_server_config(server_name: str, server_config: dict) -> None`

Validates a single server's configuration dictionary.

**Validation rules by transport:**
- `stdio`: Requires `command`. Optional `args` (list) and `env` (dict).
- `sse`/`http`: Requires `url`. Optional `headers` (dict).

**Raises:** `TypeError` for wrong types; `ValueError` for missing/invalid fields.

### `_validate_tool_filter_fields(server_name: str, server_config: dict) -> None`

Validates `allowedTools` and `disabledTools` fields.

**Rules:**
- Both fields cannot be set on the same server (`ValueError`).
- Empty lists are rejected (`ValueError`).
- Entries are literal tool names or `fnmatch`-style glob patterns (`*`, `?`, `[`).
- Matched against both the bare tool name and the server-prefixed form (`f"{server_name}_{tool}"`).

### `load_mcp_config(config_path: str) -> dict`

Loads and validates an MCP configuration from a JSON file.

**Returns:** Parsed configuration dictionary.

**Supported server config format:**
```json
{
  "mcpServers": {
    "my-tool": {
      "command": "python",
      "args": ["-m", "my_tool"],
      "allowedTools": ["read_file", "grep_*"]
    },
    "remote-api": {
      "type": "sse",
      "url": "https://example.com/mcp",
      "auth": "oauth"
    },
    "rest-api": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {"Authorization": "Bearer ..."},
      "disabledTools": ["dangerous_tool"]
    }
  }
}
```

### `discover_mcp_configs(project_context: ProjectContext | None = None) -> list[tuple[str, str]]`

Discovers `.mcp.json` config files from standard locations in precedence order (highest to lowest):

1. `~/.deepagents/.mcp.json` (user-level)
2. `<project-root>/.deepagents/.mcp.json` (project subdir)
3. `<project-root>/.mcp.json` (Claude Code compatibility)

`merge_mcp_configs()` merges multiple dicts; later entries override earlier ones by server name.

**Returns:** List of `(config_path, source)` tuples for files that exist.

### `build_oauth_provider(server_name: str, server_url: str) -> OAuthClientProvider`

Builds an `OAuthClientProvider` for a remote server's OAuth flow.

- Tokens stored on disk at `~/.deepagents/tokens/{server_name}.json` via `FileTokenStorage`.
- Interactive reauth when refresh fails; displays `"Run: deepagents mcp login {server_name}"`.
- Only valid for remote transports (`http`, `sse`). Raises `ValueError` for `stdio` servers.
- Cannot be combined with an explicit `Authorization` header in `headers`.

### `resolve_and_load_mcp_tools(*, explicit_config_path, no_mcp, trust_project_mcp, project_context) -> tuple[list[BaseTool], MCPSessionManager, list[MCPServerInfo]]`

The main async entry point for loading all MCP tools. Discovers configs, prompts for project-stdio trust if needed, connects to all servers, and returns the tools with session management.

**Returns:** `(tools, session_manager, server_info_list)`

Servers with status `"unauthenticated"` are skipped (not loaded as tools). Servers with status `"error"` are also skipped but logged.

## OAuth Login Flow

```
# User runs:
deepagents mcp login <server_name>

# Or the CLI prompts automatically when:
# - Server config has auth: "oauth"
# - Token is expired or missing
```

OAuth tokens are cached on disk and refreshed automatically on subsequent runs. If refresh fails, the server is reported as `"unauthenticated"` in `MCPServerInfo`.

## Tool Filtering

`allowedTools` and `disabledTools` support glob patterns:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "my-server",
      "allowedTools": ["read_*", "list_files"]
    }
  }
}
```

Patterns are matched against both the bare tool name (e.g., `"read_file"`) and the server-prefixed name (e.g., `"my_server_read_file"`).

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `langchain_mcp_adapters` | `langchain-mcp-adapters` | MCP client implementation |
| `contextlib.AsyncExitStack` | stdlib | Manages async MCP sessions |
| `json`, `pathlib.Path` | stdlib | Config loading |
| `fnmatch` | stdlib | Glob pattern matching for tool filters |
| `ProjectContext` | `deepagents_cli.project_utils` | Project root and context |
