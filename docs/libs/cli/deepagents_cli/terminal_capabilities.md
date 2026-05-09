# `libs/cli/deepagents_cli/terminal_capabilities.py`

> Terminal capability detection.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._env_vars import KITTY_KEYBOARD`


## Functions and classes

### `_override_supports_kitty_keyboard_protocol(env: Mapping[str, str])`

Return an explicit kitty-keyboard override from `env`, if present.

Additional notes from the source docstring:

```text
Accepted truthy values are `'1'`, `'true'`, `'yes'`, and `'on'`.
Accepted falsy values are `'0'`, `'false'`, `'no'`, and `'off'`.
`'auto'`, the empty string, and invalid values fall back to heuristic
detection.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_terminal_identity_supports_kitty_keyboard_protocol(env: Mapping[str, str])`

Return whether `env` identifies a terminal with built-in kitty support.

Additional notes from the source docstring:

```text
This intentionally only recognizes terminals whose environment markers
imply kitty-keyboard support is part of the terminal's default identity.
Configurable terminals such as iTerm2 and WezTerm are intentionally not
auto-detected because protocol support can be disabled in user settings.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `supports_kitty_keyboard_protocol()`

Return whether the attached terminal should be treated as kitty-aware.

Additional notes from the source docstring:

```text
Detection is side-effect free: it never writes escape sequences or reads
queued input bytes. That means it may under-detect some configurable
terminals, but it will not interfere with Textual's input stream.

Set `DEEPAGENTS_CLI_KITTY_KEYBOARD` to an accepted truthy value (`1`,
`true`, `yes`, `on`) to force-enable the label, a falsy value (`0`,
`false`, `no`, `off`) to force-disable it, or `auto`/unset to use
heuristic detection.

Returns:
    `True` when the terminal is known to support the kitty keyboard
    protocol, `False` otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
