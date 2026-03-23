# `widgets/tool_widgets.py`

## High-Level Purpose

This module defines tool-specific approval widget classes used in the HITL `ApprovalMenu`. Each widget renders a preview of what a specific tool is about to do, allowing users to make informed approval decisions.

## Module-Level Constants

| Constant | Value | Description |
|---|---|---|
| `_MAX_VALUE_LEN` | `200` | Max characters before truncating generic values |
| `_MAX_LINES` | `30` | Max lines for content display |
| `_MAX_DIFF_LINES` | `50` | Max diff lines to show |
| `_MAX_PREVIEW_LINES` | `20` | Max lines for file content preview |

## Classes

### `ToolApprovalWidget`

**Inherits from:** `textual.containers.Vertical`

Base class for all tool approval widgets. Subclasses override `compose()` to display tool-specific content.

**Constructor:**
```python
ToolApprovalWidget(data: dict[str, Any])
```

**Default `compose()`:** Yields `Static("Tool details not available")` as placeholder.

### `GenericApprovalWidget`

**Inherits from:** `ToolApprovalWidget`

Generic approval widget for unknown tools. Iterates over all key-value pairs in `data` and displays them as `key: value` lines. Values longer than `_MAX_VALUE_LEN` are truncated with a character count.

### `WriteFileApprovalWidget`

**Inherits from:** `ToolApprovalWidget`

Approval widget for `write_file` tool. Shows the target file path and full file content with syntax highlighting via a Markdown code block.

**Data keys used:** `file_path`, `content`, `file_extension`.

**Content limits:** Shows up to `_MAX_PREVIEW_LINES` lines, with a "... N more lines" indicator if truncated.

### `EditFileApprovalWidget`

**Inherits from:** `ToolApprovalWidget`

Approval widget for `edit_file` / `str_replace_editor` tools. Computes and displays a unified diff between the current file content and the proposed edit.

**Data keys used:** `file_path`, `old_content`, `new_content` (or `old_str`/`new_str`).

**Key Logic:**
1. Uses `difflib.unified_diff` to compute the diff.
2. Shows the diff via `compose_diff_lines` with a `_MAX_DIFF_LINES` limit.
3. Falls back to showing raw old/new content if diff computation fails.

### `ShellApprovalWidget`

**Inherits from:** `ToolApprovalWidget`

Minimal display for shell command approval. Shows only the command being executed (shell commands are already displayed in the tool call message).

### `WebSearchApprovalWidget`

**Inherits from:** `ToolApprovalWidget`

Shows the search query for web search tool approval.

### `UrlFetchApprovalWidget`

**Inherits from:** `ToolApprovalWidget`

Shows the URL being fetched for URL fetch tool approval.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `difflib` | stdlib | Edit diff computation |
| `textual.containers.Vertical` | textual | Base vertical layout |
| `textual.widgets.Markdown, Static` | textual | Content rendering |
| `theme` | `deepagents_cli.theme` | Brand colors |
| `compose_diff_lines` | `widgets.diff` | Diff line rendering (used by `EditFileApprovalWidget`) |
