# `libs/deepagents/deepagents/backends/protocol.py`

> The contract every file/shell backend implements, plus the shared result
> objects used by filesystem middleware and sandbox integrations.

## Position in the system

`BackendProtocol` is the SDK's storage boundary. `create_deep_agent()` chooses
a backend, `FilesystemMiddleware` exposes tools to the model, and those tools
delegate to backend methods for listing, reading, searching, editing, uploading,
downloading, and possibly executing commands.

```
Filesystem tool
  │
  ├─ ls/read/write/edit/grep/glob
  │    └─ BackendProtocol
  │
  └─ execute
       └─ SandboxBackendProtocol
```

Backends can be state-backed, disk-backed, store-backed, composite, or provided
by partner packages. The protocol gives all of them the same result shapes, so
tool code can render consistent messages to the model.

## Imports and module-level state

The module uses `dataclass` and `TypedDict` for low-friction result objects,
`asyncio.to_thread()` for default async wrappers around sync implementations,
and `inspect.signature()` plus `lru_cache()` to handle older sandbox backends
that do not accept the newer `timeout=` keyword. It also imports the internal
deprecation helper because several legacy APIs are still bridged through this
file.

Module-level constants define the file storage version (`FileFormat`), stable
upload/download error codes (`FILE_NOT_FOUND`, `PERMISSION_DENIED`,
`IS_DIRECTORY`, `INVALID_PATH`), and backend type aliases.

## Functions and classes

### `FileOperationError`

`FileOperationError` is a literal union of common upload/download failures:
file not found, permission denied, directory requested as a file, and invalid
path. It is intentionally narrow and recoverable: these are errors the model or
caller can often fix by choosing a different path or operation.

The named constants exist so producers and consumers do not have to repeat bare
strings. If a code is renamed, type checkers and imports have a better chance of
catching the mismatch.

### `FileDownloadResponse`

`FileDownloadResponse` reports the result of one requested download. It always
carries the requested `path`, then either `content` bytes on success or an
`error` on failure. Batch downloads return one response per requested path in
the same order, allowing partial success.

The response is shaped for both programmatic callers and LLM-facing tools: a
caller can correlate results by index or path, and a model can read a stable
error code when it needs to recover.

### `FileUploadResponse`

`FileUploadResponse` mirrors `FileDownloadResponse` for uploads. It carries the
target `path` and an optional `error`; a missing error means the upload
succeeded. There is no content field because the caller already owns the bytes
it attempted to upload.

Like downloads, upload batches preserve input order and allow partial success.
This matters for custom tools that may expose bulk file transfer to an agent.

### `FileInfo`

`FileInfo` is the minimal listing shape shared by `ls()` and `glob()`.
`path` is required; `is_dir`, `size`, and `modified_at` are best-effort fields.
Remote stores and sandboxes may not be able to supply all metadata cheaply, so
the optional fields cannot be treated as always present.

### `GrepMatch`

`GrepMatch` represents one literal text-search hit. It stores the matched file
path, a one-indexed line number, and the matching line text. The protocol uses
literal substring matching rather than regex semantics, so middleware can expose
a simpler, safer search tool to the model.

### `FileData`

`FileData` is the canonical in-memory file representation. Current `v2` data
stores `content` as one string and `encoding` as either `"utf-8"` or
`"base64"`, with optional creation and modification timestamps.

The protocol still acknowledges legacy `v1` data where `content` was
`list[str]`. Concrete backends are responsible for accepting or migrating that
shape while warning callers that it is deprecated.

### `ReadResult`

`ReadResult` wraps backend reads in a consistent success/error envelope. On
success, `file_data` contains the canonical `FileData`; on failure, `error`
contains a message and `file_data` is absent.

The middleware can then decide how to render the file content to the model,
including line numbering, truncation, and artifacts, without each backend
inventing a different return type.

### `_Unset` and `Unset`

`_Unset` is a private sentinel used to distinguish "the caller did not pass
`files_update`" from "the caller explicitly passed `None`". That distinction is
only needed during the deprecation window for `files_update`.

### `_normalize_files_update(files_update)`

This helper backs the custom constructors for `WriteResult` and `EditResult`.
If the sentinel is present, it returns `None` with no warning. If the caller
explicitly supplies any `files_update` value, it emits a deprecation warning and
returns the value.

The warning's stack level is tuned so attribution points at the caller creating
the result object, not at the helper. This is a migration affordance for custom
backend authors.

### `WriteResult`

`WriteResult` reports the outcome of creating a file. `path` is set on success,
`error` is set on failure, and `files_update` remains only for deprecated
compatibility with older state-mutating backend implementations.

The constructor is custom rather than dataclass-generated so it can route
`files_update` through `_normalize_files_update()`. New backends should not use
`files_update`; state updates are handled internally by backend methods.

### `EditResult`

`EditResult` is the write result plus `occurrences`, the count of replacements
made by an exact-string edit. That count lets the middleware report whether a
single replacement or a replace-all operation changed the expected amount of
text.

Like `WriteResult`, it keeps deprecated `files_update` compatibility through a
custom constructor. A failed edit normally sets `error` and leaves `path` and
`occurrences` absent.

### `LsResult`, `GrepResult`, and `GlobResult`

These three dataclasses standardize list/search results. Each has an `error`
field and a payload field: `entries` for `ls`, `matches` for `grep`, and
`matches` for `glob`. The payload is `None` on failure and a list on success.

The explicit wrapper types are the replacement for older APIs that returned
either raw lists or error strings. They make tool code less branchy and give
backend implementations one shape to target.

### `BackendProtocol`

`BackendProtocol` defines the file-operation surface. It is an abstract base
class but its methods are not decorated with `@abstractmethod`, because older
third-party backends may implement only a subset. Missing operations fail at
call time with `NotImplementedError`.

The core methods are `ls`, `read`, `grep`, `glob`, `write`, `edit`,
`upload_files`, and `download_files`, each with a default async counterpart
that uses `asyncio.to_thread()`. The async wrappers mean a sync backend can
work in async agent runs, while high-performance async backends can override
the `a*` methods directly.

#### `ls(path)` and `als(path)`

`ls()` returns metadata for files under a directory. Its compatibility branch
detects whether a subclass implemented the deprecated `ls_info()` method; if
so, it warns and wraps that legacy list in `LsResult`. If neither the new nor
legacy method is implemented, it raises `NotImplementedError`.

`als()` runs `ls()` in a worker thread by default. Backends with native async
clients should override it to avoid blocking threads.

#### `read(file_path, offset=0, limit=2000)` and `aread(...)`

`read()` returns canonical file data for a path, optionally constrained by
line offset and limit. The protocol does not implement a legacy bridge here;
concrete backends must provide it.

The default `aread()` wrapper delegates to `read()` in a thread. Middleware is
responsible for presenting content with line numbers and truncation policy.

#### `grep(pattern, path=None, glob=None)` and `agrep(...)`

`grep()` searches for a literal substring, optionally within a directory and
optionally filtered by a filename glob. If a subclass still implements
`grep_raw()`, the protocol warns, calls it, and converts either an error string
or list of matches into `GrepResult`.

The method is intentionally not regex-based. That keeps the model-facing search
tool predictable across backends and reduces escaping mistakes.

#### `glob(pattern, path="/")` and `aglob(...)`

`glob()` finds files whose paths match a glob pattern relative to a base path.
It bridges deprecated `glob_info()` implementations in the same way `ls()`
bridges `ls_info()`: warn, call legacy method, wrap in `GlobResult`.

The async wrapper uses `asyncio.to_thread()`, matching the other sync-first
operations.

#### `write(file_path, content)` and `awrite(...)`

`write()` creates a new file with string content and returns a `WriteResult`.
The protocol does not enforce overwrite policy itself, but the docstring and
tool contract say this operation should error if the target file already
exists.

`awrite()` delegates to the sync method by default.

#### `edit(file_path, old_string, new_string, replace_all=False)` and `aedit(...)`

`edit()` performs exact string replacement. With `replace_all=False`, concrete
backends should fail if `old_string` is not unique; with `replace_all=True`,
they should replace every occurrence and report the count.

The exact-match contract is important for agent safety. It lets the model make
targeted edits and receive a failure instead of accidentally changing multiple
similar regions.

#### `upload_files(files)` and `aupload_files(files)`

`upload_files()` accepts a list of `(path, bytes)` tuples and returns one
`FileUploadResponse` per input. It is designed for direct developer use and for
custom tools that expose batch upload to an agent.

The base method raises `NotImplementedError`; concrete sandboxes or filesystem
backends decide how to write bytes and normalize per-file errors.

#### `download_files(paths)` and `adownload_files(paths)`

`download_files()` is the inverse of upload. It accepts a list of paths and
returns ordered `FileDownloadResponse` objects containing bytes or normalized
errors.

This API is separate from `read()` because it is byte-oriented and suited to
external file transfer, while `read()` is the text-oriented operation exposed
to the model.

#### Deprecated methods: `ls_info`, `als_info`, `glob_info`, `aglob_info`, `grep_raw`, `agrep_raw`

These methods preserve compatibility for older backend implementations while
nudging authors toward result-wrapper APIs. New code should implement
`ls`, `glob`, and `grep`; the deprecated methods call the new APIs and unwrap
payloads when possible.

The compatibility bridges run both directions. If a subclass implements only a
legacy method, the new method can call it with a warning; if caller code still
uses a legacy method on a new backend, the legacy method calls the new method
and unwraps the result.

### `ExecuteResponse`

`ExecuteResponse` is the standard command-execution result: combined output,
optional exit code, and a `truncated` flag. It is intentionally simpler than a
full process object because the result is usually fed back to an LLM.

Combining stdout and stderr preserves the chronological/error context the model
needs without forcing every renderer to handle two streams.

### `SandboxBackendProtocol`

`SandboxBackendProtocol` extends `BackendProtocol` with shell execution. It
requires an `id` property and an `execute(command, timeout=None)` method.
Backends that implement it can power the SDK's `execute` tool; backends that do
not implement it can still support file operations.

`aexecute()` is sync-first like the file-operation async methods, but it has an
extra compatibility guard: if the caller supplies `timeout` and the backend's
`execute()` signature supports it, the timeout is forwarded; otherwise the call
is made without that keyword. The middleware normally validates timeout support
before this point, so this guard mainly protects direct backend callers.

### `execute_accepts_timeout(cls)`

This cached helper introspects a sandbox backend class to see whether its
`execute()` method accepts a `timeout` parameter. It exists because older
backend packages may not have raised their minimum SDK dependency when
`timeout` was added to the protocol.

If the signature cannot be inspected, the helper logs a warning and returns
`False`. Results are cached by class to avoid repeated reflection on every
command execution.

### `BackendFactory`, `BACKEND_TYPES`

`BackendFactory` is a callable from LangChain `ToolRuntime` to
`BackendProtocol`. It lets middleware choose or construct a backend at tool
execution time rather than using one fixed instance. `BACKEND_TYPES` is the
runtime type alias for "backend instance or backend factory".

## Flow walk-through

1. `graph.py:501` chooses `StateBackend()` if the caller did not pass a
   backend.
2. `graph.py:529` and `graph.py:658` pass that backend into
   `FilesystemMiddleware` for subagents and the main agent.
3. Filesystem tools call `BackendProtocol` methods and receive the result
   dataclasses documented here.
4. If a backend also implements `SandboxBackendProtocol`, the `execute` tool
   can run shell commands and return `ExecuteResponse`.

## Gotchas

The protocol does not enforce permissions. Permissions are checked by
`FilesystemMiddleware` before it calls backend methods. Direct backend callers
are responsible for their own safety checks.

The protocol is sync-first. Async defaults are convenient, but they use worker
threads; high-throughput remote backends should provide native async overrides.
