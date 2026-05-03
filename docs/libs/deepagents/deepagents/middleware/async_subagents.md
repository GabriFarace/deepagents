# `deepagents/middleware/async_subagents.py`

## High-Level Purpose

`AsyncSubAgentMiddleware` adds tools for launching and monitoring remote background tasks. Unlike `SubAgentMiddleware` (which blocks until the sub-agent returns), async sub-agents run independently in the background. The parent agent can continue doing other work and check back later for results.

---

## Key Type

### `AsyncSubAgent` (TypedDict)

| Field | Required | Description |
|---|---|---|
| `graph_id` | Yes | Remote graph identifier (LangGraph deployment or Agent Protocol) |
| `url` | No | Deployment URL (defaults to LANGSMITH_ENDPOINT) |
| `headers` | No | Auth headers for the remote deployment |
| `name` | No | Display name in tool catalog |
| `description` | No | Tells parent agent when to use this async agent |

---

## Key Class

### `AsyncSubAgentMiddleware(AgentMiddleware)`

Adds these tools to the agent:

| Tool | Description |
|---|---|
| `start_async_task(graph_id, description)` | Launches a background task; returns a `task_id` |
| `check_async_task(task_id)` | Returns current status and output (if available) |
| `update_async_task(task_id, message)` | Sends a follow-up message to a running task |
| `cancel_async_task(task_id)` | Cancels a running task |
| `list_async_tasks()` | Shows all tasks and their statuses |

**Task states:** `pending`, `running`, `completed`, `failed`, `cancelled`

---

## Architecture Notes

**Non-blocking dispatch:** `start_async_task()` returns immediately with a `task_id`. The remote graph starts executing in its own process. The parent agent doesn't wait.

**Polling pattern:** The agent typically calls `start_async_task()` for multiple independent tasks, then calls `check_async_task()` on each to collect results when needed.

**CLI configuration:** Async sub-agents are configured in `~/.deepagents/config.toml`:
```toml
[async_subagents]
researcher = {graph_id = "researcher", url = "https://deploy.langchain.com/..."}
```

---

## See Also

- [subagents.md](subagents.md) — synchronous (blocking) sub-agents
- [README.md](README.md) — middleware stack
