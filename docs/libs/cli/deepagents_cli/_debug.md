# `libs/cli/deepagents_cli/_debug.py`

> Shared debug-logging configuration for verbose file-based tracing.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._env_vars import DEBUG, DEBUG_FILE, is_env_truthy`


## Functions and classes

### `configure_debug_logging(target: logging.Logger)`

Attach a file handler to *target* when `DEEPAGENTS_CLI_DEBUG` is set.

Additional notes from the source docstring:

```text
The log file defaults to `'/tmp/deepagents_debug.log'` but can be overridden
with `DEEPAGENTS_CLI_DEBUG_FILE`. The handler appends so that multiple
modules share the same log file across a session.

Does nothing when `DEEPAGENTS_CLI_DEBUG` is not truthy (see `is_env_truthy`).

Args:
    target: Logger to configure.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
