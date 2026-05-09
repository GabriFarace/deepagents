# `libs/cli/deepagents_cli/server_graph.py`

> Graph factory loaded inside the `langgraph dev` subprocess.

## Position in the system

The parent process copies this file into the temp workspace and points
`langgraph.json` at `./server_graph.py:graph`. Importing this module constructs
the graph.

## Functions and classes

### `_get_mcp_session_manager()`

Lazily creates the process-wide MCP session manager. It lives in the server
event loop and is reused by MCP tool calls.

### `_build_tools(config, project_context)`

Creates built-in tools (`fetch_url`, optional `web_search`) and, unless MCP is
disabled, resolves MCP tools and server metadata. MCP discovery during import is
stateless; runtime sessions are opened lazily.

### `make_graph()`

Reads `ServerConfig.from_env()`, reloads settings from project context, creates
the model, builds tools, creates sandbox backend when configured, loads async
subagents, and calls `create_cli_agent()`.

### Module-level `graph`

`graph = make_graph()` is evaluated at import time. Failures are logged and
printed to stderr so parent health checks can surface startup errors.

## Gotchas

Globals here live in the server subprocess only. The Textual process cannot
read them directly.
