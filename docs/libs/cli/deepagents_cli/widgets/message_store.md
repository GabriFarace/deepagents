# `libs/cli/deepagents_cli/widgets/message_store.py`

> UI-side transcript data model.

## Functions and classes

### `MessageType` and `ToolStatus`

Enums for display message categories and tool lifecycle.

### `MessageData`

Record for one rendered message, including ids, content, timestamps, tool
metadata, artifacts, and display state.

### `MessageStore`

Maintains ordered transcript state, updates streaming assistant text, tracks
tool calls/results, and supplies widgets with renderable records.

## Gotchas

This is not the checkpoint store. It is a UI projection of streamed graph
events.
