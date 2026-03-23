# `deepagents/backends/store.py`

## High-Level Purpose

`StoreBackend` implements `BackendProtocol` using LangGraph's `BaseStore` for persistent, cross-conversation file storage. Unlike `StateBackend` (ephemeral per-thread), files stored here persist across all conversation threads and agent restarts.

Primary use case: agent "memories" (AGENTS.md files), long-lived documents, or any data that should outlive a single conversation session.

## Key Characteristics

- **Persistent across threads:** Uses `store.put()` / `store.get()` with namespaced keys.
- **Namespace isolation:** Files are partitioned by namespace tuples (e.g., per user, per assistant). Namespaces are provided via a factory function.
- **Paginated store search:** `_search_store_paginated` handles backends that return results in pages.
- **`files_update=None`:** Write/edit return `WriteResult(files_update=None)` since data is already in the store.
- **Async native:** `aread`, `awrite`, `aedit` use native `store.aget` / `store.aput` to avoid sync calls in async contexts.

## Dependencies

- `langgraph.store.base.BaseStore`, `Item`
- `langgraph.config.get_config`
- `deepagents.backends.protocol` — result types
- `deepagents.backends.utils` — file manipulation helpers
- `langchain.tools.ToolRuntime`

## Supporting Types

### `BackendContext(Generic[StateT, ContextT])` (dataclass)
Context object passed to namespace factory functions.

**Fields:**
- `state: StateT` — Current agent state.
- `runtime: Runtime[ContextT]` — LangGraph runtime.

### `NamespaceFactory`
`TypeAlias = Callable[[BackendContext], tuple[str, ...]]`

A callable that receives a `BackendContext` and returns a namespace tuple. Example:
```python
lambda ctx: ("filesystem", ctx.runtime.context.user_id)
```

### `_validate_namespace(namespace: tuple[str, ...]) -> tuple[str, ...]`
Validates that every namespace component is a non-empty string containing only alphanumeric characters, hyphens, underscores, dots, `@`, `+`, colons, and tildes. Raises `ValueError` or `TypeError` on invalid input. This prevents wildcard injection (`*`, `?`, etc.) in store lookups.

## Class: `StoreBackend(BackendProtocol)`

### Constructor

```python
StoreBackend(
    runtime: ToolRuntime,
    *,
    namespace: NamespaceFactory | None = None,
    file_format: FileFormat = "v2",
)
```

**Parameters:**
- `runtime` — `ToolRuntime` providing access to `runtime.store`.
- `namespace` — Factory function returning the namespace tuple. If `None`, falls back to deprecated legacy detection (emits `DeprecationWarning`). Will be required in v0.5.0.
- `file_format` — Storage format (`"v1"` or `"v2"`).

### Private Methods

#### `_get_store() -> BaseStore`
Returns `runtime.store`. Raises `ValueError` if the store is unavailable.

#### `_get_namespace() -> tuple[str, ...]`
Calls the namespace factory if provided, otherwise calls `_get_namespace_legacy()`.

#### `_get_namespace_legacy() -> tuple[str, ...]`
Deprecated. Reads `assistant_id` from LangGraph config metadata. Falls back to `("filesystem",)`. Emits `DeprecationWarning`.

#### `_convert_store_item_to_file_data(store_item: Item) -> FileData`
Converts a LangGraph `Item` to a `FileData` dict. Handles legacy `list[str]` content with a `DeprecationWarning`. Validates `created_at` and `modified_at` fields.

#### `_convert_file_data_to_store_value(file_data: FileData) -> dict`
Converts `FileData` to a dict suitable for `store.put()`. Applies `file_format` conversion.

#### `_search_store_paginated(store, namespace, *, query=None, filter=None, page_size=100) -> list[Item]`
Fetches all items from a namespace via paginated `store.search()` calls.

### Public Methods

#### `ls(path: str) -> LsResult`
Retrieves all store items, filters by path prefix, and returns direct children (files and inferred subdirectories).

#### `read(file_path, offset=0, limit=2000) -> ReadResult` / `aread` (async)
Fetches item from store, converts to `FileData`, applies line slicing.

#### `write(file_path, content) -> WriteResult` / `awrite` (async)
Checks for existing item; returns error if exists. Creates new item via `store.put()`.

#### `edit(file_path, old_string, new_string, replace_all=False) -> EditResult` / `aedit` (async)
Fetches item, performs string replacement, puts updated item back.

#### `grep(pattern, path=None, glob=None) -> GrepResult`
Loads all items into a local dict and delegates to `grep_matches_from_files`.

#### `glob(pattern, path="/") -> GlobResult`
Loads all items into a local dict and delegates to `_glob_search_files`.

#### `upload_files(files) -> list[FileUploadResponse]`
Batch-uploads files. Binary files are base64-encoded. Calls `store.put()` per file.

#### `download_files(paths) -> list[FileDownloadResponse]`
Batch-downloads files. Decodes content from stored encoding.
