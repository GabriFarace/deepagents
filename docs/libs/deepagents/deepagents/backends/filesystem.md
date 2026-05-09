# `libs/deepagents/deepagents/backends/filesystem.py`

> Host filesystem backend for direct disk reads, writes, search, glob, upload,
> and download.

## Position in the system

`FilesystemBackend` implements `BackendProtocol` against real local files. It
is the storage half used by local development agents when shell execution is
not needed. `LocalShellBackend` subclasses it to add `execute()`, and
`CompositeBackend` commonly routes virtual path prefixes into instances running
with `virtual_mode=True`.

## Imports and module-level state

The module uses `pathlib`, `os.open()`, and `O_NOFOLLOW` where available for
file operations, `subprocess` plus ripgrep JSON for fast search, and
`wcmatch.glob` for Python fallback glob matching. `logger` records filesystem
errors encountered during listings and searches. The Win32 symlink-loop code is
a module-level constant used by error classification.

## Functions and classes

### `FilesystemBackend(root_dir=None, virtual_mode=None, max_file_size_mb=10)`

The constructor stores the resolved root directory in `self.cwd`, resolves the
deprecated `virtual_mode=None` default with a warning, and records a maximum
file size for Python fallback search. In legacy non-virtual mode, absolute
paths are used as-is and relative paths resolve below `root_dir`; this is a
convenience behavior, not a security boundary.

With `virtual_mode=True`, incoming paths are treated as virtual paths under
`root_dir`. The resolver blocks `..`, `~`, and paths that escape the root after
resolution. This is especially useful when `CompositeBackend` strips a route
prefix before forwarding paths to a routed backend.

#### `_resolve_path(key)`

This is the central path policy helper. In virtual mode it anchors the request
under `self.cwd`, checks the resolved path remains inside the root, and probes
symlinks for loops. In non-virtual mode it preserves absolute paths and only
joins relative paths to `self.cwd`.

Many public methods return `ReadResult`, `WriteResult`, or `LsResult` errors
when `_resolve_path()` raises. Some invalid virtual-mode paths in search simply
produce empty results because that is easier for model-facing search tools to
recover from.

#### `_to_virtual_path(path)`

This converts a concrete filesystem path back to a `/`-prefixed virtual path
relative to `self.cwd`. Listing, grep, and glob use it in virtual mode so tool
responses expose stable virtual paths instead of host-specific absolute paths.

#### `ls(path)`

`ls()` resolves the directory, iterates only direct children, and returns
sorted `FileInfo` entries. In non-virtual mode entries use absolute paths. In
virtual mode entries use `_to_virtual_path()` and paths outside the root are
skipped.

The method is defensive around flaky filesystems: child stat/resolve failures
are logged, collected as newline-separated partial errors, and do not
necessarily abort the whole listing. If directory iteration itself aborts,
the result carries an error so callers know entries may be incomplete.

#### `read(file_path, offset=0, limit=2000)`

`read()` opens files with `O_NOFOLLOW` when supported. Non-text file types are
read as bytes and base64-encoded. Text files are decoded as UTF-8 and sliced by
line offset/limit while preserving trailing-newline state with
`splitlines(keepends=True)`.

Empty or whitespace-only text files return the standard system reminder string
rather than an empty payload. If the requested offset is beyond the file
length, the method returns a `ReadResult` error.

#### `write(file_path, content)`

`write()` is create-only. It resolves the path, refuses to overwrite an
existing file, creates parent directories, then writes UTF-8 content through
`os.open()` with `O_CREAT | O_TRUNC` and `O_NOFOLLOW` when available.

The file is opened with `newline=""` so Python does not translate line endings
on Windows. This preserves the exact LF/CRLF content the agent requested.

#### `edit(file_path, old_string, new_string, replace_all=False)`

`edit()` reads the target as UTF-8 text, normalizes line endings in the provided
old/new strings to match Python's universal-newline read behavior, and applies
`perform_string_replacement()`. It then truncates and rewrites the file with
`O_NOFOLLOW` where possible.

The exact-match helper enforces uniqueness unless `replace_all=True`, and it
has special messaging for the common "old string includes a trailing newline
but the file does not" failure.

#### `grep(pattern, path=None, glob=None)`

`grep()` searches for a literal string. It first tries ripgrep with `--json`
and `-F`; if ripgrep is missing, times out, or cannot be started, it falls back
to a Python recursive search using an escaped regex. Returned matches are
structured `GrepMatch` dictionaries.

The Python fallback skips non-files, unreadable files, and files larger than
`max_file_size_bytes`. `glob` filters by relative path using `wcmatch`.

#### `_ripgrep_search(pattern, base_full, include_glob)`

This helper builds the ripgrep command and parses JSON lines of type
`"match"`. It returns `None` only when the ripgrep path is unavailable or
unusable, which signals `grep()` to use the Python fallback. It remaps result
paths to virtual paths in virtual mode.

#### `_python_search(pattern, base_full, include_glob)`

This fallback compiles the already-escaped pattern, walks the base directory,
filters by glob and size, decodes readable text files, and records matching
line numbers. It is intentionally best-effort and skips files that cannot be
read without failing the whole search.

#### `glob(pattern, path="/")`

`glob()` strips a leading slash from the pattern, rejects traversal components
in virtual mode, resolves the search path, and walks with `Path.rglob()`.
Matches are files only, sorted by path, and carry best-effort size and
modification metadata.

If the walk aborts partway, the method returns accumulated matches plus an
error message so callers know the result is partial.

#### `upload_files(files)`

`upload_files()` writes byte payloads one by one, creating parent directories
and preserving input order in responses. It maps common filesystem exceptions
to stable `FileOperationError` codes. Unknown exceptions are re-raised rather
than hidden.

#### `download_files(paths)`

`download_files()` reads raw bytes for each requested path with `O_NOFOLLOW`
where available. Directories return `"is_directory"`, missing paths return
`"file_not_found"`, invalid paths return `"invalid_path"`, and permission
errors return `"permission_denied"`.

### `_map_exception_to_standard_error(exc)`

This private helper translates common filesystem exceptions into stable upload
and download error codes. It treats symlink loops as invalid paths and returns
`None` for unexpected exceptions so callers can re-raise them.

### `_is_eloop_oserror(exc)`, `_is_symlink_loop_error(exc)`, and `_raise_if_symlink_loop(path)`

These helpers normalize symlink-loop detection across Python versions and
platforms. Python 3.13 changed some `Path.resolve()` behavior, and Windows can
surface NTFS reparse-point cycles through a raw `winerror`, so the backend uses
explicit probing and classification.

## Gotchas

`root_dir` is not a sandbox unless `virtual_mode=True`, and even virtual mode
is only path validation, not process isolation. Do not expose this backend to
untrusted users without higher-level permission gates or a real sandbox.
