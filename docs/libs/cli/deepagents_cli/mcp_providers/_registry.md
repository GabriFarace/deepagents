# `libs/cli/deepagents_cli/mcp_providers/_registry.py`

> Ordered provider registry for MCP OAuth dispatch.

## Position in the system

This file is part of the provider-specific MCP OAuth layer. `mcp_auth.py` asks the provider registry for a policy object, then uses that policy to handle server-specific login quirks without hard-coding them into the generic handshake.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.mcp_providers.base import GenericProvider, OAuthProvider`

- `from deepagents_cli.mcp_providers.github import GitHubProvider`

- `from deepagents_cli.mcp_providers.slack import SlackProvider`


## Functions and classes

### `resolve_provider(server_url: str)`

Return the provider policy that owns `server_url`.

Additional notes from the source docstring:

```text
Args:
    server_url: Remote MCP endpoint URL.

Returns:
    The first matching `OAuthProvider`; falls back to `GenericProvider`.

Raises:
    RuntimeError: If no provider matches (unreachable in practice since
        `GenericProvider.matches` always returns `True`).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
