# `libs/cli/deepagents_cli/__init__.py`

> Deep Agents CLI - Interactive AI coding assistant.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._version import __version__`


## Functions and classes

### `__getattr__(name: str)`

Lazy import for `cli_main` to avoid loading `main.py` at package import.

Additional notes from the source docstring:

```text
`main.py` pulls in `argparse`, signal handling, and other startup machinery
that isn't needed when submodules like `config` or `widgets` are
imported directly.

Returns:
    The requested callable.

Raises:
    AttributeError: If *name* is not a lazily-provided attribute.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
