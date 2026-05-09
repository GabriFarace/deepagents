# `libs/cli/deepagents_cli/widgets/mcp_viewer.py`

> Read-only MCP server and tool viewer modal.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.containers import Vertical, VerticalScroll`

- `from textual.content import Content`

- `from textual.events import Click`

- `from textual.screen import ModalScreen`

- `from textual.widgets import Static`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`


## Functions and classes

### `MCPToolItem`

A selectable tool item in the MCP viewer.

Methods worth reading inside this class:

- `_desc_style(self)`: Return the markup style tag for the description span.

- `_format_collapsed(self, name: str, description: str)`: Build the collapsed (single-line) label.

- `_format_expanded(self, name: str, description: str)`: Build the expanded (multi-line) label.

- `_rerender(self)`: Re-render the label with the current selected/expanded state.

- `set_selected(self, selected: bool)`: Apply or remove the selected-row styling and re-render the label.

- `toggle_expand(self)`: Toggle between collapsed and expanded view.

- `on_mount(self)`: Re-render with correct truncation once width is known.

- `on_resize(self)`: Re-truncate when widget width changes.

- `on_click(self, event: Click)`: Handle click — select and toggle expand via parent screen.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `MCPViewerScreen`

Modal viewer for active MCP servers and their tools.

Additional notes from the source docstring:

```text
Displays servers grouped by name with transport type and tool count.
Navigate with arrow keys, Enter to expand/collapse tool descriptions,
Escape to close.
```

Methods worth reading inside this class:

- `refresh_server_info(self, server_info: list[MCPServerInfo])`: Replace the displayed server list; typically after server startup.

- `compose(self)`: Compose the screen layout.

- `on_mount(self)`: Build the body once the screen is mounted.

- `_mount_body(self, container: Vertical)`: Populate `container` with the title, list, and help footer.

- `_move_to(self, index: int)`: Move selection to the given index.

- `_move_selection(self, delta: int)`: Move selection by delta positions.

- `action_move_up(self)`: Move selection up.

- `action_move_down(self)`: Move selection down.

- `action_toggle_expand(self)`: Toggle expand/collapse on the selected tool.

- `action_page_up(self)`: Scroll up by one page.

- `action_page_down(self)`: Scroll down by one page.

- `action_cancel(self)`: Close the viewer.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
