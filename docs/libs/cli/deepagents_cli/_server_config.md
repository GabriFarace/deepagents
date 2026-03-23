# `_server_config.py`

## High-Level Purpose

This module defines typed configuration for the CLI-to-server subprocess communication channel. The CLI spawns a `langgraph dev` subprocess and passes configuration via environment variables prefixed with `DA_SERVER_`. This module provides a single `ServerConfig` dataclass shared by both sides so that the set of variables, their serialization format, and their default values are defined in one place.

**Write path:** CLI calls `ServerConfig.to_env()` before launching the subprocess.
**Read path:** Server graph calls `ServerConfig.from_env()` at startup.

## Classes

### `ServerConfig`

**Type:** `dataclass`

Typed configuration for the `langgraph dev` server process.

| Attribute | Type | Description |
|---|---|---|
| `assistant_id` | `str` | Agent/assistant identifier (default: `"agent"`) |
| `auto_approve` | `bool` | Whether to auto-approve tool calls |
| `shell_allow_list` | `list[str] \| None` | Shell commands to auto-approve |
| `model` | `str \| None` | Model spec to use (e.g., `'anthropic:claude-sonnet-4-6'`) |
| `model_params` | `dict[str, Any] \| None` | Extra model kwargs |
| `profile_override` | `dict[str, Any] \| None` | Model profile field overrides |
| `mcp_config_path` | `str \| None` | Path to explicit MCP config |
| `no_mcp` | `bool` | Whether to disable MCP |
| `trust_project_mcp` | `bool \| None` | Trust decision for project-level MCP |
| `project_context` | `ProjectContext \| None` | Project context for the server |
| `sandbox` | `str \| None` | Sandbox type |
| `sandbox_id` | `str \| None` | Existing sandbox ID to reuse |

**Methods:**

#### `to_env() -> dict[str, str]`

Serializes the config to a dict of `DA_SERVER_*` environment variable key/value pairs. Booleans become `"true"`/`"false"` strings. Complex objects are JSON-encoded.

**Returns:** Dict of environment variable strings.

#### `from_env() -> ServerConfig`

Class method that reconstructs a `ServerConfig` by reading `DA_SERVER_*` environment variables from `os.environ`.

**Returns:** Populated `ServerConfig` instance.

## Module-Level Helpers

### `_read_env_bool(suffix: str, *, default: bool = False) -> bool`

Reads a `DA_SERVER_*` boolean from the environment. Uses `"true"` / `"false"` convention (case-insensitive).

### `_read_env_json(suffix: str) -> Any`

Reads a JSON-encoded `DA_SERVER_*` variable. Returns `None` if absent.

**Raises:** `ValueError` if the variable is present but not valid JSON.

### `_read_env_str(suffix: str) -> str | None`

Reads an optional `DA_SERVER_*` string variable.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `json`, `os` | stdlib | Environment variable reading/writing |
| `_server_constants.ENV_PREFIX` | local | `"DA_SERVER_"` prefix constant |
| `ProjectContext` | `deepagents_cli.project_utils` | Project context type |
