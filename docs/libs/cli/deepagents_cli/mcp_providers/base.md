# `libs/cli/deepagents_cli/mcp_providers/base.py`

> Policy interface for provider-specific MCP OAuth quirks.

## Position in the system

This file is part of the provider-specific MCP OAuth layer. `mcp_auth.py` asks the provider registry for a policy object, then uses that policy to handle server-specific login quirks without hard-coding them into the generic handshake.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from mcp.shared.auth import AnyUrl, OAuthClientMetadata`


## Functions and classes

### `LoginResult`

Outcome of a provider's pre-handshake `run_login` step.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `OAuthProvider`

Base class for provider-specific OAuth dispatch.

Additional notes from the source docstring:

```text
Subclasses override `matches` plus whichever of `client_metadata`
and `run_login` they customize. The default implementations cover
the spec-compliant Authorization Code + PKCE + Dynamic Client
Registration path.
```

Methods worth reading inside this class:

- `matches(self, server_url: str)`: Return `True` when this provider owns `server_url`.

- `client_metadata(self)`: Return the `OAuthClientMetadata` used to build the auth provider.

- `run_login(self, *, server_name: str, server_url: str, storage: FileTokenStorage)`: Perform any provider-specific pre-handshake work.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `GenericProvider`

Fallback provider for spec-compliant MCP servers with no quirks.

Methods worth reading inside this class:

- `matches(self, server_url: str)`: Match any URL — the registry places this provider last.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
