# `libs/deepagents/deepagents/` — Core SDK

The `deepagents` SDK provides `create_deep_agent()` — a single factory function that assembles a fully configured LangGraph agent from composable pieces. Everything else in this package (backends, middleware, models) supports that factory.

---

## File Map

| File | Role |
|---|---|
| [`graph.py`](graph.md) | `create_deep_agent()` — the public API entry point |
| [`_models.py`](_models.md) | Model string → `BaseChatModel` resolution |
| [`_tools.py`](graph.md) | Built-in tool helpers (used by graph.py) |
| [`_excluded_middleware.py`](graph.md) | Validation of `exclude_middleware` parameter |
| [`backends/`](backends/README.md) | Storage & execution backends (filesystem, state, sandbox, …) |
| [`middleware/`](middleware/README.md) | Agent middleware (tools, skills, memory, HITL, summarization, …) |
| [`profiles/`](graph.md) | Model-specific tuning profiles (OpenAI Responses API, etc.) |
| [`__init__.py`](graph.md) | Public exports |

---

## The Three-Layer Model

```
create_deep_agent()
        │
        ├─── Backend   — WHERE files and shell commands go
        │    └─ BackendProtocol: ls, read, write, edit, glob, grep, execute
        │
        ├─── Middleware — WHAT the agent can do (wraps every model call)
        │    └─ Stack of AgentMiddleware: injects tools, system prompt, etc.
        │
        └─── Model     — WHICH LLM makes decisions
             └─ Any LangChain BaseChatModel
```

Separating these three concerns means you can swap any one without touching the others: change the backend (local → cloud sandbox), change the middleware (add a new tool), or change the model (Claude → GPT-4o) independently.

---

## Quick Start

```python
from deepagents import create_deep_agent

graph = create_deep_agent()  # defaults: Claude Sonnet 4.6, in-memory backend, standard middleware
result = graph.invoke({"messages": [{"role": "user", "content": "List files here"}]})
```

See [`graph.md`](graph.md) for the full parameter reference.

---

## See Also

- [graph.md](graph.md) — full `create_deep_agent()` API
- [backends/README.md](backends/README.md) — backend selection guide
- [middleware/README.md](middleware/README.md) — middleware stack reference
