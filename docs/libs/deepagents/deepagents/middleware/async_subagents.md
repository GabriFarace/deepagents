# `deepagents/middleware/async_subagents.py`

## High-Level Purpose

`AsyncSubAgentMiddleware` enables the main agent to launch **non-blocking background tasks** on remote LangGraph server deployments. Unlike `SubAgentMiddleware` (which blocks until completion), async subagents return a task ID immediately and run concurrently while the main agent and user continue working.

This module connects to remote LangGraph deployments via the `langgraph-sdk` package.

## Key Concepts

- **Non-blocking:** `start_async_task` returns a `task_id` immediately; the agent should NOT auto-check status.
- **SDK-based:** Communicates with remote LangGraph servers via `langgraph_sdk.get_client` (async) and `get_sync_client` (sync).
- **State persistence:** Running tasks are stored in `AsyncSubAgentState.async_tasks` (merged via `_tasks_reducer`).
- **Auth via environment:** Authentication is handled by the LangGraph SDK via `LANGGRAPH_API_KEY`, `LANGSMITH_API_KEY`, or `LANGCHAIN_API_KEY` environment variables.

## TypedDicts

### `AsyncSubAgent`
Specification for a remote async subagent.

| Field | Required | Type | Description |
|---|---|---|---|
| `name` | Yes | `str` | Unique identifier for the async subagent type |
| `description` | Yes | `str` | What this subagent does |
| `graph_id` | Yes | `str` | Graph name or assistant ID on the remote server |
| `url` | No | `str` | Remote LangGraph server URL |
| `headers` | No | `dict[str, str]` | Additional HTTP headers |

### `AsyncTask`
A tracked async task persisted in agent state.

| Field | Type | Description |
|---|---|---|
| `task_id` | `str` | Same as `thread_id`; returned to user for tracking |
| `agent_name` | `str` | The async subagent type name |
| `thread_id` | `str` | LangGraph thread ID |
| `run_id` | `str` | LangGraph run ID |
| `status` | `str` | Current status: `"running"`, `"success"`, `"error"`, `"cancelled"` |
| `created_at` | `str` | ISO-8601 UTC timestamp |
| `last_checked_at` | `str` | ISO-8601 UTC timestamp |
| `last_updated_at` | `str` | ISO-8601 UTC timestamp |

### `AsyncSubAgentState(AgentState)`
State schema extension. Adds `async_tasks: Annotated[dict[str, AsyncTask], _tasks_reducer]` merged by `_tasks_reducer` (dict merge, later updates overwrite earlier).

## System Prompts

### `ASYNC_TASK_TOOL_DESCRIPTION`
Agent-facing description for `start_async_task`. Lists available agent types and 5 usage rules.

### `ASYNC_TASK_SYSTEM_PROMPT`
Injected into the system message. Covers all 5 async task tools and detailed workflow rules including the critical rule: after launching, ALWAYS return control to the user immediately and never poll in a loop.

## Private Helpers

### `_resolve_headers(spec: AsyncSubAgent) -> dict[str, str]`
Builds headers for the remote server. Adds `"x-auth-scheme": "langsmith"` if not already set.

### `_ClientCache`
Lazily-created, cached LangGraph SDK clients keyed by `(url, frozenset(headers))`. Prevents creating multiple clients for the same server.

**Methods:**
- `get_sync(name: str) -> SyncLangGraphClient` — Returns cached or creates new sync client. Raises `ValueError` if `url` is None (ASGI transport requires async).
- `get_async(name: str) -> LangGraphClient` — Returns cached or creates new async client.

### `_validate_agent_type(agent_map, agent_type) -> str | None`
Returns an error message if `agent_type` is not in the available agents map, or `None` if valid.

## Tool Builder Functions

The middleware builds 5 `StructuredTool` instances via internal builder functions:

### `start_async_task`
Creates a new thread on the remote server, starts a run, persists the task in state.

**Parameters:** `description` (task instructions), `subagent_type` (agent type name).

**Returns:** `Command(update={"messages": [ToolMessage(...)], "async_tasks": {...}})` with the `task_id`.

### `check_async_task`
Polls `runs.join()` on the remote thread to get current status. Updates `last_checked_at` and status in state.

**Parameters:** `task_id`.

### `update_async_task`
Cancels the current run (via `runs.cancel(wait=True)`) then starts a new run with updated instructions on the same thread. Maintains the same `task_id`.

**Parameters:** `task_id`, `message` (new instructions).

### `cancel_async_task`
Cancels the current run.

**Parameters:** `task_id`.

### `list_async_tasks`
Lists all tracked tasks by calling `runs.list()` on each thread to get fresh statuses. Returns a formatted summary.

## Class: `AsyncSubAgentMiddleware(AgentMiddleware)`

### Constructor

```python
AsyncSubAgentMiddleware(async_subagents: list[AsyncSubAgent])
```

### `wrap_model_call` / `awrap_model_call`
Injects `ASYNC_TASK_SYSTEM_PROMPT` into the system message and adds all 5 async task tools to the request before forwarding to the handler.

## Dependencies

- `langgraph_sdk` — `get_client`, `get_sync_client`
- `langchain.agents.middleware.types` — middleware types
- `langchain_core.messages.ToolMessage`
- `langchain_core.tools.StructuredTool`
- `langgraph.types.Command`
- `deepagents.middleware._utils.append_to_system_message`
