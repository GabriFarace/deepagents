# `libs/cli/deepagents_cli/mcp_providers/__init__.py`

> Provider-specific MCP OAuth dispatch.

## Position in the system

This file is part of the provider-specific MCP OAuth layer. `mcp_auth.py` asks the provider registry for a policy object, then uses that policy to handle server-specific login quirks without hard-coding them into the generic handshake.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.mcp_providers._registry import resolve_provider`

- `from deepagents_cli.mcp_providers.base import GenericProvider, LoginResult, OAuthProvider`

- `from deepagents_cli.mcp_providers.github import GitHubProvider`

- `from deepagents_cli.mcp_providers.slack import SlackProvider`


## Functions and classes

This module has no public functions or classes. It exists for package discovery, typing markers, constants, side-effect imports, or re-export behavior described above.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
