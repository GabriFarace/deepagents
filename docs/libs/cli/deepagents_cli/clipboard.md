# `libs/cli/deepagents_cli/clipboard.py`

> Clipboard utilities for deepagents-cli.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.config import get_glyphs`


## Functions and classes

### `_copy_osc52(text: str)`

Copy text using OSC 52 escape sequence (works over SSH/tmux).

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_shorten_preview(texts: list[str])`

Shorten text for notification preview.

Additional notes from the source docstring:

```text
Returns:
    Shortened preview text suitable for notification display.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `copy_selection_to_clipboard(app: App)`

Copy selected text from app widgets to clipboard.

Additional notes from the source docstring:

```text
This queries all widgets for their text_selection and copies
any selected text to the system clipboard.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
