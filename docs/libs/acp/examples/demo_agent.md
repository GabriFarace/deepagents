# `acp/examples/demo_agent.py`

> Demo coding agent using ACP.

## Position in the system

This package sits at the boundary between deepagents and ACP clients. It imports the SDK graph/backends and ACP schema objects, then translates protocol calls into LangGraph invocations and session updates.

## Imports and module-level state

This file imports `asyncio, os, acp, acp.schema, deepagents, deepagents.backends, dotenv, langgraph.checkpoint.memory, langgraph.graph.state, deepagents_acp.server, examples.local_context`.

## Functions and classes

### `main()`

Run the demo agent. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `run`. It runs synchronously in the caller and returns directly.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
