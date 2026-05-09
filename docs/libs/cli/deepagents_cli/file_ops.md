# `libs/cli/deepagents_cli/file_ops.py`

> Helpers for tracking file operations and computing diffs for CLI display.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `ApprovalPreview`

Data used to render HITL previews.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_safe_read(path: Path)`

Read file content, returning None on failure.

Additional notes from the source docstring:

```text
Returns:
    File content as string, or None if reading fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_count_lines(text: str)`

Count lines in text, treating empty strings as zero lines.

Additional notes from the source docstring:

```text
Returns:
    Number of lines in the text.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `compute_unified_diff(before: str, after: str, display_path: str, *, max_lines: int | None=800, context_lines: int=3)`

Compute a unified diff between before and after content.

Additional notes from the source docstring:

```text
Args:
    before: Original content
    after: New content
    display_path: Path for display in diff headers
    max_lines: Maximum number of diff lines (None for unlimited)
    context_lines: Number of context lines around changes (default 3)

Returns:
    Unified diff string or None if no changes
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `FileOpMetrics`

Line and byte level metrics for a file operation.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `FileOperationRecord`

Track a single filesystem tool call.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `resolve_physical_path(path_str: str | None, assistant_id: str | None)`

Convert a virtual/relative path to a physical filesystem path.

Additional notes from the source docstring:

```text
Returns:
    Resolved physical Path, or None if path is empty or resolution fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_display_path(path_str: str | None)`

Format a path for display.

Additional notes from the source docstring:

```text
Returns:
    Formatted path string suitable for display.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `build_approval_preview(tool_name: str, args: dict[str, Any], assistant_id: str | None)`

Collect summary info and diff for HITL approvals.

Additional notes from the source docstring:

```text
Returns:
    ApprovalPreview with diff and details, or None if tool not supported.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `FileOpTracker`

Collect file operation metrics during a CLI interaction.

Methods worth reading inside this class:

- `start_operation(self, tool_name: str, args: dict[str, Any], tool_call_id: str | None)`: Begin tracking a file operation.

- `complete_with_message(self, tool_message: Any)`: Complete a file operation with the tool message result.

- `mark_hitl_approved(self, tool_name: str, args: dict[str, Any])`: Mark operations matching tool_name and file_path as HIL-approved.

- `_populate_after_content(self, record: FileOperationRecord)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `_finalize(self, record: FileOperationRecord)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
