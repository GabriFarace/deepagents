# `deepagents/backends/utils.py`

## High-Level Purpose

Shared utility functions used by all backend implementations. Covers:
- File content formatting (line numbers, truncation)
- Path normalization and validation
- String replacement logic
- Glob and grep search over in-memory file dicts
- File type classification by extension
- `FileData` creation and update helpers

This module is the single centralized location for shared logic, enabling backends and middleware to compose without duplicating parsing or formatting code.

## Constants

| Constant | Value | Description |
|---|---|---|
| `EMPTY_CONTENT_WARNING` | `"System reminder: File exists but has empty contents"` | Returned when a file exists but is empty |
| `MAX_LINE_LENGTH` | `5000` | Characters per line before continuation markers are added |
| `LINE_NUMBER_WIDTH` | `6` | Width of line number column in formatted output |
| `TOOL_RESULT_TOKEN_LIMIT` | `20000` | Approximate token limit for tool results (for truncation) |
| `TRUNCATION_GUIDANCE` | `"... [results truncated, ...]"` | Appended when results are truncated |

## Type Aliases

### `FileType`
`Literal["text", "image", "audio", "video", "file"]` — File classification by extension.

### `_EXTENSION_TO_FILE_TYPE`
Dict mapping file extensions to `FileType`. Covers common image, video, audio, and document formats (PDF, PPT).

## Functions

### File Content Operations

#### `format_content_with_line_numbers(content, start_line=1) -> str`
Formats file content with `cat -n` style line numbers. Long lines (> `MAX_LINE_LENGTH` chars) are split into chunks with continuation markers like `5.1`, `5.2`.

**Parameters:**
- `content` — String or `list[str]` of lines.
- `start_line` — Starting line number (default: 1).

**Returns:** Formatted string with `"{line_num}\t{content}"` for each line.

---

#### `check_empty_content(content: str) -> str | None`
Returns `EMPTY_CONTENT_WARNING` if `content` is empty/whitespace, otherwise `None`.

---

#### `create_file_data(content, created_at=None, encoding="utf-8") -> FileData`
Creates a new `FileData` dict with current UTC timestamp as `modified_at`.

**Parameters:**
- `content` — File content string (plain text or base64).
- `created_at` — Optional ISO timestamp; defaults to now.
- `encoding` — `"utf-8"` or `"base64"`.

---

#### `update_file_data(file_data, content) -> FileData`
Returns a new `FileData` with updated `content` and `modified_at`, preserving the original `created_at` and `encoding`.

---

#### `file_data_to_string(file_data: FileData) -> str`
Converts `FileData` to a plain string. Handles legacy `list[str]` content via `_normalize_content`.

---

#### `slice_read_response(file_data, offset, limit) -> str | ReadResult`
Slices file content to the `[offset, offset+limit)` line range without adding line number formatting. Returns a `ReadResult(error=...)` if `offset` exceeds file length.

---

#### `format_read_response(file_data, offset, limit) -> str` *(deprecated)*
Combines slicing and line number formatting. Use `slice_read_response` + `format_content_with_line_numbers` separately.

---

#### `perform_string_replacement(content, old_string, new_string, replace_all=False) -> tuple[str, int] | str`
Core string replacement logic used by all backends' `edit()` methods.

**Returns:** `(new_content, occurrences)` on success, or an error message string on failure. Error cases:
- `old_string` not found
- Multiple occurrences and `replace_all=False`

### Path Operations

#### `validate_path(path, *, allowed_prefixes=None) -> str`
Security-focused path normalization for virtual filesystem paths. Prevents `..` traversal, `~`, and Windows absolute paths. Normalizes to a `/`-prefixed POSIX path. Optionally enforces allowed prefixes.

---

#### `_normalize_path(path: str | None) -> str`
Normalizes a path to canonical form: absolute, `/`-prefixed, no trailing slash (except root `/`).

---

#### `_filter_files_by_path(files, normalized_path) -> dict`
Filters a files dict to entries within the specified path. Handles exact file matches and directory prefix matches.

### Search Operations

#### `grep_matches_from_files(files, pattern, path=None, glob=None) -> GrepResult`
Structured grep over an in-memory files dict. Performs **literal substring search** (not regex). Optionally filters by path prefix and `glob` filename pattern. Returns `GrepResult` with structured `GrepMatch` entries.

---

#### `_glob_search_files(files, pattern, path="/") -> str`
Searches a files dict for paths matching a glob pattern. Returns newline-separated paths sorted by `modified_at` (most recent first), or `"No files found"`.

---

#### `_grep_search_files(files, pattern, path=None, glob=None, output_mode="files_with_matches") -> str`
Internal regex search over a files dict. Supports output modes: `"files_with_matches"`, `"content"`, `"count"`. Returns formatted string result.

---

#### `build_grep_results_dict(matches) -> dict`
Groups a list of `GrepMatch` dicts into `{file_path: [(line_num, line_text), ...]}` format.

---

#### `format_grep_matches(matches, output_mode) -> str`
Formats structured grep matches into a string using the specified output mode.

### Miscellaneous

#### `_get_file_type(path: str) -> FileType`
Classifies a file by its extension. Returns `"text"` for unrecognized extensions.

---

#### `sanitize_tool_call_id(tool_call_id: str) -> str`
Replaces `.`, `/`, `\` in tool call IDs with underscores to prevent path traversal when used as directory names.

---

#### `truncate_if_too_long(result: list[str] | str) -> list[str] | str`
Truncates results that exceed `TOOL_RESULT_TOKEN_LIMIT * 4` characters. Appends `TRUNCATION_GUIDANCE` at the truncation point.

### Private / Internal

#### `_normalize_content(file_data: FileData) -> str`
Single backwards-compatibility conversion point: joins `list[str]` content with `"\n"` for legacy `v1` format. Emits `DeprecationWarning`.

#### `_to_legacy_file_data(file_data: FileData) -> dict`
Converts modern `FileData` (v2) to legacy v1 format (`content: list[str]`, no `encoding` key).
