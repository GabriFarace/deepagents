# `deepagents/backends/state.py`

## High-Level Purpose

`StateBackend` implements `BackendProtocol` by storing file contents directly inside LangGraph's agent state dictionary. Files are ephemeral — they persist across turns within a single conversation thread (via checkpointing) but do not survive across different thread IDs.

This is the **default backend** used by `create_deep_agent` when no `backend` is provided.

## Key Characteristics

- **Ephemeral within thread:** Files live in `state["files"]` and are checkpointed with the rest of the agent state.
- **No cross-thread persistence:** Each thread starts with an empty filesystem unless you pre-populate `files` in the `invoke` call.
- **Read-your-writes:** Within a single superstep, a tool that writes a file and then reads it back will see its own write. This is implemented by flushing pending writes before each read.
- **LangGraph state updates:** Write and edit operations return `files_update` dicts. The middleware layer merges these into LangGraph state via `Command` objects.
- **`upload_files` not supported:** Raises `NotImplementedError`. Pre-populate files via `invoke(input={"files": {...}})`.

## Dependencies

- `deepagents.backends.protocol` — `BackendProtocol`, result types
- `deepagents.backends.utils` — file manipulation helpers
- LangGraph config accessors (`CONFIG_KEY_READ` with `fresh=True`, `CONFIG_KEY_SEND`)

## Class: `StateBackend(BackendProtocol)`

### Constructor

```python
StateBackend(
    runtime: object = None,
    *,
    file_format: FileFormat = "v2"
)
```

**Parameters:**
- `runtime` — **Deprecated** (0.5.0–0.7.0). State is now read via `get_config()` / `CONFIG_KEY_READ` rather than `runtime.state`. Passing a `runtime` object emits a `DeprecationWarning`.
- `file_format` — Storage format:
  - `"v2"` (default): Content stored as plain `str` with an `encoding` field.
  - `"v1"` (legacy): Content stored as `list[str]` (lines split on `\n`), no `encoding` field.

**Key Attributes:**
- `self._file_format` — Controls write/edit serialization format.

### Read-Your-Writes

`_read_files()` uses `CONFIG_KEY_READ` with `fresh=True`, which applies any pending task writes (from `_send_files_update()`) before returning the file map. This provides single-superstep read-your-writes semantics:

```
tool_call: write_file("/foo.py", "...")   # enqueues update
tool_call: read_file("/foo.py")           # sees the write above
```

`_send_files_update(update)` uses `CONFIG_KEY_SEND` to enqueue a partial `files` update directly. The update is visible to subsequent `_read_files()` calls within the same superstep.

### Methods

#### `ls(path: str) -> LsResult`
Lists files and directories directly in `path` (non-recursive). Discovers virtual subdirectories by scanning file path prefixes. Files directly in the target directory are returned as `FileInfo` entries; nested files cause their immediate parent directory to be reported as a directory entry.

#### `read(file_path, offset=0, limit=2000) -> ReadResult`
Reads a file from state (with read-your-writes applied). Non-text files are returned as-is. Text files are sliced to the `[offset, offset+limit)` line range via `slice_read_response`.

#### `write(file_path, content) -> WriteResult`
Creates a new file entry. Returns an error if the file already exists. Returns `WriteResult(path=..., files_update={file_path: data})` for the middleware layer to merge into LangGraph state.

#### `edit(file_path, old_string, new_string, replace_all=False) -> EditResult`
Replaces occurrences of `old_string` in the file's content. Uses `perform_string_replacement` which enforces uniqueness unless `replace_all=True`. Returns `files_update` with the updated content.

#### `grep(pattern, path=None, glob=None) -> GrepResult`
Literal substring search across all state files. Uses `grep_matches_from_files`.

#### `glob(pattern, path="/") -> GlobResult`
Pattern-matches file paths in state. Uses `_glob_search_files`.

#### `upload_files(files) -> list[FileUploadResponse]`
Raises `NotImplementedError`. Not supported for `StateBackend`.

#### `download_files(paths) -> list[FileDownloadResponse]`
Downloads files from state. Decodes content from the stored encoding (`utf-8` or `base64`).

### Private Methods

#### `_read_files() -> dict`
Returns the current file map with pending writes flushed (`fresh=True`).

#### `_send_files_update(update: dict) -> None`
Enqueues a partial `files` update for the current superstep.

#### `_prepare_for_storage(file_data: FileData) -> dict`
Converts a `FileData` dict to the configured storage format (`v1` or `v2`).
