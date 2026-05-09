# `libs/deepagents/`

> Core SDK package. This is where `create_deep_agent()` is assembled and where
> all provider-agnostic filesystem, backend, middleware, and profile behavior
> lives.

## Position in the system

The SDK is the only package other parts of the monorepo are expected to build
on directly.

```
caller
  │
  ▼
deepagents.create_deep_agent(...)
  │
  ├─ resolves model/provider profile
  ├─ installs backend-backed tools
  ├─ installs middleware
  └─ returns LangGraph CompiledStateGraph
```

The CLI does not embed the SDK directly in its TUI process. It starts a
LangGraph server whose graph factory imports this package, then the TUI talks
to that server. Partner packages depend on the SDK in the opposite direction:
they implement `BackendProtocol` so the SDK's filesystem tools can run against
cloud or in-process sandboxes.

## Contents

| Path | Doc | Role |
|---|---|---|
| `deepagents/graph.py` | [`deepagents/graph.md`](./deepagents/graph.md) | Main factory and default middleware stack. |
| `deepagents/_models.py` | [`deepagents/_models.md`](./deepagents/_models.md) | Model resolution and model identity helpers. |
| `deepagents/_tools.py` | [`deepagents/_tools.md`](./deepagents/_tools.md) | Tool-name inspection and harness description overrides. |
| `deepagents/backends/` | [`deepagents/backends/`](./deepagents/backends/README.md) | Storage and shell execution abstraction. |
| `deepagents/middleware/` | [`deepagents/middleware/`](./deepagents/middleware/README.md) | Agent behavior: filesystem, subagents, skills, memory, permissions, summarization. |
| `deepagents/profiles/` | [`deepagents/profiles/`](./deepagents/profiles/README.md) | Model and provider profiles. |
| `deepagents/_api/` | [`deepagents/_api/`](./deepagents/_api/README.md) | Internal API helpers such as deprecation warnings. |

## Gotchas

The package deliberately keeps most behavior out of `graph.py`. `graph.py`
chooses and orders components; the actual tool implementations, permission
checks, prompt addenda, and sub-agent execution rules live in middleware and
backend modules.
