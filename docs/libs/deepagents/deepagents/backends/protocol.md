# `deepagents/backends/protocol.py`

## High-Level Purpose

Defines the contract that all backend implementations must satisfy. This module contains:
- The abstract base class `BackendProtocol` with sync and async versions of every file operation
- The `SandboxBackendProtocol` extension that adds shell command execution
- Structured result dataclasses for all operations
- Supporting TypedDicts and type aliases

All backends (state, filesystem, store, composite, sandbox) implement this protocol. Middleware and tools interact with backends exclusively through these interfaces.

## Type Aliases and Constants

### `FileFormat`
`Literal["v1", "v2"]` — Selects the file content storage format. `v1` stores content as `list[str]` (legacy, deprecated). `v2` stores content as a plain `str` with an `encoding` field.

### `FileOperationError`
`Literal["file_not_found", "permission_denied", "is_directory", "invalid_path"]` — Standardized error codes for upload/download operations that LLMs can act on.

### `BackendFactory`
`TypeAlias = Callable[[ToolRuntime], BackendProtocol]` — A factory function that creates a backend from a `ToolRuntime`. Used when passing `StateBackend` as a class reference (factory pattern) to deferred instantiation at tool call time.

## TypedDicts

### `FileInfo`
Structured file metadata returned by `ls` and `glob`.

| Field | Type | Required | Description |
|---|---|---|---|
| `path` | `str` | Yes | Absolute file path |
| `is_dir` | `bool` | No | Whether the entry is a directory |
| `size` | `int` | No | File size in bytes |
| `modified_at` | `str` | No | ISO 8601 timestamp of last modification |

### `GrepMatch`
A single grep result entry.

| Field | Type | Description |
|---|---|---|
| `path` | `str` | Absolute path to the file containing the match |
| `line` | `int` | 1-indexed line number of the match |
| `text` | `str` | The matching line's text content |

### `FileData`
Internal file content representation used for storage.

| Field | Type | Description |
|---|---|---|
| `content` | `str` | UTF-8 text or base64-encoded binary |
| `encoding` | `str` | `"utf-8"` or `"base64"` |
| `created_at` | `str` | ISO 8601 creation timestamp |
| `modified_at` | `str` | ISO 8601 modification timestamp |

## Dataclasses

### `ReadResult`
Result of `read()` / `aread()`.
- `error: str | None` — error message on failure
- `file_data: FileData | None` — file content on success

### `WriteResult`
Result of `write()` / `awrite()`.
- `error: str | None` — error message on failure
- `path: str | None` — absolute path of written file on success
- `files_update: dict[str, Any] | None` — LangGraph state update dict for checkpoint backends (`StateBackend`); `None` for external backends (already persisted)

### `EditResult`
Result of `edit()` / `aedit()`.
- `error: str | None`
- `path: str | None`
- `files_update: dict[str, Any] | None`
- `occurrences: int | None` — number of replacements made

### `LsResult`
Result of `ls()` / `als()`.
- `error: str | None`
- `entries: list[FileInfo] | None`

### `GrepResult`
Result of `grep()` / `agrep()`.
- `error: str | None`
- `matches: list[GrepMatch] | None`

### `GlobResult`
Result of `glob()` / `aglob()`.
- `error: str | None`
- `matches: list[FileInfo] | None`

### `FileDownloadResponse`
Result of a single file download in a batch.
- `path: str` — requested path (for correlation)
- `content: bytes | None` — file bytes on success
- `error: FileOperationError | None`

### `FileUploadResponse`
Result of a single file upload in a batch.
- `path: str`
- `error: FileOperationError | None`

### `ExecuteResponse`
Result of `execute()` / `aexecute()` (sandbox only).
- `output: str` — combined stdout and stderr
- `exit_code: int | None` — process exit code
- `truncated: bool` — whether output was truncated

## Classes

### `BackendProtocol` (abstract base)

The unified protocol that all file-storage backends must implement. Concrete methods raise `NotImplementedError` unless a deprecated alias is detected. Async variants default to `asyncio.to_thread` wrappers around their sync counterparts.

**Methods:**

| Method | Signature | Description |
|---|---|---|
| `ls` | `(path: str) -> LsResult` | List directory contents (non-recursive) |
| `als` | `(path: str) -> LsResult` | Async ls |
| `read` | `(file_path, offset=0, limit=2000) -> ReadResult` | Read file lines with optional pagination |
| `aread` | async | Async read |
| `grep` | `(pattern, path=None, glob=None) -> GrepResult` | Literal text search (not regex) |
| `agrep` | async | Async grep |
| `glob` | `(pattern, path="/") -> GlobResult` | File path pattern matching |
| `aglob` | async | Async glob |
| `write` | `(file_path, content) -> WriteResult` | Create new file (fails if exists) |
| `awrite` | async | Async write |
| `edit` | `(file_path, old_string, new_string, replace_all=False) -> EditResult` | Replace exact string in file |
| `aedit` | async | Async edit |
| `upload_files` | `(files: list[tuple[str, bytes]]) -> list[FileUploadResponse]` | Batch file upload |
| `aupload_files` | async | Async batch upload |
| `download_files` | `(paths: list[str]) -> list[FileDownloadResponse]` | Batch file download |
| `adownload_files` | async | Async batch download |

**Deprecated aliases:** `ls_info` → `ls`, `glob_info` → `glob`, `grep_raw` → `grep` (and their async variants). These emit `DeprecationWarning` and delegate to the current method names.

---

### `SandboxBackendProtocol(BackendProtocol)`

Extension of `BackendProtocol` for backends that support shell command execution. Adds two abstract members:

- **`id: str` (property)** — Unique identifier for this backend instance.
- **`execute(command, *, timeout=None) -> ExecuteResponse`** — Run a shell command. `timeout` is in seconds.
- **`aexecute`** — Async version; defaults to `asyncio.to_thread(self.execute, ...)`.

## Functions

### `execute_accepts_timeout(cls) -> bool`

`@lru_cache`-decorated function. Inspects a backend class's `execute` method signature to check if it accepts a `timeout` keyword argument. Used by `CompositeBackend` and the execute tool to safely forward `timeout` without breaking older backend implementations.

**Parameters:**
- `cls: type[SandboxBackendProtocol]`

**Returns:** `True` if `execute` accepts `timeout`, `False` otherwise.
