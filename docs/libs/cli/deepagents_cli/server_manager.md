# `libs/cli/deepagents_cli/server_manager.py`

> Parent-side orchestration for launching the local LangGraph server and
> returning a connected `RemoteAgent`.

## Position in the system

`main.py` calls `server_session()`. This module serializes CLI choices into
`DEEPAGENTS_CLI_SERVER_*` env vars, creates the temporary server workspace,
starts `ServerProcess`, and yields the remote client.

## Functions and classes

### `_set_or_clear_server_env()` and `_apply_server_config()`

Write the env vars consumed by `server_graph.py` through
`ServerConfig.from_env()`. `ServerConfig.to_env()` is the single serialization
source of truth.

### Workspace scaffolding helpers

`_capture_project_context()`, `_scaffold_workspace()`, `_write_checkpointer()`,
and `_write_pyproject()` copy the server graph, create SQLite checkpointer
glue, and make the temp directory installable by `langgraph dev`.

### `_preflight_validate_mcp_config(...)`

Validates explicit MCP config in the parent process so malformed configs become
clean UI errors instead of opaque server startup failures.

### `start_server_and_get_agent(...)`

Builds `ServerConfig`, writes env, validates MCP, scaffolds workspace, starts
the server, and returns `RemoteAgent`.

### `server_session(...)`

Async context manager that guarantees server cleanup after interactive,
non-interactive, or ACP execution.

## Gotchas

The generated checkpointer module reads the database path from an env var at
runtime, avoiding hard-coded user paths in generated source.
