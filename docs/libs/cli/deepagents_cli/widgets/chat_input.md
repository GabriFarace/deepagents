# `libs/cli/deepagents_cli/widgets/chat_input.py`

> Multiline input widget with history and autocomplete.

## Functions and classes

### `_default_history_path()`

Returns the prompt-history file location.

### `CompletionOption` and `CompletionPopup`

Render selectable autocomplete entries.

### `ChatTextArea`

TextArea subclass for submit/newline behavior, history, and completion keys.

### `_CompletionViewAdapter`

Adapts completion data to the popup view.

### `ChatInput`

Composite widget wrapping text area and completion popup; emits submitted text
or slash commands to the app.

## Gotchas

Key handling depends on terminal/platform shortcuts. Coordinate changes with
config helpers like `newline_shortcut()`.
