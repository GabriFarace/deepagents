# `libs/cli/deepagents_cli/mcp_providers/github.py`

> GitHub-hosted MCP OAuth provider.

## Position in the system

This file is part of the provider-specific MCP OAuth layer. `mcp_auth.py` asks the provider registry for a policy object, then uses that policy to handle server-specific login quirks without hard-coding them into the generic handshake.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from mcp.shared.auth import AnyUrl, OAuthClientInformationFull`

- `from deepagents_cli.mcp_auth import _run_device_flow`

- `from deepagents_cli.mcp_providers.base import LoginResult, OAuthProvider`


## Functions and classes

### `_is_github_mcp_url(url: str)`

Return `True` when `url` points at GitHub's remote MCP endpoint.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_preseed_github_auth(storage: FileTokenStorage)`

Run GitHub Device Flow and persist the token and stub client info.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `GitHubProvider`

GitHub-hosted MCP: RFC 8628 Device Authorization Grant.

Methods worth reading inside this class:

- `matches(self, server_url: str)`: Match `api.githubcopilot.com`.

- `run_login(self, *, server_name: str, server_url: str, storage: FileTokenStorage)`: Run the device flow and short-circuit the Authorization Code handshake.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
