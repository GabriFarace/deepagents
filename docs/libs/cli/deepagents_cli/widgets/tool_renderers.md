# `widgets/tool_renderers.py`

## High-Level Purpose

This module implements a **registry pattern** for selecting the correct approval widget for each tool type. When the `ApprovalMenu` needs to display tool-specific details, it calls `get_renderer(tool_name)` to get the appropriate renderer class, which then returns the correct `ToolApprovalWidget` subclass and processed data.

## Classes

### `ToolRenderer`

Base renderer class. Returns `GenericApprovalWidget` as the default fallback.

**Methods:**

#### `get_approval_widget(tool_args: dict) -> tuple[type[ToolApprovalWidget], dict]`

Returns the approval widget class and processed data dict for this tool.

**Returns:** `(GenericApprovalWidget, tool_args)` by default.

### `WriteFileRenderer`

**Inherits from:** `ToolRenderer`

Renderer for the `write_file` tool. Extracts and adds `file_extension` to the data dict for syntax highlighting.

**Returns:** `(WriteFileApprovalWidget, enhanced_data)`.

### `EditFileRenderer`

**Inherits from:** `ToolRenderer`

Renderer for `edit_file` / `str_replace_editor` tools. Prepares before/after content for diff display.

**Returns:** `(EditFileApprovalWidget, diff_data)`.

## Module-Level Registry

### `get_renderer(tool_name: str) -> ToolRenderer`

Returns the appropriate renderer for a given tool name.

**Tool name to renderer mapping:**
| Tool Name | Renderer |
|---|---|
| `write_file` | `WriteFileRenderer` |
| `edit_file` | `EditFileRenderer` |
| `str_replace_editor` | `EditFileRenderer` |
| Everything else | `ToolRenderer` (generic) |

**Returns:** A `ToolRenderer` instance (generic if not found).

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `difflib` | stdlib | Edit diff computation in `EditFileRenderer` |
| `GenericApprovalWidget`, `WriteFileApprovalWidget`, `EditFileApprovalWidget` | `widgets.tool_widgets` | Target widget classes |
