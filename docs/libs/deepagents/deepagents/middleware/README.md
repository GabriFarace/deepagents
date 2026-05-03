# `deepagents/middleware/` — Middleware Stack

Middleware is the mechanism by which tools, system prompt sections, and safety gates are added to the agent. Every `AgentMiddleware` subclass wraps each model call by implementing `wrap_model_call()`. The agent runs through the middleware stack on every LLM invocation.

---

## Default Stack (in execution order)

```
1.  TodoListMiddleware          — checklist-based task planning
2.  SkillsMiddleware            — skill catalog injection
3.  FilesystemMiddleware        — file/shell tools (required; always present)
4.  SubAgentMiddleware          — "task" delegation tool
5.  SummarizationMiddleware     — context window compaction
6.  PatchToolCallsMiddleware    — dangling tool call repair
7.  AsyncSubAgentMiddleware     — remote background tasks
8.  [user middleware]           — inserted here via create_deep_agent(middleware=[...])
9.  [profile extra_middleware]  — harness profile additions
10. _ToolExclusionMiddleware    — removes excluded tools
11. AnthropicPromptCachingMiddleware — cache_control marks
12. MemoryMiddleware            — AGENTS.md injection
13. HumanInTheLoopMiddleware    — interrupt gates
```

The stack is applied in reverse order (item 13 wraps item 12, which wraps item 11, …). Item 1 (TodoListMiddleware) is the innermost wrapper — it runs last before the model call.

---

## Middleware Inventory

| File | Middleware | Role |
|---|---|---|
| [`filesystem.md`](filesystem.md) | `FilesystemMiddleware` | All file and shell tools |
| [`skills.md`](skills.md) | `SkillsMiddleware` | Skill catalog injection |
| [`memory.md`](memory.md) | `MemoryMiddleware` | AGENTS.md loading |
| [`subagents.md`](subagents.md) | `SubAgentMiddleware`, `AsyncSubAgentMiddleware` | Subagent task delegation |
| [`async_subagents.md`](async_subagents.md) | `AsyncSubAgentMiddleware` | Background remote tasks |
| [`summarization.md`](summarization.md) | `SummarizationMiddleware`, `SummarizationToolMiddleware` | Context compaction |
| [`human_in_the_loop.md`](human_in_the_loop.md) | `HumanInTheLoopMiddleware` | HITL approval gates |

---

## How Middleware Works

Each middleware implements:
```python
class MyMiddleware(AgentMiddleware):
    def wrap_model_call(self, model_call: Callable, state: AgentState, config: RunnableConfig) -> Callable:
        # modify state, inject system prompt, add tools, etc.
        # return a modified model_call
        ...
```

Middlewares compose: each one receives the partially-wrapped `model_call` from the middleware above it and can add more behavior before/after calling it.

Common patterns:
- **Tool injection:** bind additional tools to the model (`model.bind_tools([...])`)
- **System prompt injection:** prepend or append text to the system message
- **Pre-call hook:** inspect state before the model runs
- **Post-call hook:** process the model's response (e.g., save a summary)

---

## Excluding Middleware

Pass class names or alias strings to `exclude_middleware` in `create_deep_agent()`:

```python
graph = create_deep_agent(
    exclude_middleware=["SkillsMiddleware", "TodoListMiddleware"]
)
```

`FilesystemMiddleware` and `SubAgentMiddleware` cannot be excluded.

---

## Ordering Principles

- **Outermost (last in list):** runs first; sees the original state
- **Innermost (first in list):** runs last; closest to the model call
- `HumanInTheLoopMiddleware` is outermost so it can intercept tool calls before they execute
- `AnthropicPromptCachingMiddleware` is near-outermost so it marks cache boundaries on the fully assembled prompt

---

## See Also

- [../graph.md](../graph.md) — `create_deep_agent()` assembles the stack
- [filesystem.md](filesystem.md) — the most important middleware (file/shell tools)
- [human_in_the_loop.md](human_in_the_loop.md) — HITL gates
- [summarization.md](summarization.md) — context window management
