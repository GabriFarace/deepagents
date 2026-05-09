# `libs/cli/deepagents_cli/tool_display.py`

> Formatting utilities for tool call display in the CLI.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.config import MAX_ARG_LENGTH, get_glyphs`

- `from deepagents_cli.unicode_security import strip_dangerous_unicode`


## Functions and classes

### `_format_timeout(seconds: int)`

Format timeout in human-readable units (e.g., 300 -> '5m', 3600 -> '1h').

Additional notes from the source docstring:

```text
Args:
    seconds: The timeout value in seconds to format.

Returns:
    Human-readable timeout string (e.g., '5m', '1h', '300s').
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_coerce_timeout_seconds(timeout: int | str | None)`

Normalize timeout values to seconds for display.

Additional notes from the source docstring:

```text
Accepts integer values and numeric strings. Returns `None` for invalid
values so display formatting never raises.

Args:
    timeout: Raw timeout value from tool arguments.

Returns:
    Integer timeout in seconds, or `None` if unavailable/invalid.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `truncate_value(value: str, max_length: int=MAX_ARG_LENGTH)`

Truncate a string value if it exceeds max_length.

Additional notes from the source docstring:

```text
Returns:
    Truncated string with ellipsis suffix if exceeded, otherwise original.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_sanitize_display_value(value: object, *, max_length: int=MAX_ARG_LENGTH)`

Sanitize a value for safe, compact terminal display.

Additional notes from the source docstring:

```text
Hidden/deceptive Unicode controls are stripped. When stripping occurs, a
marker is appended so users know the value changed for display safety.

Args:
    value: Any value to display.
    max_length: Maximum display length before truncation.

Returns:
    Sanitized display string.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_tool_display(tool_name: str, tool_args: dict)`

Format tool calls for display with tool-specific smart formatting.

Additional notes from the source docstring:

```text
Shows the most relevant information for each tool type rather than all arguments.

Args:
    tool_name: Name of the tool being called
    tool_args: Dictionary of tool arguments

Returns:
    Formatted string for display (e.g., "(*) read_file(config.py)" in ASCII mode)

Examples:
    read_file(path="/long/path/file.py") → "<prefix> read_file(file.py)"
    web_search(query="how to code") → '<prefix> web_search("how to code")'
    execute(command="pip install foo") → '<prefix> execute("pip install foo")'
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_format_content_block(block: dict)`

Format a single content block dict for display.

Additional notes from the source docstring:

```text
Replaces large binary payloads (e.g. base64 image/video data) with a
human-readable placeholder so they don't flood the terminal.

Args:
    block: An `ImageContentBlock`, `VideoContentBlock`, or `FileContentBlock`
        dictionary.

Returns:
    A display-friendly string for the block.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_tool_message_content(content: Any)`

Convert `ToolMessage` content into a printable string.

Additional notes from the source docstring:

```text
Returns:
    Formatted string representation of the tool message content.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
