# `deepagents/backends/store.py`

## High-Level Purpose

`StoreBackend` stores files in a LangGraph `BaseStore` — a persistent key-value store that survives across multiple threads. Unlike `StateBackend` (which is per-thread) or `FilesystemBackend` (which is per-machine disk), `StoreBackend` enables cross-thread and cross-session file sharing. Useful for agents that maintain shared notes, a knowledge base, or per-user data.

---

## Key Class

### `StoreBackend`

Implements `BackendProtocol` (no `execute`).

**Constructor parameters:**

| Parameter | Type | Description |
|---|---|---|
| `store` | `BaseStore` | LangGraph store instance |
| `namespace` | `tuple[str, ...]` | Key namespace prefix |

**Example:**

```python
from langgraph.store.memory import InMemoryStore
from deepagents.backends import StoreBackend

store = InMemoryStore()
backend = StoreBackend(store=store, namespace=("files", "user-123"))
```

---

## Use Cases

- **Per-user persistent files** in multi-user deployments: namespace by user ID
- **Shared knowledge base** across agent threads
- **Cross-session memory** without a full filesystem

---

## See Also

- [README.md](README.md) — backend comparison
- [state.md](state.md) — per-thread (non-persistent) alternative
