# `deepagents/backends/langsmith.py`

## High-Level Purpose

`LangSmithSandbox` wraps a LangSmith `Sandbox` object (from the `langsmith[sandbox]` extra) to provide a full `SandboxBackendProtocol` implementation. It inherits all file operation methods (ls, read, edit, grep, glob) from `BaseSandbox` and overrides `execute()`, `write()`, `upload_files()`, and `download_files()` with LangSmith-native API calls.

## Key Characteristics

- **Inherits `BaseSandbox`:** All file ops not overridden here are implemented by `BaseSandbox` via shell command execution.
- **`write()` override:** Uses the LangSmith SDK's native `sandbox.write()` instead of `BaseSandbox`'s shell-based template, avoiding `ARG_MAX` limits on large file content by sending data in the HTTP body.
- **Partial-success batch operations:** `upload_files` and `download_files` catch per-file errors and report them in `FileUploadResponse`/`FileDownloadResponse` rather than raising.
- **Default timeout:** 30 minutes (`self._default_timeout = 30 * 60`).

## Dependencies

- `deepagents.backends.sandbox.BaseSandbox`
- `deepagents.backends.protocol` — result types
- `langsmith.sandbox.Sandbox` (TYPE_CHECKING only; imported lazily)

## Class: `LangSmithSandbox(BaseSandbox)`

### Constructor

```python
LangSmithSandbox(sandbox: Sandbox)
```

**Parameters:**
- `sandbox` — A `langsmith.sandbox.Sandbox` instance to wrap.

**Key Attributes:**
- `self._sandbox` — The underlying LangSmith `Sandbox` object.
- `self._default_timeout: int = 1800` — Default execution timeout in seconds.

### Properties

#### `id -> str`
Returns `self._sandbox.name` — the LangSmith sandbox name.

### Methods

#### `execute(command: str, *, timeout: int | None = None) -> ExecuteResponse`
Runs a command via `self._sandbox.run(command, timeout=effective_timeout)`. Combines stdout and stderr (with a newline separator) into a single `output` string. Always sets `truncated=False`.

**Parameters:**
- `command` — Shell command string.
- `timeout` — If `None`, uses `self._default_timeout` (30 min). A value of `0` may disable timeout if the SDK supports it.

#### `write(file_path: str, content: str) -> WriteResult`
Overrides `BaseSandbox.write()` to use `self._sandbox.write(file_path, content.encode("utf-8"))` instead of the shell heredoc approach. This avoids `ARG_MAX` limits for files larger than ~100 KB. Catches `SandboxClientError` and returns a `WriteResult(error=...)` on failure.

#### `download_files(paths: list[str]) -> list[FileDownloadResponse]`
Downloads each path individually via `self._sandbox.read(path)`. Validates that paths start with `/`. Maps `ResourceNotFoundError` to `"file_not_found"` and directory errors to `"is_directory"`.

#### `upload_files(files: list[tuple[str, bytes]]) -> list[FileUploadResponse]`
Uploads each file individually via `self._sandbox.write(path, content)`. Validates that paths start with `/`. Maps `SandboxClientError` to `"permission_denied"`.
