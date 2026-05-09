# `libs/deepagents/deepagents/backends/composite.py`

> Path-prefix router that lets one agent use multiple backend implementations
> behind a single `BackendProtocol`.

## Position in the system

`CompositeBackend` sits between filesystem tools and concrete backends. It
routes paths such as `/memories/note.md` to a configured backend while sending
unmatched paths to a default backend. This lets agents combine ephemeral state,
durable stores, local disk, and shell execution in one tool surface.

## Imports and module-level state

The module imports protocol result types, `SandboxBackendProtocol`, and
`execute_accepts_timeout()` so execution can be forwarded safely to older
backends. `defaultdict` is used to batch upload/download calls by target
backend. There is no mutable global state.

## Functions and classes

### `_remap_grep_path(m, route_prefix)`

This helper returns a copy of a routed backend's `GrepMatch` with the external
route prefix restored. If a store backend searched `/note.md` under the
`/memories/` route, callers see `/memories/note.md`.

### `_strip_route_from_pattern(pattern, route_prefix)`

When a global glob pattern already includes a route prefix, this helper removes
that prefix before querying the routed backend. For example,
`/memories/**/*.md` becomes `**/*.md` for the backend mounted at
`/memories/`.

### `_remap_file_info_path(fi, route_prefix)`

This is the `FileInfo` equivalent of `_remap_grep_path()`. It restores the
route prefix on listing and glob results so callers never see the routed
backend's internal stripped paths.

### `_route_for_path(default, sorted_routes, path)`

This is the router's core matching function. It checks routes longest-first,
matches either the exact route root without a trailing slash or any path below
the route prefix, and returns `(backend, backend_path, route_prefix)`.
Unmatched paths return the default backend, original path, and `None`.

The longest-first sorting matters when routes overlap, such as `/memories/`
and `/memories/private/`.

### `CompositeBackend(default, routes, *, artifacts_root="/")`

The constructor stores the default backend, route mapping, routes sorted by
prefix length, and an `artifacts_root` hint used by higher-level middleware for
artifact placement. Routes should start with `/` and usually end with `/`.

#### `_get_backend_and_key(key)`

This small helper calls `_route_for_path()` and returns only the selected
backend plus stripped backend path. Single-path operations use it to avoid
duplicating route logic.

#### `_coerce_ls_result(raw)`

This compatibility helper wraps old list-returning `ls()` implementations in
`LsResult`. It keeps the composite router tolerant of older custom backends
while still returning the modern result type.

#### `ls(path)` and `als(path)`

If `path` is inside a route, `ls()` asks only that backend and remaps returned
paths. If `path == "/"`, it merges the default backend's root listing with
synthetic directory entries for every route. Other unmatched paths go only to
the default backend.

The async version mirrors this behavior with `await` and async backend methods.

#### `read(file_path, offset=0, limit=2000)` and `aread(...)`

Read operations route to exactly one backend. The external path is stripped to
the backend-local path before delegation, and read results are returned as-is
because file content does not contain paths to remap.

#### `_coerce_grep_result(raw)`

This compatibility helper normalizes older grep returns: a `GrepResult` passes
through, a string becomes an error result, and a raw list of matches becomes a
successful result.

#### `grep(pattern, path=None, glob=None)` and `agrep(...)`

If the requested `path` targets one route, grep searches only that backend and
remaps match paths. If `path` is `None` or `/`, it searches the default backend
and every route, merging matches. Any backend error aborts the combined result.

For non-root paths that do not match a route, the search is delegated only to
the default backend.

#### `glob(pattern, path="/")` and `aglob(...)`

For a routed `path`, glob searches that backend and remaps file paths. For
unmatched paths, it searches the default backend and every route, stripping a
route prefix from the pattern when needed. Results are sorted by path for
deterministic output.

#### `write(file_path, content)` and `awrite(...)`

Write routes to one backend and then rewrites a successful result's `path` back
to the external path. This preserves the user-facing route prefix while letting
the child backend work with its internal path.

#### `edit(file_path, old_string, new_string, replace_all=False)` and `aedit(...)`

Edit follows the same single-backend routing as write. On success, the returned
path is restored to the original external path.

#### `execute(command, *, timeout=None)` and `aexecute(...)`

Execution is not path-routable, so it always delegates to the default backend.
The method checks that the default backend implements `SandboxBackendProtocol`
and uses `execute_accepts_timeout()` before forwarding `timeout=` to support
older custom backends.

#### `upload_files(files)` and `aupload_files(files)`

Bulk upload groups files by target backend, strips route prefixes in each
batch, calls each backend once, and places responses back into the original
input order with original external paths.

#### `download_files(paths)` and `adownload_files(paths)`

Bulk download follows the same batch-by-backend pattern. It restores external
paths and preserves content/error fields from each child backend response.

## Gotchas

Only file paths are routable. Shell execution always goes to the default
backend, so choose the default carefully when composing a filesystem-only store
route with a shell-capable local or sandbox backend.
