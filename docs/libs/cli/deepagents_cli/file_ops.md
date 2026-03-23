# `file_ops.py`

## High-Level Purpose

This module provides helpers for tracking file operations and computing diffs for CLI display. It supports the Human-in-the-Loop (HITL) approval system by generating before/after previews and unified diffs of proposed file changes.

## Type Aliases

| Alias | Type | Description |
|---|---|---|
| `FileOpStatus` | `Literal["pending", "success", "error"]` | Status of a file operation |

## Classes

### `ApprovalPreview`

**Type:** `dataclass`

Data used to render HITL previews for tool approval.

| Attribute | Type | Description |
|---|---|---|
| `title` | `str` | Main title for the preview |
| `details` | `list[str]` | Detail lines to display |
| `diff` | `str \| None` | Unified diff string if applicable |
| `diff_title` | `str \| None` | Title for the diff section |
| `error` | `str \| None` | Error message if preview generation failed |

### `FileOpMetrics`

**Type:** `dataclass`

Line and byte-level metrics for a file operation.

| Attribute | Type | Description |
|---|---|---|
| `lines_read` | `int` | Number of lines read |
| `start_line` | `int \| None` | Starting line number for range reads |
| `end_line` | `int \| None` | Ending line number for range reads |
| `bytes_read` | `int` | Number of bytes read |
| `lines_written` | `int` | Number of lines written |
| `bytes_written` | `int` | Number of bytes written |

## Functions

### `_safe_read(path: Path) -> str | None`

Reads file content safely, returning `None` on any failure (OSError, UnicodeDecodeError).

**Returns:** File content as string, or `None`.

### `_count_lines(text: str) -> int`

Counts lines in text, treating empty strings as zero lines.

**Returns:** Number of lines.

### `compute_unified_diff(before, after, display_path, *, max_lines=800, context_lines=3) -> str | None`

Computes a unified diff between original (`before`) and new (`after`) content.

**Parameters:**
- `before`: Original content string.
- `after`: New content string.
- `display_path`: Path string used in the diff headers.
- `max_lines`: Maximum diff lines to include (truncates with `"..."` if exceeded). Pass `None` for unlimited.
- `context_lines`: Number of context lines around changes (default 3).

**Returns:** Unified diff string, or `None` if no changes exist.

**Key Logic:** Uses `difflib.unified_diff` to generate the diff. If `max_lines` is exceeded, the diff is truncated and a `"..."` line is appended.

### `build_approval_preview(tool_name: str, tool_args: dict, backend: BackendProtocol | None = None) -> ApprovalPreview`

Builds an `ApprovalPreview` for a tool call, computing diffs when the tool modifies files.

**Parameters:**
- `tool_name`: Name of the tool being called.
- `tool_args`: Arguments passed to the tool.
- `backend`: Optional backend for reading current file content.

**Returns:** `ApprovalPreview` with populated fields.

**Supported tools with diff preview:**
- `write_file` — Shows full file content diff
- `edit_file` / `str_replace_editor` — Shows targeted edit diff
- `shell` — Shows the command being executed

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `difflib` | stdlib | Unified diff generation |
| `pathlib.Path` | stdlib | File path handling |
| `BackendProtocol` | `deepagents.backends.protocol` | Backend type hint for file reading |
