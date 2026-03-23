# `server.py`

## High-Level Purpose

This module manages the lifecycle of a `langgraph dev` server subprocess. It is responsible for:

- Starting, monitoring, and stopping the LangGraph development server
- Generating the `langgraph.json` configuration file required by `langgraph dev`
- Port management (finding free ports, checking in-use ports)
- Scoped environment variable management for subprocess isolation
- Health polling until the server is ready to accept requests

The CLI spawns a `langgraph dev` process for interactive sessions so that the agent runs in a separate process with proper LangGraph server semantics (persistent checkpoints, LangSmith tracing, etc.).

## Module-Level Constants

| Constant | Value | Description |
|---|---|---|
| `_DEFAULT_HOST` | `"127.0.0.1"` | Default server host |
| `_DEFAULT_PORT` | `2024` | Default server port |
| `_HEALTH_POLL_INTERVAL` | `0.3` | Seconds between health check polls |
| `_HEALTH_TIMEOUT` | `60` | Maximum seconds to wait for server to become healthy |
| `_SHUTDOWN_TIMEOUT` | `5` | Seconds to wait for graceful shutdown |

## Functions

### `_port_in_use(host: str, port: int) -> bool`

Checks whether a TCP port is already bound.

**Parameters:**
- `host`: Host to check.
- `port`: Port to check.

**Returns:** `True` if the port is in use.

### `_find_free_port(host: str) -> int`

Finds an available TCP port by binding to port 0 (OS assigns a free port).

**Parameters:**
- `host`: Host to bind to.

**Returns:** An available port number.

### `get_server_url(host: str = _DEFAULT_HOST, port: int = _DEFAULT_PORT) -> str`

Builds the base URL for the LangGraph server.

**Returns:** URL string like `"http://127.0.0.1:2024"`.

### `generate_langgraph_json(output_dir, *, graph_ref, env_file=None, checkpointer_path=None) -> Path`

Generates the `langgraph.json` configuration file needed by `langgraph dev`.

**Parameters:**
- `output_dir`: Directory to write the config file.
- `graph_ref`: Python module:variable reference (default `"./server_graph.py:graph"`).
- `env_file`: Optional path to an env file to pass to the server.
- `checkpointer_path`: Import path to an async context manager yielding a `BaseCheckpointSaver`. When set, the server persists checkpoints to disk instead of in-memory.

**Returns:** Path to the generated `langgraph.json` file.

**Generated structure:**
```json
{
  "dependencies": ["."],
  "graphs": {"agent": "./server_graph.py:graph"},
  "env": "...",
  "checkpointer": {"path": "..."}
}
```

### `_scoped_env_overrides(overrides: dict[str, str]) -> Iterator[None]`

Context manager that applies environment variable overrides. On **normal exit**, the overrides are left in place (caller "keeps" them). On **exception**, the previous values are restored.

**Parameters:**
- `overrides`: Key/value pairs to set in `os.environ`.

### `start_server_and_get_agent(...) -> tuple[ServerProcess, RemoteAgent]`

Main server startup function. Creates a temp directory, generates `langgraph.json`, starts a `langgraph dev` subprocess, polls the health endpoint until ready, and returns a `(ServerProcess, RemoteAgent)` tuple.

## Classes

### `ServerProcess`

Represents a running `langgraph dev` subprocess. Manages the process lifecycle.

**Key Methods:**
- `start()` — Launch the subprocess.
- `stop()` — Send SIGTERM and wait for graceful shutdown (with `_SHUTDOWN_TIMEOUT`).
- `is_alive() -> bool` — Check if the subprocess is still running.
- `poll()` — Check subprocess exit status without blocking.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `asyncio`, `subprocess`, `signal` | stdlib | Async I/O and process management |
| `contextlib`, `json`, `os`, `tempfile` | stdlib | Config generation and env management |
| `socket` | stdlib | Port availability checking |
