# `skills/__init__.py`

## High-Level Purpose

This is the public API entry point for the `skills` subpackage. It exports only the two functions needed for CLI integration: one for executing skill subcommands and one for setting up the argparse configuration.

## Public API

| Export | Source | Description |
|---|---|---|
| `execute_skills_command` | `skills.commands` | Execute a skills subcommand (list/create/info/delete) |
| `setup_skills_parser` | `skills.commands` | Configure argparse subparsers for skills commands |

## Usage

```python
from deepagents_cli.skills import execute_skills_command, setup_skills_parser
```

All other components in the package are internal implementation details not intended for direct import.
