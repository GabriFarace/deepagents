# `libs/cli/deepagents_cli/ui.py`

> Help screens and argparse utilities for the CLI.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from rich.markup import escape`

- `from deepagents_cli import theme`

- `from deepagents_cli._version import DOCS_URL, __version__`

- `from deepagents_cli.config import _get_editable_install_path, _is_editable_install, console`


## Functions and classes

### `positive_int(value: str)`

Argparse type for integer arguments that must be >= 1.

Additional notes from the source docstring:

```text
Args:
    value: Raw CLI argument string to parse.

Returns:
    Parsed positive integer.

Raises:
    argparse.ArgumentTypeError: If `value` is not an integer or is < 1.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_print_option_section(*lines: str, title: str='Options')`

Print a help-screen options section with shared JSON/help flags.

Additional notes from the source docstring:

```text
Args:
    *lines: Command-specific option lines to print before the shared flags.
    title: Section title to display.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_help()`

Show top-level help information for the deepagents CLI.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_list_help()`

Show help information for the `list` subcommand.

Additional notes from the source docstring:

```text
Invoked via the `-h` argparse action or directly from `cli_main`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_agents_help()`

Show help information for the `agents` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_reset_help()`

Show help information for the `reset` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_skills_help()`

Show help information for the `skills` subcommand.

Additional notes from the source docstring:

```text
Invoked via the `-h` argparse action or directly from
`execute_skills_command` when no subcommand is given.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_skills_list_help()`

Show help information for the `skills list` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_skills_create_help()`

Show help information for the `skills create` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_skills_info_help()`

Show help information for the `skills info` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_skills_delete_help()`

Show help information for the `skills delete` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_update_help()`

Show help information for the `update` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_print_mcp_discovery_paths()`

Print the auto-discovered MCP config paths in precedence order.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_mcp_help()`

Show help information for the `mcp` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_mcp_login_help()`

Show help information for the `mcp login` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_threads_help()`

Show help information for the `threads` subcommand.

Additional notes from the source docstring:

```text
Invoked via the `-h` argparse action or directly from `cli_main`
when no threads subcommand is given.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_threads_delete_help()`

Show help information for the `threads delete` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `show_threads_list_help()`

Show help information for the `threads list` subcommand.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
