# `libs/cli/deepagents_cli/_server_config.py`

> Typed configuration for the CLI-to-server subprocess communication channel.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._constants import DEFAULT_AGENT_NAME as DEFAULT_ASSISTANT_ID`

- `from deepagents_cli._env_vars import SERVER_ENV_PREFIX`


## Functions and classes

### `_read_env_bool(suffix: str, *, default: bool=False)`

Read a `DEEPAGENTS_CLI_SERVER_*` boolean from the environment.

Additional notes from the source docstring:

```text
Boolean env vars use the `'true'` / `'false'` convention (case insensitive).
Missing variables fall back to *default*.

Args:
    suffix: Variable name suffix after the `DEEPAGENTS_CLI_SERVER_` prefix.
    default: Value when the variable is absent.

Returns:
    Parsed boolean.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_read_env_json(suffix: str)`

Read a JSON-encoded `DEEPAGENTS_CLI_SERVER_*` variable.

Additional notes from the source docstring:

```text
Args:
    suffix: Variable name suffix after the `DEEPAGENTS_CLI_SERVER_` prefix.

Returns:
    Parsed JSON value, or `None` if the variable is absent.

Raises:
    ValueError: If the variable is present but not valid JSON.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_read_env_str(suffix: str)`

Read an optional `DEEPAGENTS_CLI_SERVER_*` string variable.

Additional notes from the source docstring:

```text
Args:
    suffix: Variable name suffix after the `DEEPAGENTS_CLI_SERVER_` prefix.

Returns:
    The string value, or `None` if absent.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_read_env_optional_bool(suffix: str)`

Read a tri-state `DEEPAGENTS_CLI_SERVER_*` boolean (`True` / `False` / `None`).

Additional notes from the source docstring:

```text
Used for settings where `None` carries a distinct meaning (e.g. "not
specified, use default logic").

Args:
    suffix: Variable name suffix after the `DEEPAGENTS_CLI_SERVER_` prefix.

Returns:
    `True`, `False`, or `None` when the variable is absent.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `ServerConfig`

Full configuration payload passed from the CLI to the server subprocess.

Additional notes from the source docstring:

```text
Serialized to/from `DEEPAGENTS_CLI_SERVER_*` environment variables so
that the server
graph (which runs in a separate Python interpreter) can reconstruct the
CLI's intent without sharing memory.
```

Methods worth reading inside this class:

- `__post_init__(self)`: Normalize fields and validate invariants.

- `to_env(self)`: Serialize this config to a `DEEPAGENTS_CLI_SERVER_*` env-var mapping.

- `from_env(cls)`: Reconstruct a `ServerConfig` from `DEEPAGENTS_CLI_SERVER_*` env vars.

- `from_cli_args(cls, *, project_context: ProjectContext | None, model_name: str | None, model_params: dict[str, Any] | None, assistant_id: str, auto_approve: bool, interrupt_shell_only: bool=False, shell_allow_list: list[str] | None=None, sandbox_type: str='none', sandbox_id: str | None, sandbox_setup: str | None, enable_shell: bool, enable_ask_user: bool, mcp_config_path: str | None, no_mcp: bool, trust_project_mcp: bool | None, interactive: bool)`: Build a `ServerConfig` from parsed CLI arguments.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_normalize_path(raw_path: str | None, project_context: ProjectContext | None, label: str)`

Resolve a possibly-relative path to absolute.

Additional notes from the source docstring:

```text
The server subprocess runs in a different working directory, so relative
paths must be resolved against the user's original cwd before serialization.

Args:
    raw_path: Path from CLI arguments (may be relative).
    project_context: User/project context for path resolution.
    label: Human-readable label for error messages (e.g. "MCP config").

Returns:
    Absolute path string, or `None` when *raw_path* is `None` or empty.

Raises:
    ValueError: If the path cannot be resolved.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
