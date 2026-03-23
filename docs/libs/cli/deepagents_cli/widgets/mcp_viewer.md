# `widgets/mcp_viewer.py`

## High-Level Purpose

This module defines the `MCPViewerScreen` — a read-only modal that lists all connected MCP (Model Context Protocol) servers and their available tools. Accessible via the `/mcp` slash command.

## Classes

### `MCPToolItem`

**Inherits from:** `textual.widgets.Static`

A selectable tool item in the MCP viewer list.

**Constructor:**
```python
MCPToolItem(name: str, description: str, index: int, *, classes: str = "")
```

**Parameters:**
- `name`: Tool name.
- `description`: Full tool description.
- `index`: Flat index of this tool in the overall list.

**Behavior:**
- Collapsed by default: shows `name` and truncated description.
- Clicking toggles expanded view (shows full description on multiple lines).
- Smart truncation: if the description would overflow the widget width, it is cut with `(...)`.

**Methods:**
- `toggle_expand() -> None` — Toggles between collapsed and expanded display.
- `_format_collapsed(name, description) -> Content` — Builds the single-line label.

### `MCPServerItem`

**Inherits from:** `textual.containers.Vertical`

A server section header in the MCP viewer.

**Displays:**
- Server name with transport type badge (`stdio`, `sse`, `http`)
- All `MCPToolItem` children indented below it

### `MCPViewerScreen`

**Inherits from:** `textual.screen.ModalScreen`

Read-only MCP server and tool viewer modal.

**Bindings:**
| Key | Action | Description |
|---|---|---|
| `escape` | `dismiss` | Close the modal |
| `up / k` | `move_up` | Navigate up |
| `down / j` | `move_down` | Navigate down |
| `enter / space` | `toggle_expand` | Expand/collapse selected tool description |

**Constructor:**
```python
MCPViewerScreen(mcp_server_info: list[MCPServerInfo])
```

**Behavior:**
1. Groups tools by server.
2. For each server, shows name and transport type.
3. For each tool, shows name and (truncated) description.
4. Tools can be expanded to see full descriptions.
5. Empty server list shows a "No MCP servers connected" message.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.screen.ModalScreen` | textual | Modal dialog base |
| `MCPServerInfo` | `deepagents_cli.mcp_tools` | Server/tool metadata |
| `theme` | `deepagents_cli.theme` | Brand colors |
| `get_glyphs`, `is_ascii_mode` | `deepagents_cli.config` | Glyph selection |
