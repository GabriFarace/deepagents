# `deepagents/backends/protocol.py`

## High-Level Purpose

`protocol.py` defines `BackendProtocol` — the abstract interface that every backend must implement. It also defines all result TypedDicts (`ReadResult`, `WriteResult`, etc.), the `FilesystemPermission` dataclass, and the file data format. This is the contract between the middleware layer (which uses backends) and the backend implementations (which provide storage and execution).

---

## Key Types

### `BackendProtocol`

Abstract base class (uses `Protocol` typing). All methods are async.

| Method | Signature | Description |
|---|---|---|
| `ls` | `(path: str) → LsResult` | List directory |
| `read` | `(path: str) → ReadResult` | Read file content |
| `write` | `(path: str, data: FileData) → WriteResult` | Write or overwrite file |
| `edit` | `(path: str, old: str, new: str, replace_all=False) → EditResult` | Replace text in file |
| `glob` | `(pattern: str) → GlobResult` | Find matching files |
| `grep` | `(path: str, pattern: str, recursive=False) → GrepResult` | Search file contents |
| `upload_files` | `(files: dict[str, FileData]) → FileUploadResponse` | Bulk write |
| `download_files` | `(paths: list[str]) → FileDownloadResponse` | Bulk read |

### `SandboxBackendProtocol`

Extends `BackendProtocol` with:

| Method | Signature | Description |
|---|---|---|
| `execute` | `(command: str, env: dict \| None) → ExecuteResult` | Run a shell command |

### `FileData` (TypedDict)

```python
class FileData(TypedDict):
    content: str                        # file content (string or base64)
    encoding: Literal["utf-8", "base64"]
    created_at: str                     # ISO 8601
    modified_at: str                    # ISO 8601
```

### `FilesystemPermission` (dataclass)

```python
@dataclass
class FilesystemPermission:
    operations: list[str]   # e.g., ["read", "write", "execute"]
    paths: list[str]        # wcmatch glob patterns, e.g., ["/workspace/**"]
    mode: Literal["allow", "deny"]
```

First matching rule wins. Default (no rules): allow all.

### Result TypedDicts

| Type | Key fields |
|---|---|
| `ReadResult` | `content: str`, `encoding: str`, `path: str` |
| `WriteResult` | `path: str`, `bytes_written: int` |
| `EditResult` | `path: str`, `replacements_made: int`, `diff: str` |
| `LsResult` | `entries: list[FileInfo]` |
| `GrepResult` | `matches: list[GrepMatch]` |
| `GlobResult` | `paths: list[str]` |
| `ExecuteResult` | `stdout: str`, `stderr: str`, `exit_code: int` |

### `FileInfo` (TypedDict)

```python
class FileInfo(TypedDict):
    name: str
    path: str
    is_dir: bool
    size: int | None
    modified_at: str | None
```

### `GrepMatch` (TypedDict)

```python
class GrepMatch(TypedDict):
    path: str
    line_number: int
    line: str
```

---

## Path Conventions

- All paths must be absolute (`/`-prefixed POSIX paths)
- No `..` or `~` in paths
- Paths are stored and returned as POSIX strings
- `glob` patterns support: `*` (any chars in segment), `**` (any path depth), `?` (single char), `[abc]` (character class)

---

## Legacy (v1) Format

The v1 format (used before 0.5.0) stored file content as `list[str]` (lines). v1 is still accepted by backends with a deprecation warning and auto-converted to v2.

---

## See Also

- [README.md](README.md) — choosing a backend
- [middleware/filesystem.md](../middleware/filesystem.md) — uses BackendProtocol for all file tools
