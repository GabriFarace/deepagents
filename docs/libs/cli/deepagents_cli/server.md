# `libs/cli/deepagents_cli/server.py`

## High-Level Purpose

`server.py` contains `ServerProcess`, the class that manages the `langgraph dev` subprocess. It handles starting, monitoring, and stopping the server process, as well as capturing its stdout/stderr for display in the TUI. It is a thin wrapper around `asyncio.create_subprocess_exec` with readiness detection and clean shutdown.

---

## Key Class

### `ServerProcess`

Represents the lifecycle of a single `langgraph dev` process.

**Constructor parameters:**

| Parameter | Type | Purpose |
|---|---|---|
| `workspace_dir` | `Path` | Directory where `langgraph.json` lives |
| `port` | `int` | Port to bind the server to |
| `env` | `dict[str, str]` | Environment variables for the subprocess |

**Key methods:**

#### `async start() → None`

Starts the server subprocess:
```
uv run langgraph dev --host 127.0.0.1 --port <port> --no-browser
```
Sets up async readers for stdout and stderr. Sends server logs to a ring buffer (accessible as `self.recent_logs`) and optionally to the TUI status area.

#### `async wait_until_ready(timeout_s=30) → bool`

Polls `GET http://127.0.0.1:{port}/ok` every 200ms. Returns `True` when the server responds with HTTP 200. Returns `False` if `timeout_s` expires.

The readiness check also monitors the subprocess exit code — if the process dies during startup (e.g., bad `langgraph.json`), `wait_until_ready` returns `False` immediately so the TUI can report an error rather than spinning.

#### `async stop() → None`

Sends `SIGTERM` to the subprocess and waits up to 5 seconds. If the process doesn't exit, sends `SIGKILL`. Called from `CLIApp.on_unmount()` during TUI shutdown.

**Properties:**

- `is_running: bool` — `True` if the subprocess is alive
- `recent_logs: list[str]` — Last N lines of combined stdout+stderr (used for error reporting)
- `url: str` — `http://127.0.0.1:{port}`

---

## Architecture Notes

**Log capture:** Stdout and stderr are read line-by-line in async tasks. Lines matching `INFO:` or `ERROR:` patterns from uvicorn/langgraph are parsed and displayed in the TUI's startup overlay. This gives users visibility into server startup without cluttering the main transcript.

**Exit detection:** An async watchdog task monitors `process.returncode`. If the server crashes after startup, it posts a `ServerCrashed` message to the TUI, which shows an error banner and disables the input field.

---

## See Also

- [server_manager.md](server_manager.md) — creates `ServerProcess` and calls `start()`
- [server_graph.md](server_graph.md) — code that runs inside the server process
