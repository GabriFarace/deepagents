# `libs/cli/deepagents_cli/formatting.py`

> Lightweight text-formatting helpers.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `format_duration(seconds: float)`

Format a duration in seconds into a human-readable string.

Additional notes from the source docstring:

```text
Args:
    seconds: Duration in seconds.

Returns:
    Formatted string like `"5s"`, `"2.3s"`, `"5m 12s"`, or `"1h 23m 4s"`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
