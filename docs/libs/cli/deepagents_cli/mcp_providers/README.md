# `libs/cli/deepagents_cli/mcp_providers/`

> Provider-specific policies for remote MCP OAuth quirks.

## Position in the system

`mcp_auth.py` implements the generic OAuth and token-storage machinery. This
folder keeps provider-specific matching and pre-login behavior out of that core:
the registry returns the first provider whose `matches()` method accepts a
server URL.

## Files

- [`base.md`](./base.md) defines `OAuthProvider`, `LoginResult`, and the generic
  fallback provider.
- [`_registry.md`](./_registry.md) selects GitHub, Slack, or generic handling.
- [`github.md`](./github.md) handles GitHub Copilot MCP via Device Authorization
  Grant and pre-seeded client info.
- [`slack.md`](./slack.md) handles Slack hosted MCP with a public client ID,
  paste-back redirect URL, and optional team query parameter.

## Gotchas

Provider constants here include public OAuth client IDs, not secrets. Token
storage and browser/paste-back flow remain in `mcp_auth.py`.
