# `langchain_modal/sandbox.py`

## High-Level Purpose

Provides `ModalSandbox`, a sandbox backend implementation that wraps a [Modal](https://modal.com/) sandbox. Enables Deep Agents to execute shell commands and read/write files inside a Modal cloud sandbox.

## Classes

### `ModalSandbox`

**Purpose:** Wraps an existing `modal.Sandbox` instance, implementing `BaseSandbox`.

**Inherits from:** `deepagents.backends.sandbox.BaseSandbox`

#### `__init__(*, sandbox: modal.Sandbox) -> None`
Stores the Modal sandbox instance. Sets `_default_timeout` to 30 minutes.

#### `id -> str` (property)
Returns `self._sandbox.object_id`.

#### `execute(command: str, *, timeout: int | None = None) -> ExecuteResponse`

**Purpose:** Execute a shell command using `sandbox.exec("bash", "-c", command)`.

**Parameters:**
- `command`: Shell command string.
- `timeout`: Override timeout. Falls back to `_default_timeout` if `None`. A value of `0` means "wait indefinitely" in Modal's implementation.

**Return Value:** `ExecuteResponse(output, exit_code, truncated=False)`.

**Key Logic:**
1. Calls `sandbox.exec("bash", "-c", command, timeout=...)` and waits for completion.
2. Reads stdout and stderr.
3. Appends stderr to stdout with a newline separator if both are present.

#### `download_files(paths: list[str]) -> list[FileDownloadResponse]`
Delegates each path to `_read_file()`.

#### `upload_files(files: list[tuple[str, bytes]]) -> list[FileUploadResponse]`
Delegates each `(path, content)` pair to `_write_file()`.

#### `_read_file(path: str) -> FileDownloadResponse`

Opens a file via `sandbox.open(path, "rb")` and reads its content. Handles:
- `FileNotFoundError` → `error="file_not_found"`
- `modal.exception.FilesystemExecutionError` for directory errors → `error="is_directory"`
- `memoryview` content is converted to `bytes`.

#### `_write_file(path: str, content: bytes) -> FileUploadResponse`

Opens a file via `sandbox.open(path, "wb")` and writes bytes. Handles:
- `PermissionError` → `error="permission_denied"`
- `FileNotFoundError` → `error="file_not_found"`

**Note:** Shell-based file operations (read, write, edit, ls, glob, grep) are inherited from `BaseSandbox` and run through `execute()`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `modal` | `modal` | Modal SDK — `Sandbox` type |
| `BaseSandbox` | `deepagents.backends.sandbox` | Base class with shell-based file operations |
| Protocol types | `deepagents.backends.protocol` | Response types |
| `contextlib.suppress` | stdlib | Suppressing close errors |
