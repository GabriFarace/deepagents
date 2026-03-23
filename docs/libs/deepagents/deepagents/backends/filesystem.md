# `deepagents/backends/filesystem.py`

## High-Level Purpose

`FilesystemBackend` implements `BackendProtocol` by reading and writing files directly from the real filesystem. It is intended for local development tools (coding assistants, CLIs) where the agent needs persistent file access across sessions.

> **Security Warning:** This backend grants direct filesystem read/write access. Do not use it in web servers or multi-tenant environments. Use `StateBackend`, `StoreBackend`, or a `SandboxBackend` instead for production workloads.

## Key Characteristics

- **Persistent:** Changes survive agent restarts and thread boundaries.
- **Two path modes:**
  - `virtual_mode=False` (default): Absolute paths are used as-is; relative paths resolve against `root_dir`. No path restrictions.
  - `virtual_mode=True`: All paths are treated as virtual paths anchored to `root_dir`. Path traversal (`..`, `~`) and absolute paths outside `root_dir` are blocked. Primarily designed for use with `CompositeBackend`.
- **Grep implementation:** Tries `ripgrep` first for performance; falls back to Python `re` search.
- **`files_update=None`:** Write/edit operations set `files_update=None` since data is already persisted to disk.

## Dependencies

- `pathlib.Path`, `os`, `subprocess`, `re`, `base64` — standard library
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
- `virtual_mode` — When `None`, emits a `DeprecationWarning` and defaults to `False`. Will change default to `True` in v0.5.0.
- `max_file_size_mb` — Maximum file size (in MB) for the Python fallback grep search. Files larger than this are skipped. Default: 10 MB.

**Key Attributes:**
- `self.cwd: Path` — Resolved root directory.
- `self.virtual_mode: bool`
- `self.max_file_size_bytes: int`

### Private Methods

#### `_resolve_path(key: str) -> Path`
Resolves a path string to an absolute `Path` object.
- In `virtual_mode=True`: Treats `key` as a virtual path under `self.cwd`. Blocks `..` and `~`. Raises `ValueError` if the resolved path escapes `root_dir`.
- In `virtual_mode=False`: Returns absolute paths as-is; relative paths resolve under `cwd`.

#### `_to_virtual_path(path: Path) -> str`
Converts a filesystem path to a virtual path string relative to `cwd` (e.g., `/subdir/file.txt`).

### Public Methods

#### `ls(path: str) -> LsResult`
Lists direct children of a directory. In `virtual_mode=False`, returns absolute paths. In `virtual_mode=True`, returns virtual paths. Results are sorted by path. Silently skips entries where `stat()` fails.

#### `read(file_path, offset=0, limit=2000) -> ReadResult`
Reads a file using `O_NOFOLLOW` to avoid following symlinks. Non-text files (by extension) are base64-encoded and returned as `file_data`. Text files are sliced to the `[offset, offset+limit)` line range.

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
Uses `Path.rglob()` for recursive matching. In `virtual_mode=True`, returns virtual paths and filters out results outside `root_dir`.

#### `upload_files(files) -> list[FileUploadResponse]`
Batch-writes binary files to disk. Maps common exceptions to standardized `FileOperationError` codes.

#### `download_files(paths) -> list[FileDownloadResponse]`
Batch-reads binary file contents. Maps exceptions to `FileOperationError` codes.
