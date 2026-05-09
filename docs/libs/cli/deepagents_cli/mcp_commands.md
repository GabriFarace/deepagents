# `libs/cli/deepagents_cli/mcp_commands.py`

> CLI commands for `deepagents mcp`.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `_lazy_ui_help(fn_name: str)`

Return a callable that lazily imports and invokes a `ui` help function.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `setup_mcp_parsers(subparsers: Any, *, make_help_action: Callable[[Callable[[], None]], type[argparse.Action]])`

Register the `deepagents mcp` command group.

Additional notes from the source docstring:

```text
Args:
    subparsers: The `argparse` subparsers object from the top-level CLI
        parser, onto which the `mcp` command group is attached.
    make_help_action: Factory that wraps a `show_*` callable into an
        `argparse.Action` so `-h/--help` renders the hand-maintained
        help screens from `deepagents_cli.ui` instead of argparse's
        auto-generated text.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `run_mcp_login(*, server: str, config_path: str | None)`

Handle `deepagents mcp login <server>`.

Additional notes from the source docstring:

```text
When `config_path` is omitted, auto-discovered MCP configs are merged in
the same precedence order as the runtime loader, with matching trust
gating: user-level configs are always included, but project-level configs
are only included when the trust store has a fingerprint match. An
untrusted project-level config (for example, a `.mcp.json` in a cloned
repo) is skipped so attacker-controlled `headers` entries cannot exfiltrate
local secrets during the OAuth handshake. When `config_path` is set, that
file alone is loaded and treated as explicitly trusted.

Args:
    server: Target server name from `mcpServers`.
    config_path: Optional explicit MCP config path.

Returns:
    Process exit code: 0 on success, 1 on config or login failure,
    2 if no config file could be found.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
