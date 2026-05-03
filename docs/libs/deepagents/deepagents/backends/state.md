# `deepagents/backends/state.py`

## High-Level Purpose

`StateBackend` stores files in LangGraph agent state — as keys in the graph's state dict — rather than on disk. Files persist within a conversation thread (across multiple agent turns) but are lost when the thread ends. This makes `StateBackend` ideal for testing, in-memory agents, and scenarios where you don't want any disk side effects.

---

## Key Class

### `StateBackend`

Implements `BackendProtocol`. No shell (`execute`) support.

**How it works:** Each file is stored as a key in the LangGraph state under a configurable namespace. Reads use `get_config()` to access the current state. Writes use `CONFIG_KEY_SEND` to update state.

**Constructor parameters:**

| Parameter | Default | Description |
|---|---|---|
| `namespace` | `"files"` | State key prefix for file storage |

**Usage with `create_deep_agent()`:**

```python
graph = create_deep_agent(backend=StateBackend())
result = graph.invoke({
    "messages": [{"role": "user", "content": "Read /data.txt"}],
    "files": {"/data.txt": {"content": "hello", "encoding": "utf-8", ...}},
})
```

Pass initial files in the `"files"` key of the input state.

---

## Architecture Notes

**No execute:** `StateBackend` does not implement `execute()`. The agent will not have a shell tool. If you need shell access, use `LocalShellBackend` or a sandbox backend.

**State isolation:** Each LangGraph thread has its own state. Files written in one thread are not visible to another. For cross-thread file sharing, use `StoreBackend`.

---

## See Also

- [README.md](README.md) — backend comparison
- [store.md](store.md) — for cross-thread persistence
