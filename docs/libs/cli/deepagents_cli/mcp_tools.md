# `mcp_tools.py`

## High-Level Purpose

This module provides MCP (Model Context Protocol) tool loading and management for the deepagents CLI. It:

- Loads and validates MCP server configurations from JSON files (Claude Desktop format)
- Supports automatic discovery of `.mcp.json` files from user-level and project-level locations
- Connects to MCP servers (stdio, SSE, HTTP transports) and collects tool metadata
- Provides trust prompting for project-level stdio MCP servers

MCP enables agents to use external tools provided by local processes or remote HTTP/SSE servers.

## Classes

### `MCPToolInfo`

**Type:** `dataclass`

Metadata for a single MCP tool.

| Attribute | Type | Description |
|---|---|---|
| `name` | `str` | Tool name (may include server name prefix) |
| `description` | `str` | Human-readable description of the tool |

### `MCPServerInfo`

**Type:** `dataclass`

Metadata for a connected MCP server and its tools.

| Attribute | Type | Description |
|---|---|---|
| `name` | `str` | Server name from the MCP configuration |
| `transport` | `str` | Transport type (`stdio`, `sse`, or `http`) |
| `tools` | `list[MCPToolInfo]` | Tools exposed by this server |

## Module-Level Constants

| Constant | Value | Description |
|---|---|---|
| `_SUPPORTED_REMOTE_TYPES` | `{"sse", "http"}` | Supported remote transport types |

## Functions

### `_resolve_server_type(server_config: dict) -> str`

Determines the transport type for a server config. Supports both `type` and `transport` field names, defaulting to `"stdio"`.

**Returns:** Transport type string.

### `_validate_server_config(server_name: str, server_config: dict) -> None`

Validates a single server's configuration dictionary.

**Raises:**
- `TypeError`: If config fields have wrong types.
- `ValueError`: If required fields are missing or transport type is unsupported.

**Validation rules by transport:**
- `stdio`: Requires `command`. Optional `args` (list) and `env` (dict).
- `sse`/`http`: Requires `url`. Optional `headers` (dict).

### `load_mcp_config(config_path: str) -> dict`

Loads and validates an MCP configuration from a JSON file (Claude Desktop format).

**Parameters:**
- `config_path`: Path to the MCP JSON config file.

**Returns:** Parsed configuration dictionary.

**Raises:**
- `FileNotFoundError`: If the config file doesn't exist.
- `json.JSONDecodeError`: If the file contains invalid JSON.
- `TypeError`/`ValueError`: If the config structure is invalid.

**Supported server config formats:**
```json
{
  "mcpServers": {
    "my-tool": {"command": "python", "args": ["-m", "my_tool"]},
    "remote-api": {"type": "sse", "url": "https://example.com/mcp"},
    "rest-api": {"type": "http", "url": "https://api.example.com/mcp", "headers": {"Authorization": "Bearer ..."}
    }
  }
}
```

### `discover_mcp_configs(project_context: ProjectContext | None = None) -> list[tuple[str, str]]`

Discovers `.mcp.json` config files from standard locations (user-level `~/.deepagents/.mcp.json` and project-level `.deepagents/.mcp.json`).

**Returns:** List of `(config_path, source)` tuples.

### `resolve_and_load_mcp_tools(*, explicit_config_path, no_mcp, trust_project_mcp, project_context) -> tuple[list[BaseTool], SessionManager, list[MCPServerInfo]]`

The main async entry point for loading all MCP tools. Discovers configs, prompts for project-stdio trust if needed, connects to all servers, and returns the tools with session management.

**Returns:** `(tools, session_manager, server_info_list)`.

The `session_manager` must be kept alive as long as the MCP tools are in use. Call `session_manager.cleanup()` when done.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `langchain_mcp_adapters.client.MultiServerMCPClient` | `langchain-mcp-adapters` | MCP client implementation |
| `contextlib.AsyncExitStack` | stdlib | Manages async MCP sessions |
| `json`, `pathlib.Path` | stdlib | Config loading |
| `ProjectContext` | `deepagents_cli.project_utils` | Project root and context |
