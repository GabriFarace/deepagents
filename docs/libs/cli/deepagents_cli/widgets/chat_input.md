# `deepagents_cli/widgets/chat_input.py`

## High-Level Purpose

`ChatInput` is the text input widget at the bottom of the TUI. It is a multiline editable area that supports slash-command autocomplete, `@file` mention highlighting, input history navigation (Up/Down arrows), and pasting media (images, PDFs). When the user presses Enter, it emits an `InputSubmitted` message to `CLIApp`.

---

## Key Class

### `ChatInput(Widget)`

**Key behaviors:**

#### Slash-command autocomplete

When the user types `/`, an `AutocompleteOverlay` floats above the input, showing matching slash commands. It is populated by:
1. Static entries from `command_registry.COMMANDS`
2. Dynamically discovered skill entries: `/skill:web-research`, `/skill:code-review`, etc.

Autocomplete uses fuzzy matching (not prefix-only). Selecting an entry with Tab or Enter completes the command name.

#### `@file` mention highlighting

As the user types, the widget applies inline Rich markup to `@path/to/file` patterns — highlighting them in a distinct color if the path resolves, or dimmed if it doesn't exist. This is cosmetic only; the actual file content is attached in `input.py` when the message is submitted.

#### Input history

Up/Down arrow keys navigate through the history of submitted messages (stored in a ring buffer in `HistoryBrowser`). Pressing Up replaces the current text with the previous input. Pressing Down moves forward.

#### Media paste

When the user pastes a file path, image data, or drags a file onto the terminal (via OSC 52 or terminal drag-and-drop), `ChatInput` detects it and calls `MediaTracker.add()`. The media is attached to the message when submitted.

#### Submit

Enter (on a non-empty line that isn't part of a multiline paste) emits:
```python
class InputSubmitted(Message):
    text: str
    media: list[MediaItem]
```

Shift+Enter inserts a newline for multiline input.

#### Disabled state

While the agent is running, `ChatInput.disabled = True`. The widget shows a "thinking..." placeholder and rejects keyboard input. Re-enabled when the agent finishes.

---

## Architecture Notes

**Autocomplete overlay:** The `AutocompleteOverlay` is a separate widget mounted as a child of `ChatInput`. It has `absolute` CSS positioning and floats visually above the input. It is unmounted when the user submits or presses Escape.

**History persistence:** The input history ring buffer is stored in memory only (not persisted to disk). It resets when the CLI exits.

---

## See Also

- [app.md](../app.md) — receives `InputSubmitted` and routes it
- [command_registry.md](../command_registry.md) — provides autocomplete entries
- [input.md](../input.md) — processes `@file` mentions and media after submit
