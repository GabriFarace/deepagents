# `libs/cli/deepagents_cli/iterm_cursor_guide.py`

> iTerm2 cursor guide workaround for Textual alternate-screen rendering.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `_write_iterm_escape(sequence: str)`

Write an iTerm2 escape sequence to stderr.

Additional notes from the source docstring:

```text
Silently fails if the terminal is unavailable (redirected, closed, broken
pipe). This is a cosmetic feature, so failures should never crash the app.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_plist_bool(value: object)`

Return a plist boolean/int value as `bool`, or `None` if not boolean-like.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_profile_uses_cursor_guide(profile: dict[str, object])`

Return whether an iTerm2 profile has cursor guide enabled.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_coerce_profile(raw: object)`

Return a string-keyed profile dictionary from raw plist data.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_find_iterm_profile(profiles: list[object], *, name: str, guid: str)`

Find the current iTerm2 profile by name, then by default profile GUID.

Additional notes from the source docstring:

```text
Args:
    profiles: Profile entries from iTerm2 preferences.
    name: Active profile name from `ITERM_PROFILE`.
    guid: Default profile GUID from iTerm2 preferences.

Returns:
    The matching profile dictionary, or `None` when no match is found.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_iterm_profile_cursor_guide_enabled()`

Infer whether iTerm2 cursor guide was enabled before CLI startup.

Additional notes from the source docstring:

```text
iTerm2's OSC 1337 `HighlightCursorLine` command can set the guide to yes/no
but does not report the current state. The best cheap signal available at
startup is the active profile preference, exposed in the iTerm2 plist. The
`ITERM_PROFILE` environment variable is set by iTerm2; when it is missing,
fall back to the default profile GUID in preferences.

Returns:
    `True` if the matched iTerm2 profile has cursor guide enabled.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `restore_iterm_cursor_guide()`

Restore iTerm2 cursor guide when launch-time profile state required it.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_disable_iterm_cursor_guide()`

Disable iTerm2 cursor guide only when the module has a restore path.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
