# `__main__.py`

## High-Level Purpose

This module is the entry point that enables running the CLI as a Python module with `python -m deepagents_cli`. It imports and calls `cli_main` from `main.py`, enabling the standard module-invocation pattern.

## Key Logic

When invoked via `python -m deepagents_cli`, Python executes this file. The `if __name__ == "__main__"` guard ensures `cli_main()` is called only during direct execution, not on import.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `cli_main` | `deepagents_cli.main` | The main CLI entry point function |

## Usage

```bash
python -m deepagents_cli
# or
python -m deepagents_cli --help
```
