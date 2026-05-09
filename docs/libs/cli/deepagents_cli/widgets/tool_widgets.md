# `libs/cli/deepagents_cli/widgets/tool_widgets.py`

> Tool-specific approval widgets.

## Functions and classes

### `_format_stats()`, `_file_header()`, `_count_diff_stats()`

Build concise diff/file metadata.

### `ToolApprovalWidget`, `GenericApprovalWidget`, `WriteFileApprovalWidget`, `EditFileApprovalWidget`

Render approval UI for generic tools, new files, and edits.

## Gotchas

The graph remains paused until the app resumes it with the selected approval
decision.
