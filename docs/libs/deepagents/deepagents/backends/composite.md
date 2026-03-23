# `deepagents/backends/composite.py`

## High-Level Purpose

`CompositeBackend` is a meta-backend that routes file operations to different backend implementations based on path prefixes. This allows using, for example, `StateBackend` for temporary working files while routing `/memories/` paths to `StoreBackend` for persistent storage — all within a single unified filesystem seen by the agent.

## Key Design

- Routes are matched by longest prefix first, ensuring more specific routes take precedence.
- Each file operation strips the route prefix before forwarding to the target backend, then re-prepends it on the way back (for paths in results).
- **Execution (`execute`)** is not path-routable; it always delegates to the `default` backend. If `default` is not a `SandboxBackendProtocol`, `execute` raises `NotImplementedError`.
- Root listing (`ls("/")`) aggregates the default backend's listing with a virtual directory entry for each route.

## Module-Level Helper Functions

### `_route_for_path(*, default, sorted_routes, path) -> tuple[BackendProtocol, str, str | None]`

Core routing logic. Given a path, finds the matching backend and returns:
1. The selected `BackendProtocol`
2. The normalized path to pass to that backend (with route prefix stripped)
3. The matched route prefix (or `None` if the default backend is used)

Matching rules:
- Exact match on the route root (e.g., `/memories` → backend root `/`)
- Prefix match with path separator (e.g., `/memories/note.txt` → `/note.txt`)

### `_strip_route_from_pattern(pattern, route_prefix) -> str`
For glob operations: strips the route prefix from a glob pattern when the pattern targets that route's subtree.

### `_remap_grep_path(m: GrepMatch, route_prefix) -> GrepMatch`
Prepends the route prefix to a grep match's `path` field.

### `_remap_file_info_path(fi: FileInfo, route_prefix) -> FileInfo`
Prepends the route prefix to a `FileInfo`'s `path` field.

## Class: `CompositeBackend(BackendProtocol)`

### Constructor

```python
CompositeBackend(
    default: BackendProtocol | StateBackend,
    routes: dict[str, BackendProtocol],
)
```

**Parameters:**
- `default` — Backend for paths that don't match any route.
- `routes` — Dict mapping path prefixes to backends. Prefixes must start with `/` and should end with `/` (e.g., `{"/memories/": store_backend}`).

**Key Attributes:**
- `self.default` — The default backend.
- `self.routes` — The raw route dict.
- `self.sorted_routes` — Routes sorted by length (longest first) for correct matching.

### Methods

#### `ls(path) / als(path)`
- If path matches a route: delegates to that backend, remaps result paths.
- If path is `/`: aggregates default backend's listing with virtual directory entries for each route.
- Otherwise: delegates to default backend.

#### `read / aread`
Routes to the appropriate backend and returns the result unchanged.

#### `grep / agrep`
- If path targets a specific route: searches only that backend.
- If path is `None` or `/`: searches default and all route backends, merges results with remapped paths.

#### `glob / aglob`
- If path targets a specific route: searches only that backend.
- Otherwise: searches default AND all route backends (routing is additive for glob). Strips route prefix from pattern when appropriate.

#### `write / awrite`
Routes write to the appropriate backend. Additionally performs best-effort state sync: if the write produces a `files_update` and the default backend has a `runtime.state`, merges the update into state so that root `ls()` remains accurate.

#### `edit / aedit`
Same routing and state-sync logic as `write`.

#### `execute / aexecute`
Always uses `self.default`. Checks for `SandboxBackendProtocol` and `execute_accepts_timeout` support before forwarding `timeout`. Raises `NotImplementedError` if `default` does not support execution.

#### `upload_files / aupload_files`
Groups files by their target backend, calls each backend's `upload_files` once with all its files (batching for efficiency), then reconstructs results in original input order.

#### `download_files / adownload_files`
Same batching strategy as `upload_files`.

## Example Usage

```python
from deepagents.backends.composite import CompositeBackend
from deepagents.backends.state import StateBackend
from deepagents.backends.store import StoreBackend

composite = CompositeBackend(
    default=StateBackend,   # factory: instantiated per tool call
    routes={"/memories/": StoreBackend(runtime, namespace=lambda ctx: ("memories",))}
)
```
