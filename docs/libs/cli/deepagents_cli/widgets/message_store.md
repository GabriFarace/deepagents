# `deepagents_cli/widgets/message_store.py`

## High-Level Purpose

`MessageStore` is an in-memory registry of all message widgets currently displayed in the transcript. It lets `CLIApp` and `StreamHandler` look up a specific `ToolCallMessage` by ID to update it when its result arrives, without scanning the entire Textual widget tree.

---

## Key Class

### `MessageStore`

**Key methods:**

#### `register(widget) → None`

Registers a message widget. The widget must have a `message_id` attribute. Used for `ToolCallMessage` and `AssistantMessage` widgets that need to be updated in place.

#### `get(message_id) → MessageWidget | None`

Returns the widget with the given `message_id`, or `None` if not found.

#### `update_tool_call(tool_call_id, result, status) → None`

Convenience method: looks up the `ToolCallMessage` with the matching `tool_call_id`, then sets its result text and status. Called by `StreamHandler` when a `ToolMessage` arrives.

#### `get_assistant_message(message_id) → AssistantMessage | None`

Returns the `AssistantMessage` widget for the current streaming response. Used to append tokens.

#### `clear() → None`

Called on `/clear` to reset the store for a new thread.

---

## Architecture Notes

**Why not use Textual's `query()`?** Textual's DOM query (`app.query(ToolCallMessage)`) scans the widget tree. For a long conversation with many tool calls, this would be O(n) on every token append. `MessageStore` provides O(1) lookups.

**Lifecycle:** The store is created once in `CLIApp.__init__` and cleared on `/clear`. Widget registrations happen in `CLIApp` immediately after `mount()`.

---

## See Also

- [messages.md](messages.md) — the widget classes registered here
- [remote_client.md](../remote_client.md) — `StreamHandler.update_tool_call()` calls `MessageStore`
- [app.md](../app.md) — owns the `MessageStore` instance
