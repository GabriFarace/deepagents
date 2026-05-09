# `libs/cli/deepagents_cli/output.py`

> Machine-readable JSON output helpers for CLI subcommands.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `add_json_output_arg(parser: argparse.ArgumentParser, *, default: OutputFormat | None=None)`

Add a `--json` flag to an argparse parser.

Additional notes from the source docstring:

```text
Args:
    parser: Parser to update.
    default: Default output format for this parser.

        Pass `None` for subparsers so parent parser values are preserved.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `write_json(command: str, data: list | dict)`

Write a JSON envelope to stdout and flush.

Additional notes from the source docstring:

```text
The envelope is a single-line JSON object with a stable schema:

```json
{"schema_version": 1, "command": "...", "data": ...}
```

Args:
    command: Self-documenting command name (e.g. `'list'`,
        `'threads list'`).
    data: Payload — typically a list for listing commands or a dict
        for action/info commands.

        `default=str` is used so that `Path` and `datetime` objects
        serialize without error.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
