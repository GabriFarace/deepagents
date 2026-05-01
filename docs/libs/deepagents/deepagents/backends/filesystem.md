# `deepagents/backends/filesystem.py`

## High-Level Purpose

`FilesystemBackend` implements `BackendProtocol` by reading and writing files directly from the real filesystem. It is intended for local development tools (coding assistants, CLIs) where the agent needs persistent file access across sessions.

> **Security Warning:** This backend grants direct filesystem read/write access. Do not use it in web servers or multi-tenant environments. Use `StateBackend`, `StoreBackend`, or a `SandboxBackend` instead for production workloads.

## Key Characteristics

- **Persistent:** Changes survive agent restarts and thread boundaries.
- **Two path modes:**
  - `virtual_mode=False` (default, deprecated): Absolute paths are used as-is; relative paths resolve against `root_dir`. No path restrictions.
  - `virtual_mode=True` (recommended): All paths are treated as virtual paths anchored to `root_dir`. Path traversal (`..`, `~`) and absolute paths outside `root_dir` are blocked. Primarily designed for use with `CompositeBackend`.
- **Symlink loop hardening:** Detects and raises on symlink loops, restoring pre-Python-3.13 behavior where `Path.resolve()` silently swallowed loops.
- **Grep implementation:** Tries `ripgrep` first for performance; falls back to Python `re` search.
- **`files_update=None`:** Write/edit operations return `files_update=None` since data is already persisted to disk.

## Dependencies

- `pathlib.Path`, `os`, `subprocess`, `re`, `base64`, `errno` — standard library
- `wcmatch.glob` — extended glob matching
- `deepagents.backends.protocol` — result types
- `deepagents.backends.utils` — `perform_string_replacement`, `create_file_data`, `_get_file_type`

## Class: `FilesystemBackend(BackendProtocol)`

### Constructor

```python
FilesystemBackend(
    root_dir: str | Path | None = None,
    virtual_mode: bool | None = None,
    max_file_size_mb: int = 10
)
```

**Parameters:**
- `root_dir` — Working directory. Defaults to `Path.cwd()`. In `virtual_mode=True`, acts as the virtual root. In `virtual_mode=False`, only affects relative path resolution.
- `virtual_mode` — When `None`, emits a `DeprecationWarning` and defaults to `False`. Default changes to `True` in v0.6.0. Explicit `False` suppresses the warning.
- `max_file_size_mb` — Maximum file size (in MB) for the Python fallback grep. Files larger than this are skipped. Default: 10 MB.

**Key Attributes:**
- `self.cwd: Path` — Resolved root directory.
- `self.virtual_mode: bool`
- `self.max_file_size_bytes: int`

### Private Methods

#### `_resolve_path(key: str) -> Path`
Resolves a path string to an absolute `Path` object.
- In `virtual_mode=True`: Treats `key` as a virtual path under `self.cwd`. Blocks `..` and `~`. Raises `ValueError` if the resolved path escapes `root_dir`. Calls `_raise_if_symlink_loop()` on the resolved path.
- In `virtual_mode=False`: Returns absolute paths as-is; relative paths resolve under `cwd`. Calls `_raise_if_symlink_loop()` on both the raw and resolved paths.

#### `_raise_if_symlink_loop(path: Path) -> None`
New in this version. Restores pre-Python-3.13 behavior where `Path.resolve()` raised on symlink loops.

- Python 3.13+ silently returns unresolved paths for symlink loops. This function probes with `stat()` (which follows symlinks) and re-raises if `errno.ELOOP` or Windows `winerror=1921` is detected.
- Called throughout `ls()`, `read()`, `edit()`, `glob()`, `download_files()`, and `_resolve_path()` to catch loops early.

#### `_is_symlink_loop_error(exc: Exception) -> bool`
Detects both `OSError(errno.ELOOP)` and Python ≤3.12's `RuntimeError` that wraps a loop from `Path.resolve()`.

#### `_is_eloop_oserror(exc: BaseException | None) -> bool`
Returns `True` for `OSError` with `errno == ELOOP` or Windows `winerror == 1921`.

#### `_to_virtual_path(path: Path) -> str`
Converts a filesystem path to a virtual path string relative to `cwd` (e.g., `/subdir/file.txt`).

### Public Methods

#### `ls(path: str) -> LsResult`
Lists direct children of a directory. In `virtual_mode=False`, returns absolute paths. In `virtual_mode=True`, returns virtual paths. Results are sorted by path. Silently skips entries where `stat()` fails. Raises on symlink loops.

#### `read(file_path, offset=0, limit=2000) -> ReadResult`
Reads a file using `O_NOFOLLOW` to avoid following symlinks at the final component. Non-text files (by extension) are base64-encoded. Text files are sliced to the `[offset, offset+limit)` line range.

#### `write(file_path, content) -> WriteResult`
Creates a new file with the given content. Creates parent directories as needed. Returns an error if the file already exists. Uses `O_NOFOLLOW` for write security.

#### `edit(file_path, old_string, new_string, replace_all=False) -> EditResult`
Reads, performs string replacement, and rewrites the file atomically. Uses `perform_string_replacement` which validates occurrence count.

#### `grep(pattern, path=None, glob=None) -> GrepResult`
Literal text search. Tries `ripgrep` (`rg -F`) first; falls back to `_python_search` if `rg` is unavailable or times out.

#### `_ripgrep_search(pattern, base_full, include_glob) -> dict | None`
Runs `rg --json -F` with a 30-second timeout. Returns structured matches dict, or `None` on failure.

#### `_python_search(pattern, base_full, include_glob) -> dict`
Pure Python fallback grep using `re.compile(re.escape(pattern))`. Skips files larger than `max_file_size_bytes`. Supports `wcmatch` glob filtering.

#### `glob(pattern, path="/") -> GlobResult`
Uses `Path.rglob()` for recursive matching. In `virtual_mode=True`, returns virtual paths and filters out results outside `root_dir`. Checks for symlink loops during traversal.

#### `upload_files(files) -> list[FileUploadResponse]`
Batch-writes binary files to disk. Maps common exceptions to standardized `FileOperationError` codes.

#### `download_files(paths) -> list[FileDownloadResponse]`
Batch-reads binary file contents. Checks for symlink loops. Maps exceptions to `FileOperationError` codes.
