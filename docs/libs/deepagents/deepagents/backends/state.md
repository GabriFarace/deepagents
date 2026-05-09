# `libs/deepagents/deepagents/backends/state.py`

> Ephemeral virtual filesystem backend that stores file data in the LangGraph
> `files` state channel.

## Position in the system

`StateBackend` is the default low-friction backend for SDK agents. The
filesystem tools call backend methods, and this implementation reads and writes
the graph's `files` channel through LangGraph's current runtime config. Files
therefore persist within a checkpointed thread but do not become global storage
outside that graph state.

```
FilesystemMiddleware tool
  │
  └─ StateBackend
       ├─ CONFIG_KEY_READ("files", fresh=True)
       └─ CONFIG_KEY_SEND([("files", update)])
```

## Imports and module-level state

The module imports LangGraph Pregel config keys (`CONFIG_KEY_READ`,
`CONFIG_KEY_SEND`) and `get_config()` so a backend instance can be created once
but read the active graph state at call time. It imports protocol result types
and shared utility helpers for file-data conversion, slicing, globbing, grep,
and exact replacement. There is no mutable module-level state.

## Functions and classes

### `StateBackend(runtime=None, *, file_format="v2")`

`StateBackend` implements `BackendProtocol` against LangGraph state. The
deprecated `runtime` argument is ignored after emitting a warning; state access
now comes from `get_config()` at the moment each method runs. `file_format`
controls whether writes are stored as modern `FileData` (`content: str` plus
`encoding`) or legacy v1 dictionaries with `content: list[str]`.

The class mutates graph state only through `_send_files_update()`. Callers see
ordinary `ReadResult`, `WriteResult`, and `EditResult` objects; they do not have
to return a `files_update` patch from tool code.

#### `_get_config()`

This helper fetches the current LangGraph config and validates that Pregel's
state read/send hooks are present. If called outside graph execution, it raises
a clear `RuntimeError` explaining that files should be pre-populated through
the graph input, for example by passing `{"files": {...}}` on invoke.

#### `_read_files()`

`_read_files()` reads the `files` channel through `CONFIG_KEY_READ` with
`fresh=True`. That detail gives read-your-writes behavior inside one graph
superstep: a tool can write a file and a later tool call in the same execution
can read the pending update.

#### `_send_files_update(update)`

This queues a partial update to the `files` channel through `CONFIG_KEY_SEND`.
The channel reducer merges dictionaries, so the backend only sends changed
paths and leaves unrelated files intact. The update becomes committed at the
node boundary but is visible to fresh reads earlier.

#### `_prepare_for_storage(file_data)`

This converts modern `FileData` to the configured storage representation. In
`v2` mode it returns a shallow dict copy. In `v1` mode it delegates to
`_to_legacy_file_data()` for backwards compatibility.

#### `ls(path)`

`ls()` reads the full file mapping and returns only direct children of `path`.
It normalizes the requested directory to a trailing-slash prefix, collects file
entries directly under that prefix, and synthesizes directory entries for
deeper paths. Results are sorted by path for deterministic tool output.

It computes file size from either modern string content or legacy list content.
Directory entries have `is_dir=True`, size zero, and empty `modified_at`.

#### `read(file_path, offset=0, limit=2000)`

`read()` looks up a path in the state mapping. Missing paths return an error.
Non-text files, as classified by extension, are returned as full `FileData`;
text files are passed through `slice_read_response()` so the middleware gets
only the requested raw content window.

The method preserves `created_at` and `modified_at` metadata on sliced text
responses. Line-number formatting happens later in middleware, not here.

#### `write(file_path, content)`

`write()` creates a new file and refuses to overwrite an existing path. It
builds timestamped `FileData` with `create_file_data()`, converts it to the
configured storage format, sends a state update, and returns the created path.

#### `edit(file_path, old_string, new_string, replace_all=False)`

`edit()` loads the existing file, converts legacy or modern content to a plain
string, and delegates exact replacement validation to
`perform_string_replacement()`. If the replacement is ambiguous or missing, the
helper returns a user-facing error string.

On success, the backend updates content and `modified_at` while preserving
creation metadata, then queues the changed path through `_send_files_update()`.

#### `grep(pattern, path=None, glob=None)`

`grep()` runs a literal substring search over state files by delegating to
`grep_matches_from_files()`. It defaults the search path to `/` and returns a
structured `GrepResult`.

#### `glob(pattern, path="/")`

`glob()` delegates matching to `_glob_search_files()`, then converts the
newline-separated legacy helper output into structured `FileInfo` entries. It
returns an empty match list when the helper reports `"No files found"`.

#### `upload_files(files)`

`upload_files()` accepts byte payloads, decodes UTF-8 files as text, and stores
non-UTF-8 payloads as base64 text. Existing paths are updated rather than
rejected, matching bulk upload semantics rather than `write()`'s create-only
semantics. All pending writes are sent in one channel update.

#### `download_files(paths)`

`download_files()` returns one response per requested path. Missing files use
the stable `"file_not_found"` error code. Existing files are converted back to
bytes according to their `encoding`: UTF-8 strings are encoded directly, while
base64 strings are decoded.

## Flow walk-through

1. A filesystem tool calls `StateBackend.write()` or `edit()` (`state.py:247`,
   `state.py:265`).
2. The backend reads current `files` through `CONFIG_KEY_READ` (`state.py:121`).
3. It validates create/edit semantics and builds a `FileData` update.
4. `_send_files_update()` queues a partial `files` channel write
   (`state.py:126`).
5. Later reads in the same superstep use `fresh=True` and can observe the
   pending update (`state.py:123`).

## Gotchas

`StateBackend` cannot be called directly from arbitrary Python code unless a
LangGraph execution config is active. Use an explicit `files` graph input for
preload data, or choose `StoreBackend(store=...)` / `FilesystemBackend(...)`
for out-of-graph use.
