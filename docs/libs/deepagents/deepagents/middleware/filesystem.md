# `deepagents/middleware/filesystem.py`

## High-Level Purpose

`FilesystemMiddleware` is the most important middleware. It adds the six built-in file and shell tools to the agent's tool list: `ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`, and (when the backend supports it) `execute`. It also enforces filesystem permission rules, ensuring tools can only access allowed paths and perform allowed operations.

---

## Key Class

### `FilesystemMiddleware(AgentMiddleware)`

**Constructor parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `backend` | `BackendProtocol` | required | The storage backend |
| `permissions` | `list[FilesystemPermission]` | `[]` | Access control rules |
| `custom_tool_descriptions` | `dict[str, str]` | `{}` | Override tool descriptions |
| `max_read_lines` | `int` | `2000` | Truncate reads at this many lines |

---

## Built-in Tools

### `ls(path: str)`

Lists directory contents. Returns entries with name, type (file/dir), size, and modification time. Formatted as a simple table.

### `read_file(path: str, start_line: int = None, end_line: int = None)`

Reads a file. Output is formatted as `cat -n` (line numbers prefix). Binary files are returned as base64. Truncated to `max_read_lines` with a "file truncated" note.

### `write_file(path: str, content: str)`

Writes or overwrites a file. Creates parent directories if needed. Shows a diff of the change in the tool result.

### `edit_file(path: str, old_string: str, new_string: str, replace_all: bool = False)`

Performs an exact string replacement. Fails with an error if `old_string` is not found (or not unique, unless `replace_all=True`). Returns a diff of the change.

### `glob(pattern: str)`

Finds files matching a glob pattern. Supports `*`, `**`, `?`, `[abc]`. Returns a list of matching paths.

### `grep(path: str, pattern: str, recursive: bool = False)`

Searches for a literal string (or regex, controlled by call site) in file contents. Returns matching lines with file path and line number. Not a full regex grep by default — pattern is treated as a literal string for safety.

### `execute(command: str, env: dict = None)`

Runs a shell command. Only added to the tool list if the backend implements `SandboxBackendProtocol`. Returns `stdout`, `stderr`, and `exit_code`. The agent is instructed to check `exit_code` and re-read files after writes.

---

## Permission System

`FilesystemPermission` rules are checked at tool-call time before the backend is called.

**Rule structure:**
```python
FilesystemPermission(
    operations=["read", "write"],   # or ["execute"], ["*"]
    paths=["/workspace/**"],         # wcmatch glob patterns
    mode="allow"                     # or "deny"
)
```

**Matching:**
- Rules are checked in order; first match wins
- Operations: `"read"`, `"write"`, `"edit"`, `"execute"`, `"ls"`, `"glob"`, `"grep"`, `"*"` (all)
- Path patterns use wcmatch globbing (`**` = any depth)
- Default (no rules): allow all

**Example — read-only access:**
```python
permissions=[
    FilesystemPermission(operations=["write", "execute"], paths=["/**"], mode="deny"),
]
```

---

## Architecture Notes

**Tool description customization:** The `custom_tool_descriptions` dict maps tool name to a custom description string. This lets you tune what the agent is told about the tool without subclassing.

**`execute` conditionality:** The tool is only added if `isinstance(backend, SandboxBackendProtocol)`. This means using `FilesystemBackend` (no shell) automatically means no `execute` tool, with no code changes needed.

**`edit_file` exactness:** The old-string requirement for `edit_file` is intentional — it forces the agent to read the file first and use the actual content, preventing blind overwrites.

---

## See Also

- [README.md](README.md) — middleware stack overview
- [../backends/protocol.md](../backends/protocol.md) — BackendProtocol called by these tools
- [../backends/README.md](../backends/README.md) — which backend provides `execute`
