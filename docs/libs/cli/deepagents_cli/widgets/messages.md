# `widgets/messages.py`

## High-Level Purpose

This module defines the message widget classes used to display different types of messages in the chat history. Each widget type renders a specific kind of content: user messages, assistant responses (with Markdown streaming), tool call status, skill invocations, errors, and app notifications.

## Classes

### `_TimestampClickMixin`

A mixin for any message widget that shows a timestamp toast when clicked. Looks up the message's creation timestamp from the `MessageStore` and displays it as a toast notification.

**Methods:**
- `on_click(event)` — Shows a timestamp toast via `_show_timestamp_toast`.

### `UserMessage`

**Inherits from:** `textual.widgets.Static`, `_TimestampClickMixin`

Displays a user's chat message. Shows the text with syntax highlighting for `@file` mentions and mode prefixes (shell `!`, command `/`). Shows a colored mode badge if the message was sent in shell or command mode.

### `QueuedUserMessage`

**Inherits from:** `textual.widgets.Static`

A dimmed placeholder for a user message that has been submitted but not yet processed (shown while waiting for the agent response).

### `AssistantMessage`

**Inherits from:** `textual.containers.Vertical`

Displays an AI assistant message with Markdown rendering. Supports streaming (token-by-token updates via `MarkdownStream`).

**Key Attributes:**
- `is_streaming: bool` — Whether the message is currently being streamed.

**Key Methods:**
- `start_streaming() -> None` — Switches to streaming mode with a `MarkdownStream` widget.
- `append_text(text: str) -> None` — Appends streamed text tokens.
- `finish_streaming(final_text: str) -> None` — Completes streaming and renders final Markdown.

### `ToolCallMessage`

**Inherits from:** `textual.containers.Vertical`

Displays a tool call with its name, status icon, arguments, and output. Supports collapsible output display.

**States:** pending, running, success, error, rejected, skipped (mapped to `ToolStatus` enum).

**Key Methods:**
- `set_status(status: ToolStatus) -> None` — Updates the status icon and color.
- `set_output(text: str) -> None` — Sets the tool output content.
- `toggle_expand() -> None` — Toggles expanded/collapsed output display.

### `SkillMessage`

**Inherits from:** `textual.containers.Vertical`

Displays a skill invocation with its name, status, and content.

### `ErrorMessage`

**Inherits from:** `textual.widgets.Static`, `_TimestampClickMixin`

Displays an error message styled in the error color.

### `AppMessage`

**Inherits from:** `textual.widgets.Static`

Displays informational app-level messages (e.g., `/new thread started`, model switch confirmations).

## Module-Level Functions

### `_show_timestamp_toast(widget) -> None`

Shows a toast notification with the creation timestamp of a message widget. Looks up the `MessageStore` attached to the app.

**Parameters:**
- `widget`: The message widget whose timestamp to display.

### `_mode_color(mode: str | None, widget_or_app=None) -> str`

Returns the hex color string for an input mode, falling back to the primary theme color.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.widgets.Static` | textual | Base static widget |
| `textual.widgets._markdown.MarkdownStream` | textual | Streaming Markdown rendering |
| `theme` | `deepagents_cli.theme` | Brand colors |
| `format_tool_display` | `deepagents_cli.tool_display` | Tool argument formatting |
| `compose_diff_lines` | `widgets.diff` | Diff rendering |
| `open_style_link` | `widgets._links` | Clickable link creation |
