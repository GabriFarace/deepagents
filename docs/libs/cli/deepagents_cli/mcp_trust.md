# `libs/cli/deepagents_cli/mcp_trust.py`

> Trust store for project-level MCP server configurations.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.model_config import DEFAULT_CONFIG_PATH as _DEFAULT_CONFIG_PATH`


## Functions and classes

### `compute_config_fingerprint(config_paths: list[Path])`

Compute a SHA-256 fingerprint over sorted, concatenated config contents.

Additional notes from the source docstring:

```text
Args:
    config_paths: Paths to config files to fingerprint.

Returns:
    Fingerprint string in the form `sha256:<hex>`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_load_config(config_path: Path)`

Read the TOML config file.

Additional notes from the source docstring:

```text
Returns:
    Parsed TOML data, or an empty dict on failure.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_save_config(data: dict[str, Any], config_path: Path)`

Atomic write of TOML data to config_path.

Additional notes from the source docstring:

```text
Uses `tempfile.mkstemp` + `Path.replace` for crash safety.

Args:
    data: Full TOML data dict to write.
    config_path: Destination path.

Returns:
    `True` on success, `False` on I/O failure.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `is_project_mcp_trusted(project_root: str, fingerprint: str, *, config_path: Path | None=None)`

Check whether a project's MCP config is trusted with the given fingerprint.

Additional notes from the source docstring:

```text
Args:
    project_root: Absolute path to the project root.
    fingerprint: Expected fingerprint string (`sha256:<hex>`).
    config_path: Path to the trust config file.

Returns:
    `True` if the stored fingerprint matches.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `trust_project_mcp(project_root: str, fingerprint: str, *, config_path: Path | None=None)`

Persist trust for a project's MCP config.

Additional notes from the source docstring:

```text
Args:
    project_root: Absolute path to the project root.
    fingerprint: Fingerprint to store (`sha256:<hex>`).
    config_path: Path to the trust config file.

Returns:
    `True` if the entry was saved successfully.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `revoke_project_mcp_trust(project_root: str, *, config_path: Path | None=None)`

Remove trust for a project's MCP config.

Additional notes from the source docstring:

```text
Args:
    project_root: Absolute path to the project root.
    config_path: Path to the trust config file.

Returns:
    `True` if the entry was removed (or didn't exist).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
