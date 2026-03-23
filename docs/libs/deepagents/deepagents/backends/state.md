# `deepagents/backends/state.py`

## High-Level Purpose

`StateBackend` implements `BackendProtocol` by storing file contents directly inside LangGraph's agent state dictionary. Files are ephemeral — they persist across turns within a single conversation thread (via checkpointing) but do not survive across different thread IDs.

This is the **default backend** used by `create_deep_agent` when no `backend` is provided.

## Key Characteristics

- **Ephemeral within thread:** Files live in `state["files"]` and are checkpointed with the rest of the agent state.
- **No cross-thread persistence:** Each thread starts with an empty filesystem unless you pre-populate `files` in the `invoke` call.
- **LangGraph state updates:** Write and edit operations return `files_update` dicts instead of `None`. The middleware layer handles merging these into LangGraph state via `Command` objects.
- **`upload_files` not supported:** Raises `NotImplementedError`. Files must be pre-populated via `invoke(input={"files": {...}})`.

## Dependencies

- `deepagents.backends.protocol` — `BackendProtocol`, result types
- `deepagents.backends.utils` — file manipulation helpers
- `langchain.tools.ToolRuntime` — provides access to `runtime.state`

## Class: `StateBackend(BackendProtocol)`

### Constructor

```python
StateBackend(runtime: ToolRuntime, *, file_format: FileFormat = "v2")
```

**Parameters:**
- `runtime` — `ToolRuntime` instance. Access to `runtime.state` and `runtime.store`.
- `file_format` — Storage format. `"v2"` (default) stores content as plain string with `encoding` field. `"v1"` uses the legacy `list[str]` format.

**Key Attributes:**
- `self.runtime` — The tool runtime for state access.
- `self._file_format` — Controls write/edit serialization format.

### Methods

All methods access `self.runtime.state.get("files", {})` to read the in-memory file store.

#### `ls(path: str) -> LsResult`
Lists files and directories directly in `path` (non-recursive). Discovers virtual subdirectories by scanning file path prefixes. Files directly in the target directory are returned as `FileInfo` entries; nested files cause their immediate parent directory to be reported as a directory entry.

#### `read(file_path, offset=0, limit=2000) -> ReadResult`
Reads a file from state. Non-text files (by extension) are returned as-is. Text files are sliced to the `[offset, offset+limit)` line range using `slice_read_response`.

#### `write(file_path, content) -> WriteResult`
Creates a new file entry. Returns an error if the file already exists. Returns `WriteResult(path=..., files_update={file_path: data})` which the middleware layer merges into LangGraph state.

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

#### `_prepare_for_storage(file_data: FileData) -> dict`
Converts a `FileData` dict to the configured storage format (`v1` or `v2`).
