# `libs/cli/deepagents_cli/mcp_providers/slack.py`

> Slack-hosted MCP OAuth provider.

## Position in the system

This file is part of the provider-specific MCP OAuth layer. `mcp_auth.py` asks the provider registry for a policy object, then uses that policy to handle server-specific login quirks without hard-coding them into the generic handshake.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from mcp.shared.auth import AnyUrl, OAuthClientInformationFull, OAuthClientMetadata`

- `from deepagents_cli.mcp_providers.base import LoginResult, OAuthProvider`


## Functions and classes

### `_is_slack_mcp_url(url: str)`

Return `True` when `url` points at a Slack-hosted MCP endpoint.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_prompt_slack_team()`

Interactively ask the user which Slack workspace to install into.

Additional notes from the source docstring:

```text
Runs the blocking `input()` in a worker thread so `login()` stays safe
to await from an already-running event loop (Textual worker, IPython).

Returns:
    The entered Slack team ID, or `None` if the prompt was left blank.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_preseed_slack_client_info(storage: FileTokenStorage)`

Write the hardcoded Slack `client_info` to `storage` if not already set.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `SlackProvider`

Slack-hosted MCP: paste-back Authorization Code with a public client.

Methods worth reading inside this class:

- `matches(self, server_url: str)`: Match `slack.com` and any `*.slack.com` subdomain.

- `client_metadata(self)`: Return public-client metadata with the Slack loopback redirect URI.

- `run_login(self, *, server_name: str, server_url: str, storage: FileTokenStorage)`: Preseed client info and optionally thread the team ID into auth URL.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
