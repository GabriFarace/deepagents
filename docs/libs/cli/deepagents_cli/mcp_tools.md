# `libs/cli/deepagents_cli/mcp_tools.py`

> MCP config discovery, validation, session management, tool discovery, and
> LangChain tool wrapping.

## Position in the system

`server_graph.py` loads MCP tools during graph construction and passes them to
the CLI agent.

## Functions and classes

### Metadata and errors

`MCPToolInfo`, `MCPServerInfo`, `MCPConfigError`, and `_MCPSessionEntry`
represent UI metadata, validation errors, and cached sessions.

### `MCPSessionManager`

Owns long-lived MCP sessions and reuses them across tool calls in the server
process.

### Config helpers

Server type resolution, config validation, discovery/classification, merge, and
lenient loading helpers manage explicit/user/project MCP files.

### Tool loading helpers

Server health checks, `_discover_tools()`, `_build_cached_mcp_tool()`, tool
filtering, `_load_tools_from_config()`, `get_mcp_tools()`, and
`resolve_and_load_mcp_tools()` turn MCP servers into callable LangChain tools.

## Gotchas

Explicit MCP config errors are fatal; discovered optional configs are handled
more leniently.
