# `widgets/autocomplete.py`

## High-Level Purpose

This module implements the autocomplete system for the chat input. It provides trigger-based completion for two contexts:

1. **Slash commands** (`/`) — complete available CLI slash commands
2. **File mentions** (`@`) — fuzzy file name completion using `git ls-files` or directory traversal

The system is built around a `Protocol`-based architecture that separates the completion logic (controllers) from the display (views) and from managing multiple controllers simultaneously (`MultiCompletionManager`).

## Classes

### `CompletionResult`

**Type:** `StrEnum`

Result returned from key event handlers in the completion system.

| Value | Description |
|---|---|
| `IGNORED` | Key not handled, let default behavior proceed |
| `HANDLED` | Key handled, prevent default |
| `SUBMIT` | Key triggers submission (e.g., Enter on slash command completion) |

### `CompletionView` (Protocol)

Interface that views must implement to display completion suggestions.

**Methods:**
- `render_completion_suggestions(suggestions: list[tuple[str, str]], selected_index: int) -> None` — Render the popup with `(label, description)` tuples.
- `clear_completion_suggestions() -> None` — Hide/clear the popup.
- `replace_completion_range(start: int, end: int, replacement: str) -> None` — Replace text in the input.

### `CompletionController` (Protocol)

Interface for completion logic controllers.

**Methods:**
- `can_handle(text: str, cursor_index: int) -> bool` — Check if this controller handles the current input state.
- `on_text_changed(text: str, cursor_index: int) -> None` — Update suggestions when text changes.
- `on_key_up() -> CompletionResult` — Handle up arrow.
- `on_key_down() -> CompletionResult` — Handle down arrow.
- `on_key_tab() -> CompletionResult` — Handle Tab key.
- `on_key_enter() -> CompletionResult` — Handle Enter key.
- `on_key_escape() -> CompletionResult` — Handle Escape key.
- `is_active() -> bool` — Whether the controller is currently showing suggestions.

### `SlashCommandController`

Handles `/command` autocompletion. Triggers when the input starts with `/` and the cursor is within the command token.

**Parameters:**
- `view`: The `CompletionView` to render into.
- `commands`: Dict of command names to descriptions.

**Behavior:**
- Shows a filtered list of slash commands that match the typed prefix.
- Arrow keys and Tab navigate the list.
- Enter or Tab on a selection replaces the `/...` prefix with the chosen command.

### `FuzzyFileController`

Handles `@filename` autocompletion. Triggers when `@` is typed.

**Parameters:**
- `view`: The `CompletionView` to render into.
- `project_root`: Optional project root for `git ls-files`.

**Behavior:**
- Uses `git ls-files` (if available) to get a list of tracked files.
- Falls back to directory traversal if `git` is unavailable.
- Fuzzy-matches the typed text after `@` against the file list using `difflib.SequenceMatcher`.
- Arrow keys and Tab navigate; Enter/Tab completes the `@mention`.

### `MultiCompletionManager`

Manages multiple `CompletionController` instances. Delegates key events and text changes to the currently active controller.

**Constructor:**
```python
MultiCompletionManager(controllers: list[CompletionController])
```

**Key Methods:**
- `on_text_changed(text, cursor_index) -> None` — Notifies all controllers; activates the one that `can_handle`.
- `on_key_*(…) -> CompletionResult` — Dispatches key events to the active controller.
- `is_active() -> bool` — Whether any controller is showing suggestions.

## Module-Level Functions

### `_get_git_executable() -> str | None`

Returns the full path to the `git` executable via `shutil.which`, or `None` if not found.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `difflib.SequenceMatcher` | stdlib | Fuzzy file matching |
| `subprocess` | stdlib | `git ls-files` execution |
| `find_project_root` | `deepagents_cli.project_utils` | Project root detection |
