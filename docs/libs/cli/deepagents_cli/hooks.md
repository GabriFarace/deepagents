# `hooks.py`

## High-Level Purpose

This module provides a lightweight hook dispatch system for external tool integration. It fires external commands with JSON payloads on stdin when specific CLI events occur. Background thread execution ensures the main async event loop is never stalled. Failures are logged but never propagate to callers.

Hook configuration is read from `~/.deepagents/hooks.json`.

## Configuration Format

```json
{
  "hooks": [
    {
      "command": ["bash", "adapter.sh"],
      "events": ["session.start", "session.end"]
    },
    {
      "command": ["python", "my_hook.py"]
    }
  ]
}
```

- If `events` is omitted or empty, the hook receives **all** events.
- `command` must be a non-empty list of strings.

## Module-Level State

| Variable | Type | Description |
|---|---|---|
| `_hooks_config` | `list[dict] \| None` | Cached hook definitions, loaded lazily on first dispatch |
| `_background_tasks` | `set[asyncio.Task]` | Strong references to fire-and-forget tasks to prevent GC |

## Functions

### `_load_hooks() -> list[dict]`

Loads and caches hook definitions from `~/.deepagents/hooks.json`. Returns an empty list if the file is missing or malformed, ensuring normal execution is never interrupted.

**Returns:** List of hook definition dicts.

### `_run_single_hook(command: list[str], event: str, payload_bytes: bytes) -> None`

Executes a single hook command, writing the JSON payload to its stdin. Uses `subprocess.run` with a 5-second timeout to prevent zombie processes.

**Parameters:**
- `command`: The command and arguments to run.
- `event`: Event name (for logging).
- `payload_bytes`: JSON payload to write to stdin.

### `_dispatch_hook_sync(event: str, payload_bytes: bytes, hooks: list[dict]) -> None`

Dispatches all matching hooks concurrently via a `ThreadPoolExecutor`. Hooks whose event filter doesn't match are skipped. Errors per hook are caught and logged without propagating.

**Parameters:**
- `event`: Dotted event name (e.g., `'session.start'`).
- `payload_bytes`: JSON payload bytes.
- `hooks`: List of hook definition dicts.

### `dispatch_hook(event: str, payload: dict) -> None`

The public async dispatch function. Loads hooks, serializes the payload to JSON bytes, and fires matching hooks in a background thread (via `asyncio.get_event_loop().run_in_executor`). Returns immediately.

**Parameters:**
- `event`: Dotted event name (e.g., `'session.start'`).
- `payload`: Dictionary to serialize as JSON and send to each matching hook's stdin.

**Common events:**
- `session.start` — Fired when a CLI session begins.
- `session.end` — Fired when a CLI session ends.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `asyncio` | stdlib | Background task scheduling |
| `json` | stdlib | Payload serialization |
| `subprocess` | stdlib | Hook command execution |
| `concurrent.futures.ThreadPoolExecutor` | stdlib | Concurrent hook execution |
| `DEFAULT_CONFIG_DIR` | `deepagents_cli.model_config` | Config directory path |
