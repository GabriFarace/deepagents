# `libs/cli/deepagents_cli/editor.py`

> External editor support for composing prompts.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `resolve_editor()`

Resolve editor command from environment.

Additional notes from the source docstring:

```text
Checks $VISUAL, then $EDITOR, then falls back to platform default.

Returns:
    Tokenized command list, or `None` if the env var was set but empty after
        tokenization.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_prepare_command(cmd: list[str], filepath: str)`

Build the full command list with appropriate flags.

Additional notes from the source docstring:

```text
Adds --wait/-w for GUI editors and `-i NONE` for vim-family editors.

Returns:
    The complete command list with flags and filepath appended.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `open_in_editor(current_text: str)`

Open current_text in an external editor.

Additional notes from the source docstring:

```text
Creates a temp .md file, launches the editor, and reads back the result.

Args:
    current_text: The text to pre-populate in the editor.

Returns:
    The edited text with normalized line endings, or `None` if the editor
        exited with a non-zero status, was not found, or the result was
        empty/whitespace-only.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
