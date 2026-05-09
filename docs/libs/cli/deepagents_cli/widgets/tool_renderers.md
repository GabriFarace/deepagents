# `libs/cli/deepagents_cli/widgets/tool_renderers.py`

> Compact renderers for known tool calls.

## Functions and classes

### `ToolRenderer`

Base display interface.

### `WriteFileRenderer`, `TaskRenderer`, `EditFileRenderer`

Specialized displays for common file and subagent tools.

### `get_renderer(tool_name)`

Dispatches tool names to renderer instances.

## Gotchas

Renderers are display-only and must tolerate partial tool args.
