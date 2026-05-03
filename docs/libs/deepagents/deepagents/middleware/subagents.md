# `deepagents/middleware/subagents.py`

## High-Level Purpose

`subagents.py` defines the `SubAgent` spec type and `SubAgentMiddleware`, which adds a `task` tool to the agent. The `task` tool lets the parent agent spawn ephemeral sub-agents for specific delegated work — for example, "use the researcher sub-agent to look up X". Each sub-agent runs its own independent LangGraph graph and returns a single result.

---

## Key Types

### `SubAgent` (TypedDict)

The configuration spec for a sub-agent:

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Identifier used in `task(subagent_type="name")` |
| `description` | Yes | Tells the parent when to use this sub-agent |
| `system_prompt` | No | System prompt for the sub-agent |
| `model` | No | Model override (defaults to parent's model) |
| `tools` | No | Tool list override (defaults to parent's tools) |
| `middleware` | No | Additional middleware for this sub-agent |
| `interrupt_on` | No | Tool names that pause for HITL in this sub-agent |
| `skills` | No | Skill paths for this sub-agent |
| `permissions` | No | Filesystem permissions for this sub-agent |

### `CompiledSubAgent` (TypedDict)

A pre-compiled sub-agent:

| Field | Description |
|---|---|
| `name` | Same as `SubAgent.name` |
| `description` | Same as `SubAgent.description` |
| `runnable` | A `Runnable` that accepts messages and returns a response |

Use `CompiledSubAgent` when you want to bring your own graph rather than using `create_deep_agent()`.

---

## Key Class

### `SubAgentMiddleware(AgentMiddleware)`

Adds the `task` tool and injects sub-agent catalog into the system prompt.

**The `task` tool:**

```
task(subagent_type: str, description: str) → str
```

- `subagent_type` — name of the sub-agent to invoke
- `description` — detailed description of what to do (becomes the sub-agent's first human message)
- Returns the sub-agent's final response as a string

**Sub-agent invocation steps:**

1. Strips state keys that shouldn't pass to sub-agents: `todos`, `skills_metadata`, `memory_contents`, `structured_response`
2. Invokes the sub-agent's compiled graph with the description as the first message
3. Extracts the final `AIMessage` content from the sub-agent's output
4. Returns it as a `ToolMessage` to the parent

---

## General-Purpose Sub-agent

`create_deep_agent()` auto-adds a general-purpose sub-agent unless disabled. It has:
- The same tools as the parent agent
- The same permissions
- No specialized system prompt beyond the default

This means any task delegation works out-of-the-box, even without explicitly configuring sub-agents.

---

## Parallel Dispatch

Sub-agents can be invoked in parallel. The agent can call `task()` multiple times in a single response (via parallel tool calls), and the middleware dispatches them concurrently using `asyncio.gather()`. Results arrive together before the next model call.

---

## Architecture Notes

**State isolation:** Each sub-agent invocation starts a fresh conversation with only the task description as the first message. There is no shared state — the sub-agent can't read the parent's conversation history.

**No bidirectional communication:** Sub-agents are fire-and-return. There are no callbacks or mid-execution updates. The parent gets one final response.

---

## See Also

- [README.md](README.md) — middleware stack
- [async_subagents.md](async_subagents.md) — background (non-blocking) sub-agents
- [../graph.md](../graph.md) — `subagents` parameter
