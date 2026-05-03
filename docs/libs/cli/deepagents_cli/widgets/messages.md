# `deepagents_cli/widgets/messages.py`

## High-Level Purpose

`messages.py` contains all the widget classes used to display conversation turns in the scrollable transcript. Each message type is a distinct widget so Textual can efficiently update only the changed portion of the screen — appending tokens to an `AssistantMessage` doesn't re-render `UserMessage` widgets above it.

---

## Message Widget Classes

### `UserMessage(Widget)`

Displays the user's input. Features:
- `@file` mentions are highlighted in a distinct color
- `/command` text is styled to stand out
- Multi-line messages render correctly
- Shows a timestamp on hover

### `AssistantMessage(Widget)`

Displays the agent's text response. Features:
- **Streaming:** tokens are appended progressively as they arrive from the SSE stream. The widget calls `self.refresh()` after each append, producing the "typewriter" effect.
- **Markdown rendering:** the final response is rendered as Rich Markdown (code blocks, bold, links, tables)
- **Copy:** keyboard shortcut copies the full message text to clipboard

### `ToolCallMessage(Widget)`

Displays a tool invocation and its result as a collapsible pair. Layout:

```
▶ execute("ls -la /home")          [status: completed ✓]
  └─ Result:
     total 48
     drwxr-xr-x 12 user ...
```

States:
- `PENDING` — tool called, result not yet received (shows spinner)
- `APPROVED` — user approved (for HITL tools)
- `REJECTED` — user rejected
- `COMPLETED` — result received
- `ERROR` — tool raised an exception

Tool arguments are pretty-printed JSON. Results are truncated if too long (with a "show more" expander).

### `DiffMessage(Widget)`

Displays a file edit as a colored unified diff. Shown when `write_file` or `edit_file` produces a diff. Uses Rich's syntax highlighting for diff output.

### `ErrorMessage(Widget)`

Displays an error (network failure, server crash, tool exception) in a red-bordered box. Includes the error type and message. Optionally shows a stack trace (hidden by default, expandable).

---

## Message Widget Lifecycle

1. `StreamHandler` produces `UIAction` objects
2. `CLIApp` applies actions:
   - `CreateToolCallAction` → `mount(ToolCallMessage(...))`
   - `AppendTextAction` → `assistant_msg.append_text(chunk)`
   - etc.
3. Each widget renders itself via Textual's reactive model
4. `VerticalScroll` auto-scrolls to the new widget

---

## Architecture Notes

**Widget identity:** Each `ToolCallMessage` has a `tool_call_id` field. When the server sends a `ToolMessage` result, `CLIApp` looks up the corresponding `ToolCallMessage` by ID and updates it in place — rather than creating a new widget.

**Markdown rendering timing:** `AssistantMessage` renders as plain text during streaming (for speed) and re-renders as Markdown once the stream ends. This avoids partial Markdown glitches during streaming (e.g., an unclosed `**`).

---

## See Also

- [message_store.md](message_store.md) — tracks all message widgets by ID
- [remote_client.md](../remote_client.md) — produces `UIAction`s that create/update these widgets
- [approval.md](approval.md) — `ToolCallMessage` with HITL status shows the approval flow
