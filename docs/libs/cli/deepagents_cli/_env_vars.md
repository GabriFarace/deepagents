# `libs/cli/deepagents_cli/_env_vars.py`

> Canonical registry of `DEEPAGENTS_CLI_*` environment variables.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `is_env_truthy(name: str, *, default: bool=False)`

Return whether env var *name* is set to a recognizably truthy value.

Additional notes from the source docstring:

```text
Unlike `bool(os.environ.get(name))`, this does not treat `"0"` or
`"false"` as enabled. Use this for on/off flags where the user would
reasonably expect `VAR=0` to mean "disabled".

Args:
    name: Environment variable name (typically a `DEEPAGENTS_CLI_*`
        constant from this module).
    default: Value returned when the variable is unset OR set to a
        value that is neither recognizably truthy nor falsy.

Returns:
    `True` for `1`/`true`/`yes`/`on` (case-insensitive), `False` for
    `0`/`false`/`no`/`off`/empty string, or `default` otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
