# `libs/deepagents/deepagents/backends/store.py`

> Persistent virtual filesystem backend backed by LangGraph `BaseStore`.

## Position in the system

`StoreBackend` is the cross-thread persistence option for SDK file tools. It
implements the same `BackendProtocol` surface as `StateBackend`, but stores
files as LangGraph store items under a namespace. This makes it suitable for
memories or durable artifacts that should survive beyond one graph thread.

## Imports and module-level state

The module imports `get_store()`, `get_runtime()`, and `get_config()` to locate
the active LangGraph store and runtime at call time. It uses shared backend
utilities for file-data creation, legacy conversion, slicing, grep, glob, and
string replacement. `_NAMESPACE_COMPONENT_RE` constrains namespace components
to safe characters so wildcard-like values cannot alter store search scope.

## Functions and classes

### `BackendContext`

`BackendContext` is a deprecated dataclass that used to wrap state and runtime
for namespace factories. New namespace factories receive a LangGraph `Runtime`
directly. The class remains only for backwards compatibility and warns through
the `@deprecated` decorator.

### `_NamespaceRuntimeCompat`

This private compatibility wrapper lets old factories access `.runtime` and
`.state` while new factories access runtime attributes directly. Deprecated
properties emit warnings, and `__getattr__()` forwards unknown attributes to
the wrapped runtime when it exists.

If the backend is called outside graph execution and no runtime is available,
runtime attribute access raises `AttributeError`. A namespace factory that does
not need runtime data can still work.

### `NamespaceFactory`

`NamespaceFactory` is a callable type alias for functions that accept a
`Runtime` and return a namespace tuple. Namespaces are the storage partition for
all store operations.

### `_validate_namespace(namespace)`

This helper enforces that namespace tuples are non-empty, contain only strings,
and use only alphanumeric plus selected safe punctuation characters. It rejects
empty components and wildcard-like punctuation to prevent accidental broad
searches or glob injection in store lookups.

### `StoreBackend(runtime=None, *, store=None, namespace=None, file_format="v2")`

The constructor accepts either an explicit `BaseStore` or defers store lookup
to LangGraph's execution context. The deprecated `runtime` argument is ignored
with a warning. `namespace` controls where files are stored; omitting it falls
back to deprecated assistant-id detection.

`file_format` controls whether values are written in modern v2 form
(`content`, `encoding`, timestamps) or legacy v1 form (`content: list[str]`,
timestamps only). The backend reads both.

#### `_get_store()`

This returns the explicit store if one was supplied. Otherwise it calls
`get_store()` and raises a clear `RuntimeError` if no graph execution context
provides a store.

#### `_get_namespace()`

This resolves the namespace for each operation. If a factory was configured,
it tries to fetch the active runtime, wraps it in `_NamespaceRuntimeCompat`,
calls the factory, and validates the tuple. Without a factory, it delegates to
the deprecated legacy path.

#### `_get_namespace_legacy()`

The legacy resolver warns, then tries to read `assistant_id` from LangGraph
config metadata. With an assistant ID it stores under
`(assistant_id, "filesystem")`; otherwise it uses `("filesystem",)`.

#### `_convert_store_item_to_file_data(store_item)`

This converts a LangGraph `Item` value into canonical `FileData`. It accepts
legacy `list[str]` content with a warning, accepts modern string content, and
copies string timestamp fields when present. Missing or invalid content raises
`ValueError` or `TypeError`.

#### `_convert_file_data_to_store_value(file_data)`

This is the inverse conversion for writes. In v1 mode it delegates to
`_to_legacy_file_data()`. In v2 mode it emits a dict containing content,
encoding, and optional timestamps.

#### `_search_store_paginated(store, namespace, *, query=None, filter=None, page_size=100)`

Store search is paginated because `BaseStore.search()` returns bounded pages.
This helper loops with `offset` until an empty or short page is returned,
collecting all matching items. Listing, grep, and glob use it to avoid
store-specific assumptions about filter semantics.

#### `ls(path)`

`ls()` loads all items in the namespace, filters locally by the requested path
prefix, returns direct file children, and synthesizes immediate subdirectory
entries. It skips items that cannot be converted to file data and sorts results
by path.

#### `read(file_path, offset=0, limit=2000)` and `aread(...)`

`read()` fetches one store item by key and converts it to `FileData`. Non-text
files are returned in full; text files are sliced by line with
`slice_read_response()`, preserving timestamps on the sliced response.

`aread()` is the same logic using `store.aget()` to avoid sync store calls in
async contexts.

#### `write(file_path, content)` and `awrite(...)`

`write()` and `awrite()` are create-only. They check whether the key already
exists, build timestamped `FileData`, convert it to a store value, and write it
with `put()` or `aput()`.

#### `edit(file_path, old_string, new_string, replace_all=False)` and `aedit(...)`

The edit methods fetch the existing item, convert content to a string, use
`perform_string_replacement()` for exact-match validation, update file data and
timestamps, then write the new value back to the store. Async and sync versions
only differ in `aget`/`aput` versus `get`/`put`.

#### `grep(pattern, path=None, glob=None)`

`grep()` reads all convertible store items into an in-memory mapping and
delegates literal matching to `grep_matches_from_files()`. Invalid store items
are skipped so one malformed item does not break search across the namespace.

#### `glob(pattern, path="/")`

`glob()` similarly reads all convertible files and delegates path matching to
`_glob_search_files()`. It converts matched paths back into structured
`FileInfo` entries with size and modification metadata when available.

#### `upload_files(files)`

Bulk upload decodes UTF-8 payloads as text and stores non-UTF-8 payloads as
base64. Unlike `write()`, upload overwrites existing keys. Responses preserve
input order.

#### `download_files(paths)`

Bulk download fetches each key, returns `"file_not_found"` for misses, converts
store values to `FileData`, and then emits bytes according to `encoding`.

## Gotchas

Always pass an explicit `namespace` for new code. The assistant-id fallback is
deprecated and couples persistence to graph metadata that may not exist in all
runtime environments.
