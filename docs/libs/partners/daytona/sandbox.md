# `langchain_daytona/sandbox.py`

## High-Level Purpose

Provides `DaytonaSandbox`, a sandbox backend implementation that wraps a Daytona cloud development environment. Enables Deep Agents to execute shell commands and transfer files in a Daytona sandbox.

## Module-Level Types

| Type Alias | Definition | Description |
|------------|------------|-------------|
| `SyncPollingInterval` | `float \| Callable[[float], float]` | Either a fixed polling delay or a function of elapsed time |
| `PollingStrategy` | `Callable[[float], float]` | Normalized polling strategy: always a callable |

## Classes

### `DaytonaSandbox`

**Purpose:** Wraps an existing Daytona `Sandbox` instance, implementing `BaseSandbox` (which extends `SandboxBackendProtocol`).

**Inherits from:** `deepagents.backends.sandbox.BaseSandbox`

#### `__init__`

```python
def __init__(
    self,
    *,
    sandbox: daytona.Sandbox,
    timeout: int = 30 * 60,
    sync_polling_interval: SyncPollingInterval = 0.1,
) -> None
```

**Parameters:**
- `sandbox`: Existing Daytona sandbox instance to wrap.
- `timeout`: Default command timeout in seconds (default 30 minutes). A value of `0` means "wait indefinitely".
- `sync_polling_interval`: Polling delay between completion checks. Can be a fixed float (seconds) or a callable `(elapsed_seconds) -> delay_seconds`.

**Key Logic:** Normalizes `sync_polling_interval` into a callable `PollingStrategy` regardless of whether a float or callable was provided.

#### `id -> str` (property)
Returns `self._sandbox.id`.

#### `execute(command: str, *, timeout: int | None = None) -> ExecuteResponse`

**Purpose:** Execute a shell command in the Daytona sandbox synchronously.

**Parameters:**
- `command`: Shell command string.
- `timeout`: Override timeout. Falls back to `self._default_timeout` if `None`.

**Return Value:** `ExecuteResponse(output, exit_code, truncated=False)`.

**Key Logic:** Delegates to `_execute_via_session_logs()`.

#### `_execute_via_session_logs(command: str, *, timeout: int) -> ExecuteResponse`

**Purpose:** Internal implementation. Creates a session, executes the command asynchronously in Daytona, and polls until completion.

**Key Logic:**
1. Creates a new Daytona process session with a UUID ID.
2. Executes the command with `run_async=True`.
3. Polls `get_session_command()` using the configured polling strategy, checking elapsed time against the timeout.
4. On timeout: returns `ExecuteResponse(output="Command timed out...", exit_code=124)`.
5. Fetches logs and concatenates stdout + stderr (wrapped in `<stderr>` tags).
6. Always deletes the session in the `finally` block.

#### `download_files(paths: list[str]) -> list[FileDownloadResponse]`

Downloads files from the sandbox using `sandbox.fs.download_files()`. Paths not starting with `/` receive an `"invalid_path"` error. Maps Daytona responses to `FileDownloadResponse` objects.

#### `upload_files(files: list[tuple[str, bytes]]) -> list[FileUploadResponse]`

Uploads files using `sandbox.fs.upload_files()`. Paths not starting with `/` receive an `"invalid_path"` error.

**Note:** File operation methods (read, write, edit, ls, glob, grep) are inherited from `BaseSandbox` and execute through the `execute()` method using shell commands.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `daytona` | `daytona` | Daytona SDK — `Sandbox`, `SessionExecuteRequest`, etc. |
| `BaseSandbox` | `deepagents.backends.sandbox` | Base class with shell-based file operations |
| `ExecuteResponse`, `FileDownloadResponse`, `FileUploadResponse` | `deepagents.backends.protocol` | Response types |
| `time`, `uuid4` | stdlib | Polling timer and session ID generation |
