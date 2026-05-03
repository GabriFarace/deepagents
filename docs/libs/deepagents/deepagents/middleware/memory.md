# `deepagents/middleware/memory.py`

## High-Level Purpose

`MemoryMiddleware` loads `AGENTS.md` files — per-project or per-agent instruction files — and injects their contents into the agent's system prompt. It is the mechanism by which agents "remember" project conventions, team preferences, and accumulated knowledge across sessions.

---

## Key Class

### `MemoryMiddleware(AgentMiddleware)`

**Constructor parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `memory_paths` | `list[str]` | required | Absolute paths to AGENTS.md files |
| `backend` | `BackendProtocol` | required | Backend used to read memory files |
| `add_cache_control` | `bool` | `False` | Add Anthropic ephemeral cache control to memory block |

---

## AGENTS.md Format

`AGENTS.md` is plain Markdown. There is no required structure. Common sections include:

- Project overview and purpose
- Build and test commands
- Code style conventions
- Known gotchas and constraints
- Architecture decisions

The agent is instructed to:
- Consult memory before starting tasks
- Update memory when it learns something that should persist
- Use `edit_file` to update `AGENTS.md` when the user asks it to remember something

---

## System Prompt Injection

Memory is injected as an XML block:

```xml
<agent_memory>
  # My Project

  ## Build Commands
  - `uv run pytest` — run tests
  - `uv run ruff check .` — lint
  
  ...
</agent_memory>
```

The system prompt also includes explicit guidelines about when and how to update memory:
- Save knowledge that persists across sessions (conventions, decisions)
- Don't save transient information (current task, temporary state)
- Never save credentials or API keys

---

## Cache Control

When `add_cache_control=True`, the memory block gets Anthropic's `cache_control: {type: "ephemeral"}` mark. This caches the memory block across API calls within a 5-minute window, reducing latency and cost for long sessions.

---

## Multiple Memory Sources

`memory_paths` can contain multiple paths:
```python
MemoryMiddleware(
    memory_paths=[
        "/home/user/.deepagents/AGENTS.md",    # user-level
        "/project/.deepagents/AGENTS.md",      # project-level
    ],
    backend=backend,
)
```

Files are injected in order; later files' contents appear below earlier ones.

---

## See Also

- [README.md](README.md) — middleware stack overview
- [../graph.md](../graph.md) — `memory_paths` parameter
