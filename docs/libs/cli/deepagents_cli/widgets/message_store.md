# `widgets/message_store.py`

## High-Level Purpose

This module provides the `MessageStore` class and associated data structures for **virtualized chat history**. Rather than keeping all message widgets in the DOM (which would slow down Textual for long conversations), the store holds all message data as lightweight `MessageData` dataclasses and only renders a sliding window of widgets in the DOM.

The design is inspired by Textual's `Log` widget, which keeps only `N` lines in the DOM.

## Classes

### `MessageType`

**Type:** `StrEnum`

Types of messages in the chat.

| Value | Description |
|---|---|
| `USER` | User input message |
| `ASSISTANT` | AI assistant response |
| `TOOL` | Tool call and result |
| `SKILL` | Skill invocation |
| `ERROR` | Error message |
| `APP` | App-level informational message |
| `SUMMARIZATION` | Conversation summarization event |
| `DIFF` | File diff display |

### `ToolStatus`

**Type:** `StrEnum`

Status of a tool call.

| Value | Description |
|---|---|
| `PENDING` | Tool call queued but not started |
| `RUNNING` | Tool call in progress |
| `SUCCESS` | Tool call completed successfully |
| `ERROR` | Tool call failed |
| `REJECTED` | Tool call rejected by user |
| `SKIPPED` | Tool call skipped |

### `MessageData`

**Type:** `dataclass`

In-memory message data for virtualization. Designed to be lightweight so thousands of messages can be stored without significant memory overhead.

| Attribute | Type | Description |
|---|---|---|
| `type` | `MessageType` | Kind of message |
| `content` | `str` | Primary text content |
| `id` | `str` | Unique identifier matching the DOM widget ID |
| `timestamp` | `float` | Unix epoch creation timestamp |
| `tool_name` | `str \| None` | Tool name (TOOL messages only) |
| `tool_args` | `dict \| None` | Tool arguments (TOOL messages only) |
| `tool_status` | `ToolStatus \| None` | Execution status (TOOL messages only) |
| `tool_output` | `str \| None` | Tool output text |
| `tool_expanded` | `bool` | Whether tool output is expanded |
| `skill_expanded` | `bool` | Whether skill output is expanded |
| `is_streaming` | `bool` | Whether this message is being streamed |
| `height_hint` | `int \| None` | Cached height hint for layout |
| `mode` | `str \| None` | Input mode (`'shell'`, `'command'`) |

**Updatable fields** (via `update_message()`): `content`, `tool_status`, `tool_output`, `tool_expanded`, `skill_expanded`, `is_streaming`, `height_hint`.

### `MessageStore`

Manages the in-memory message list and DOM widget lifecycle.

**Key Methods:**

#### `add_message(data: MessageData) -> None`
Appends a new message to the store.

#### `update_message(message_id: str, **kwargs) -> MessageData | None`
Updates fields on an existing message. Only fields in `_UPDATABLE_FIELDS` can be updated.

**Returns:** Updated `MessageData`, or `None` if not found.

#### `get_message(message_id: str) -> MessageData | None`
Retrieves a message by ID.

#### `all_messages() -> list[MessageData]`
Returns all messages in order.

#### `clear() -> None`
Clears all messages from the store.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `uuid` | stdlib | Message ID generation |
| `time` | stdlib | Timestamp generation |
| `enum.StrEnum` | stdlib | Enum types |
| `textual.widget.Widget` | textual | Widget type hint |
