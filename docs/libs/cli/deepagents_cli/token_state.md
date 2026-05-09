# `libs/cli/deepagents_cli/token_state.py`

> Middleware that adds a `_context_tokens` channel to the graph state.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from langchain.agents.middleware.types import AgentMiddleware, AgentState, PrivateStateAttr`


## Functions and classes

### `TokenTrackingState`

Extends agent state with a persisted context-token counter.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `TokenStateMiddleware`

Schema-only middleware that registers `_context_tokens` in the state schema.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
