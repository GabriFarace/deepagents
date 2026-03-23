# `langchain_runloop/sandbox.py`

## High-Level Purpose

Provides `RunloopSandbox`, a sandbox backend implementation that wraps a [Runloop](https://runloop.ai/) devbox. Enables Deep Agents to execute shell commands and transfer files inside a Runloop development environment.

## Classes

### `RunloopSandbox`

**Purpose:** Wraps an existing Runloop `Devbox` instance, implementing `BaseSandbox`.

**Inherits from:** `deepagents.backends.sandbox.BaseSandbox`

#### `__init__(*, devbox: Devbox) -> None`

**Parameters:**
- `devbox`: Existing Runloop `Devbox` instance to wrap.

**Key Logic:** Stores `devbox.id` immediately (so the ID is stable even if the devbox object is later refreshed), sets `_default_timeout` to 30 minutes.

#### `id -> str` (property)
Returns `self._devbox_id`.

#### `execute(command: str, *, timeout: int | None = None) -> ExecuteResponse`

**Purpose:** Execute a shell command using `devbox.cmd.exec()`.

**Parameters:**
- `command`: Shell command string.
- `timeout`: Override timeout in seconds. Falls back to `_default_timeout`.

**Return Value:** `ExecuteResponse(output, exit_code, truncated=False)`.

**Key Logic:**
1. Calls `self._devbox.cmd.exec(command, timeout=effective_timeout)`.
2. Reads stdout and stderr from the result.
3. Appends stderr to stdout with a newline if both are non-empty.

#### `download_files(paths: list[str]) -> list[FileDownloadResponse]`

Downloads each file using `devbox.file.download(path=path)`. Returns content bytes directly.

#### `upload_files(files: list[tuple[str, bytes]]) -> list[FileUploadResponse]`

Uploads each file using `devbox.file.upload(path=path, file=content)`. Always returns success.

**Note:** Shell-based file operations (read, write, edit, ls, glob, grep) are inherited from `BaseSandbox` and run through `execute()`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `Devbox` | `runloop_api_client.sdk` (TYPE_CHECKING) | Runloop devbox type |
| `BaseSandbox` | `deepagents.backends.sandbox` | Base class with shell-based file operations |
| Protocol types | `deepagents.backends.protocol` | Response types |
