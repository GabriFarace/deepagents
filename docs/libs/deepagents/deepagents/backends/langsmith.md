# `deepagents/backends/langsmith.py`

## High-Level Purpose

`LangSmithSandbox` wraps a LangSmith `Sandbox` object (from the `langsmith[sandbox]` extra) to provide a full `SandboxBackendProtocol` implementation. It uses **native SDK methods** for both reads and writes — avoiding shell-command transport limits and potential hangs — and adds a write preflight check.

## Key Characteristics

- **Native read and write:** Both `read()` and `write()` use the LangSmith SDK's HTTP API directly, not piped shell commands. This avoids `ARG_MAX` limits for large files and prevents hangs on large reads.
- **Write preflight:** Before writing, checks that the path doesn't already exist and that parent directories are creatable.
- **Partial-success batch operations:** `upload_files` and `download_files` catch per-file errors and report them in response objects rather than raising.
- **Default timeout:** 30 minutes (`self._default_timeout = 1800`).

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
- `self._default_timeout: int = 1800` — Default execution timeout (30 minutes).

### Properties

#### `id -> str`
Returns `self._sandbox.name` — the LangSmith sandbox name.

### Methods

#### `execute(command: str, *, timeout: int | None = None) -> ExecuteResponse`
Runs a shell command via `self._sandbox.run(command, timeout=effective_timeout)`. Combines stdout and stderr (with a newline separator) into a single `output` string. Always sets `truncated=False`.

**Parameters:**
- `command` — Shell command string.
- `timeout` — If `None`, uses `self._default_timeout` (30 min).

---

#### `read(file_path: str, offset: int = 0, limit: int = 2000) -> ReadResult`

**New in this version.** Uses the LangSmith SDK's native read API instead of piped shell commands.

**Behavior:**
- Routes by file extension (text vs binary).
- **Text files:** Normalizes line endings (`\r\n` and bare `\r` → `\n`, matching `open(..., newline=None)`). Applies `offset`/`limit` pagination locally on the normalized content.
- **Binary files (or non-UTF-8 text):** Base64-encodes content, capped at `MAX_BINARY_BYTES`.
- **Empty files:** Returns a system reminder message.
- **Large files:** Truncates at `MAX_OUTPUT_BYTES` and appends `TRUNCATION_MSG`.

---

#### `write(file_path: str, content: str) -> WriteResult`

Overrides `BaseSandbox.write()` to use the LangSmith SDK's native write API (`self._sandbox.write(file_path, content.encode("utf-8"))`).

**Write Preflight (`_write_preflight()`):**
1. Checks that the file doesn't already exist.
2. Creates parent directories if needed.
3. Returns a `WriteResult(error=...)` if checks fail; otherwise write proceeds normally.

**Why native write:** Avoids `ARG_MAX` limits that constrain shell heredoc approaches for files larger than ~100 KB.

---

#### `download_files(paths: list[str]) -> list[FileDownloadResponse]`
Downloads each path individually via the SDK `read()`. Validates paths start with `/`. Maps:
- `ResourceNotFoundError` → `"file_not_found"`
- Directory errors → `"is_directory"`

#### `upload_files(files: list[tuple[str, bytes]]) -> list[FileUploadResponse]`
Uploads each file individually via the SDK `write()`. Validates paths start with `/`. Maps `SandboxClientError` → `"permission_denied"`.
