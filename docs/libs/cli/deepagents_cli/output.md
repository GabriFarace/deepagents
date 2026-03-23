# `output.py`

## High-Level Purpose

This module provides machine-readable JSON output helpers for CLI subcommands. It is intentionally kept **stdlib-only** (no third-party dependencies) so it can be imported from CLI startup paths without pulling in heavy dependency trees.

## Type Aliases

| Alias | Type | Description |
|---|---|---|
| `OutputFormat` | `Literal["text", "json"]` | Accepted output modes for CLI subcommands |

## Functions

### `add_json_output_arg(parser: argparse.ArgumentParser, *, default: OutputFormat | None = None) -> None`

Adds a `--json` flag to an `argparse` parser. When `--json` is passed, `args.output_format` is set to `"json"`.

**Parameters:**
- `parser`: The argparse parser to update.
- `default`: Default output format. Pass `None` for subparsers (preserves parent value). Pass `"text"` for root parsers (sets explicit default).

**Key Logic:** When `default` is `None`, uses `argparse.SUPPRESS` to avoid overriding a parent parser's value. When a default is provided, it becomes the explicit default for the parser.

### `write_json(command: str, data: list | dict) -> None`

Writes a JSON envelope to stdout and flushes.

**Parameters:**
- `command`: Self-documenting command name (e.g., `'list'`, `'threads list'`).
- `data`: Payload — typically a list for listing commands, dict for action/info commands.

**Output format:**
```json
{"schema_version": 1, "command": "list", "data": [...]}
```

**Key Details:**
- Uses `default=str` so `Path` and `datetime` objects serialize without error.
- Writes a single-line JSON string followed by `\n`.
- Flushes stdout immediately.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `argparse` | stdlib | Argument parser integration |
| `json` | stdlib | JSON serialization |
| `sys` | stdlib | stdout writing |
