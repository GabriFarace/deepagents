# `deepagents/backends/sandbox.py`

## High-Level Purpose

Provides `BaseSandbox`, an abstract base class that implements all `SandboxBackendProtocol` file operations (ls, read, write, edit, grep, glob) by generating Python scripts and executing them via the single abstract method `execute()`. Concrete sandbox backends only need to implement `execute()` to get a full, working backend.

This design allows any remote execution environment (Docker container, LangSmith sandbox, cloud VM, etc.) to become a complete backend simply by implementing how to run a shell command.

## Key Design Principles

- All file operations are translated into Python one-liners or heredoc scripts passed to `execute()`.
- To avoid shell injection and `ARG_MAX` limits on large file content, operations use **base64-encoded JSON payloads** passed via stdin through heredocs.
- The approach is universal: if you can execute a shell command, you can do file I/O.

## Module-Level Command Templates

These string constants define the Python scripts executed for each file operation. They use base64 encoding to safely pass arbitrary content without escaping issues.

| Template | Operation | Notes |
|---|---|---|
| `_GLOB_COMMAND_TEMPLATE` | `glob()` | Runs `glob.glob()` in a Python subprocess |
| `_WRITE_COMMAND_TEMPLATE` | `write()` | Checks file existence, creates parent dirs, writes file |
| `_EDIT_COMMAND_TEMPLATE` | `edit()` | Reads file, counts/validates occurrences, replaces, writes |
| `_READ_COMMAND_TEMPLATE` | `read()` | Reads file, handles encoding, slices lines |

The write/edit/read templates use heredocs (`<<'__DEEPAGENTS_EOF__'`) to pass `base64(json(payload))` via stdin, bypassing `ARG_MAX` and shell injection issues entirely.

## Class: `BaseSandbox(SandboxBackendProtocol, ABC)`

### Abstract Methods (subclasses must implement)

#### `execute(command: str, *, timeout: int | None = None) -> ExecuteResponse`
The only method subclasses **must** implement. Runs a shell command in the sandbox environment and returns an `ExecuteResponse`.

#### `id -> str` (abstract property)
Unique identifier for this sandbox instance.

#### `upload_files(files) -> list[FileUploadResponse]`
Abstract. Subclasses must implement batch file upload with per-file error handling (partial success, no exceptions).

#### `download_files(paths) -> list[FileDownloadResponse]`
Abstract. Subclasses must implement batch file download with per-file error handling.

### Implemented File Methods (via execute)

All these methods generate a command string and call `self.execute(cmd)`, then parse the output.

#### `ls(path: str) -> LsResult`
Runs `os.scandir()` via `python3 -c`, outputs JSON lines with `path` and `is_dir` fields.

#### `read(file_path, offset=0, limit=2000) -> ReadResult`
Uses `_READ_COMMAND_TEMPLATE`. Handles both text (line-sliced) and binary (base64-encoded) files. Returns `ReadResult` with `file_data`.

#### `write(file_path, content) -> WriteResult`
Uses `_WRITE_COMMAND_TEMPLATE`. Atomically checks existence and creates file.

#### `edit(file_path, old_string, new_string, replace_all=False) -> EditResult`
Uses `_EDIT_COMMAND_TEMPLATE`. Maps exit codes 1–4 to specific error messages (not found, multiple occurrences, file not found, decode error).

#### `grep(pattern, path=None, glob=None) -> GrepResult`
Runs `grep -rHnF` (literal, recursive, with filename and line numbers). Uses `shlex.quote` for safe argument passing.

#### `glob(pattern, path="/") -> GlobResult`
Uses `_GLOB_COMMAND_TEMPLATE`. Runs `glob.glob(..., recursive=True)` inside the sandbox.

## Dependencies

- `abc`, `base64`, `json`, `shlex` — standard library
- `deepagents.backends.protocol` — all result types and `SandboxBackendProtocol`
- `deepagents.backends.utils` — `_get_file_type`, `create_file_data`
