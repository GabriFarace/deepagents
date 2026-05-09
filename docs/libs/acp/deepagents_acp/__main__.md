# `acp/deepagents_acp/__main__.py`

> Entry point for running the ACP server as a module.

## Position in the system

This package sits at the boundary between deepagents and ACP clients. It imports the SDK graph/backends and ACP schema objects, then translates protocol calls into LangGraph invocations and session updates.

## Imports and module-level state

This file imports `asyncio, deepagents_acp.server`.

## Functions and classes

### `main()`

Run the test ACP agent server. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `run`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
