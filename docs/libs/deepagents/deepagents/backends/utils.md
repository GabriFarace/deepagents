# `libs/deepagents/deepagents/backends/utils.py`

> Shared helpers for backend file data, path normalization, line formatting,
> globbing, and grep result shaping.

## Position in the system

Backend implementations use this module to keep storage formats and tool-facing
behavior consistent. `StateBackend` and `StoreBackend` rely on the in-memory
helpers heavily; `FilesystemBackend` uses replacement and empty-content
helpers; middleware uses formatting helpers when rendering file content to the
model.

## Imports and module-level state

The module imports `wcmatch.glob` for richer glob semantics, `PurePosixPath`
for virtual path handling, and the protocol `FileData`, `GrepResult`, and
typed match/info shapes. Constants define empty-file messaging, file-type
classification by extension, line-number formatting widths, approximate token
limits, and truncation guidance. `FileInfo` and `GrepMatch` are re-exported for
backwards compatibility.

## Functions and classes

### `FileType`

`FileType` is a literal type describing coarse file categories:
`"text"`, `"image"`, `"audio"`, `"video"`, and `"file"`. Unknown extensions
default to text so ordinary source files do not need an allowlist.

### `_normalize_content(file_data)`

This is the central compatibility conversion for old file data. Modern
`FileData` stores `content` as a string; legacy records may store a list of
lines. The helper warns on list content and joins with `\n`.

### `sanitize_tool_call_id(tool_call_id)`

This replaces `.`, `/`, and backslash with underscores so a tool-call ID can be
used safely in generated paths. It is intentionally simple and does not attempt
full path validation.

### `format_content_with_line_numbers(content, start_line=1)`

This renders content in `cat -n` style using a fixed-width line number column.
Lines longer than `MAX_LINE_LENGTH` are split into chunks and labeled with
continuation markers such as `5.1`, `5.2`.

The helper accepts either a string or a list of lines. For strings, a final
empty segment created by a trailing newline is dropped so rendered line numbers
match visible lines.

### `check_empty_content(content)`

This returns the standard empty-file reminder when content is empty or only
whitespace. Backends use it so an LLM sees a clear signal instead of a blank
tool result.

### `_get_file_type(path)`

This classifies a path by suffix using `_EXTENSION_TO_FILE_TYPE`. Image, audio,
video, PDF, and PowerPoint formats are treated as non-text; everything else is
treated as text.

### `_to_legacy_file_data(file_data)`

This converts modern `FileData` to the v1 storage shape: `content` split on
`\n`, timestamps copied, and no `encoding` field. `StateBackend` and
`StoreBackend` use it when configured with `file_format="v1"`.

### `file_data_to_string(file_data)`

This public helper returns plain text content from either modern or legacy
`FileData` by delegating to `_normalize_content()`.

### `create_file_data(content, created_at=None, encoding="utf-8")`

This builds a timestamped `FileData` dictionary. If `created_at` is omitted,
the current UTC timestamp becomes both `created_at` and `modified_at`; otherwise
the supplied creation timestamp is preserved and only `modified_at` is current.

### `update_file_data(file_data, content)`

This returns a new `FileData` with updated content, preserved encoding,
preserved `created_at` when available, and a fresh `modified_at` timestamp.
Edit operations use it after successful replacement.

### `slice_read_response(file_data, offset, limit)`

This slices text content by line without adding line numbers. It normalizes
CRLF and bare CR to LF, preserves line terminators with
`splitlines(keepends=True)`, and returns either the raw sliced string or a
`ReadResult` error when the offset exceeds file length.

### `perform_string_replacement(content, old_string, new_string, replace_all=False)`

This is the exact-match edit validator shared by state, store, and filesystem
backends. It counts occurrences, errors when no match exists, errors on
multiple matches unless `replace_all=True`, and otherwise returns
`(new_content, occurrences)`.

It includes a specialized EOF-newline diagnostic: if the only mismatch is that
`old_string` ends with a newline but the file does not, the returned error tells
the model to retry without that trailing newline and possibly add context.

### `truncate_if_too_long(result)`

This overload-friendly helper truncates either a list of strings or one string
using a rough four-characters-per-token estimate. It appends the standard
truncation guidance when the result exceeds `TOOL_RESULT_TOKEN_LIMIT`.

### `to_posix_path(path)`

This best-effort helper replaces backslashes with forward slashes before
virtual-path parsing. It helps Windows-style paths behave sensibly with
`PurePosixPath`.

### `validate_path(path, *, allowed_prefixes=None)`

This validates a virtual filesystem path. It rejects path traversal
components, leading `~`, and Windows absolute paths, normalizes separators and
leading slash, and optionally enforces an allowed-prefix list.

### `_normalize_path(path)`

This normalizes optional search paths for in-memory helpers. `None` becomes
`/`, relative paths gain a leading slash, and trailing slashes are removed
except for root. Empty paths raise `ValueError`.

### `_filter_files_by_path(files, normalized_path)`

This filters an in-memory file mapping by exact file match or directory prefix.
Root matches all slash-prefixed paths; non-root directories match descendants
under `normalized_path + "/"`.

### `_glob_search_files(files, pattern, path="/")`

This matches in-memory file paths against a glob pattern relative to `path`.
Patterns without separators match only the current directory; recursive search
requires `**`. Matches are sorted by `modified_at` descending and returned as a
newline-separated string, or `"No files found"`.

### `_format_grep_results(results, output_mode)`

This formats the legacy grep results dictionary into one of three output modes:
file names, per-file counts, or content with line numbers. It sorts file paths
for deterministic output.

### `_grep_search_files(files, pattern, path=None, glob=None, output_mode="files_with_matches")`

This older helper searches in-memory file content using a regex pattern and
returns formatted text. It compiles the regex, filters by path and optional
filename glob, and emits `"No matches found"` or an invalid-regex message
instead of raising.

### `grep_matches_from_files(files, pattern, path=None, glob=None)`

This is the structured replacement for `_grep_search_files()` used by modern
backends. It performs literal substring search, returns `GrepResult`, and
keeps invalid paths as empty match lists rather than exceptions.

### `build_grep_results_dict(matches)`

This groups structured `GrepMatch` objects by path into the legacy
`dict[str, list[tuple[int, str]]]` format expected by `_format_grep_results()`.

### `format_grep_matches(matches, output_mode)`

This formats modern structured grep matches through the existing legacy
formatter. Empty matches return `"No matches found"`.

## Gotchas

There are two grep paths: `_grep_search_files()` is regex-based and returns a
formatted string, while `grep_matches_from_files()` is literal and returns a
structured `GrepResult`. New backend implementations should prefer the
structured literal helper to match `BackendProtocol.grep()`.
