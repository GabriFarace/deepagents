# `libs/deepagents/deepagents/middleware/async_subagents.py`

> Async subagent middleware. It exposes tools for launching, checking,
> updating, cancelling, and listing background runs on remote LangGraph Agent
> Protocol servers.

## Position in the system

Synchronous `SubAgentMiddleware` blocks until a child graph returns one final
tool result. This module is for long-running or remote work: it starts a run on
a remote graph, stores the returned thread/run IDs in agent state, and lets the
main agent continue.

```
main agent
  ├─ start_async_task(description, subagent_type)
  │    └─ remote LangGraph thread + run
  ├─ async_tasks state tracks task_id/thread_id/run_id/status
  ├─ check_async_task(task_id) fetches live result when requested
  ├─ update_async_task(task_id, message) interrupts with a new run
  ├─ cancel_async_task(task_id)
  └─ list_async_tasks(status_filter) refreshes live statuses
```

`create_deep_agent()` installs this middleware when async subagent specs are
provided. The middleware's tools are normal LangChain structured tools whose
results are LangGraph `Command` updates, so task metadata survives checkpointing
and context compaction.

## Imports and module-level state

The module depends on `langgraph_sdk` for sync and async Agent Protocol
clients, LangChain middleware/tool types, `Command` for state updates, and
`append_to_system_message()` for prompt injection. `logger` is the only
module-level object with behavior.

Important constants are the model-facing launch tool description,
`ASYNC_TASK_TOOL_DESCRIPTION`; the full prompt addendum,
`ASYNC_TASK_SYSTEM_PROMPT`; and `_TERMINAL_STATUSES`, the statuses for which
`list_async_tasks` will not re-fetch a live run status.

## Functions and classes

### `AsyncSubAgent`

TypedDict describing a remote async agent. `name`, `description`, and
`graph_id` are required. `url` is optional and is passed to the LangGraph SDK;
when absent, sync clients reject use because ASGI/local transport requires
async invocation. `headers` lets callers add or override remote request
headers.

The main model sees only the name and description in prompt/tool text. The SDK
fields determine where tool implementations create threads and runs.

### `AsyncTask`

TypedDict persisted under the `async_tasks` state channel. It stores the
public `task_id` (same as remote `thread_id`), agent name, remote thread/run
IDs, cached status, and UTC timestamps for creation, last status check, and
last status change/update.

### `_tasks_reducer(existing, update)`

Merges async task updates into the existing task dictionary. LangGraph calls
this reducer whenever a tool returns `Command(update={"async_tasks": ...})`.
Existing tasks remain present unless overwritten by ID.

### `AsyncSubAgentState`

Extends `AgentState` with the `async_tasks` channel annotated with
`_tasks_reducer`. This is the persistence hook that allows task IDs to survive
multiple turns.

### Tool schema classes

`StartAsyncTaskSchema`, `CheckAsyncTaskSchema`, `UpdateAsyncTaskSchema`,
`CancelAsyncTaskSchema`, and `ListAsyncTasksSchema` define the model-facing
arguments for the five tools. Their field descriptions are part of the tool
schema and tell the model to pass task IDs verbatim and choose subagent types
from the available list.

### `_resolve_headers(spec)`

Copies any configured headers and adds `x-auth-scheme: langsmith` unless the
caller already supplied that header. This default matches LangGraph Platform
auth expectations while being harmless for many self-hosted servers.

### `_ClientCache`

Caches sync and async LangGraph SDK clients by `(url, headers)`. Multiple
subagent names pointing to the same server reuse the same underlying client.

`get_sync()` refuses specs with `url=None` because local ASGI transport is
async-only. `get_async()` permits `url=None` and lets the SDK choose its
default transport.

### `_validate_agent_type(agent_map, agent_type)`

Returns `None` for a known async subagent type or a model-facing error string
listing valid names. All tools call this or resolve a tracked task before
touching the remote server.

### `_build_start_tool(agent_map, clients, tool_description)`

Builds `start_async_task`. The sync and async implementations validate the
subagent type, create a remote thread, start a run against `spec["graph_id"]`
with the user task as a single user message, and return a `Command` that adds
a confirmation `ToolMessage` plus an `AsyncTask` record.

Failures from the SDK are caught, logged, and returned as plain tool strings so
the agent can report them instead of crashing the run.

### `_build_check_result(run, thread_id, thread_values)`

Converts a remote run and optional thread state into a small JSON-serializable
dict. On success it extracts the final message content when available; on
error it includes remote error detail or a generic error string. Running and
other statuses include only `status` and `thread_id`.

### `_build_check_command(result, task, tool_call_id)`

Builds the state update for a check. It refreshes `last_checked_at`, updates
`last_updated_at` only when the status changed, writes the new task record, and
returns the check result as a JSON `ToolMessage`.

### `_resolve_tracked_task(task_id, runtime)`

Looks up a task in `runtime.state["async_tasks"]` after stripping whitespace
from the provided ID. It returns either the `AsyncTask` or an error string. The
check, update, cancel, and list flows never trust conversation-history task
text; they use this state channel.

### `_build_check_tool(clients)`

Builds `check_async_task`. It resolves the tracked task, fetches the live run
status, and if the run succeeded fetches thread values so it can include the
final output. It returns a `Command` from `_build_check_command()`.

The async variant mirrors the sync flow with awaited SDK calls. Fetching thread
values is best-effort; failures are logged and the status result still returns.

### `_build_update_tool(agent_map, clients)`

Builds `update_async_task`. The tool creates a new run on the same thread with
`multitask_strategy="interrupt"`, which interrupts the current run and sends a
new user message into the remote conversation. The public task ID stays the
same, but the stored `run_id` changes and status resets to `running`.

### `_build_cancel_tool(clients)`

Builds `cancel_async_task`. It resolves the tracked task, calls the SDK cancel
endpoint for the stored run, and writes a task update with status
`cancelled`. SDK errors are returned as tool-facing strings.

### `_fetch_live_status(clients, task)` and `_afetch_live_status(clients, task)`

Fetch the current remote run status unless the cached status is terminal. If
the remote status call fails, the helpers log the exception and return the
cached status so listing tasks remains useful during transient remote failures.

### `_format_task_entry(task, status)`

Formats one task for `list_async_tasks` output as a single bullet containing
the full task ID, agent name, and status. It deliberately does not abbreviate
the task ID because the prompt tells the model never to do so.

### `_filter_tasks(tasks, status_filter)`

Filters by cached status before live statuses are fetched. `None` and `"all"`
return every task. This means a task whose cached status is `"running"` but
live status is now `"success"` appears in a `running`-filtered list once, then
its cache is updated by the command result.

### `_build_list_tasks_tool(clients)`

Builds `list_async_tasks`. It reads all tracked tasks, filters by cached
status, refreshes live statuses for the selected tasks, returns a summary
`ToolMessage`, and updates every selected task's cached status and timestamps.

The async implementation fetches live statuses concurrently with
`asyncio.gather()`.

### `_build_async_subagent_tools(agents)`

Normalizes the list of agent specs into a name map, creates one shared
`_ClientCache`, renders the launch tool description with available agents, and
returns the five structured tools in start/check/update/cancel/list order.

### `AsyncSubAgentMiddleware`

`AgentMiddleware` that installs async subagent tools and appends async
subagent instructions to every model request. Its state schema is
`AsyncSubAgentState`.

The constructor rejects an empty subagent list and duplicate names, builds the
tools, and appends an "Available async subagent types" list to the configured
system prompt. Passing `system_prompt=None` disables prompt injection while
still exposing the tools.

#### `wrap_model_call(request, handler)` and `awrap_model_call(request, handler)`

Append `self.system_prompt` to the request system message and delegate to the
next model-call handler. They do not inspect messages, alter tool lists, or
post-process model responses.

## System prompts and tool descriptions

`ASYNC_TASK_TOOL_DESCRIPTION`:

```text
Start an async subagent on a remote server. The subagent runs in the background and returns a task ID immediately.

Available async agent types:
{available_agents}

## Usage notes:
1. This tool launches a background task and returns immediately with a task ID. Report the task ID to the user and stop — do NOT immediately check status.
2. Use `check_async_task` only when the user asks for a status update or result.
3. Use `update_async_task` to send new instructions to a running task.
4. Multiple async subagents can run concurrently — launch several and let them run in the background.
5. The subagent runs on a remote server, so it has its own tools and capabilities.
```

`ASYNC_TASK_SYSTEM_PROMPT`:

```text
## Async subagents (remote LangGraph servers)

You have access to async subagent tools that launch background tasks on remote LangGraph servers.

### Tools:
- `start_async_task`: Start a new background task. Returns a task ID immediately.
- `check_async_task`: Get current status and result of a task. Returns status + result (if complete).
- `update_async_task`: Send new instructions to a running task. Returns confirmation + updated status.
- `cancel_async_task`: Stop a running task. Returns confirmation.
- `list_async_tasks`: List all tracked tasks with live statuses. Returns summary of all tasks.

### Workflow:
1. **Start** — Use `start_async_task` to start a task. Report the task ID to the user and stop.
   Do NOT immediately check the status — the task runs in the background while you and the user continue other work.
2. **Check (on request)** — Only use `check_async_task` when the user explicitly asks for a status update or
   result. If the status is "running", report that and stop — do not poll in a loop.
3. **Update** (optional) — Use `update_async_task` to send new instructions to a running task. This interrupts
   the current run and starts a fresh one on the same thread. The task_id stays the same.
4. **Cancel** (optional) — Use `cancel_async_task` to stop a task that is no longer needed.
5. **Collect** — When `check_async_task` returns status "success", the result is included in the response.
6. **List** — Use `list_async_tasks` to see live statuses for all tasks at once, or to recall task IDs after context compaction.

### Critical rules:
- After launching, ALWAYS return control to the user immediately. Never auto-check after launching.
- Never poll `check_async_task` in a loop. Check once per user request, then stop.
- If a check returns "running", tell the user and wait for them to ask again.
- Task statuses in conversation history are ALWAYS stale — a task that was "running" may now be done.
  NEVER report a status from a previous tool result. ALWAYS call a tool to get the current status:
  use `list_async_tasks` when the user asks about multiple tasks or "all tasks",
  use `check_async_task` when the user asks about a specific task.
- Always show the full task_id — never truncate or abbreviate it.

### When to use async subagents:
- Long-running tasks that would block the main agent
- Tasks that benefit from running on specialized remote deployments
- When you want to run multiple tasks concurrently and collect results later
```

Other tool descriptions:

```text
Check the status of an async subagent task. Returns the current status and, if complete, the result.
```

```text
Send updated instructions to an async subagent. Interrupts the current run and starts a new one on the same thread, so the subagent sees the full conversation history plus your new message. The task_id remains the same.
```

```text
Cancel a running async subagent task. Use this to stop a task that is no longer needed.
```

```text
List tracked async subagent tasks with their current live statuses. By default shows all tasks. Use `status_filter` to narrow by status (e.g. 'running', 'success', 'error', 'cancelled'). Use `check_async_task` to get the full result of a specific completed task.
```

## Flow walk-through

1. `AsyncSubAgentMiddleware.__init__` validates names, builds tools, and
   prepares prompt text.
2. On each model call, `wrap_model_call()` appends async-subagent instructions.
3. The model calls `start_async_task`; the tool creates a remote thread/run and
   persists the `AsyncTask`.
4. Later user requests call `check_async_task` or `list_async_tasks`, which
   refresh live status instead of trusting stale transcript text.
5. `update_async_task` reuses the same thread with a new interrupted run;
   `cancel_async_task` marks the stored task cancelled.

## Gotchas

`list_async_tasks(status_filter=...)` filters by cached status before live
refresh. A running task that finished remotely may still be included in the
running-filtered result once, and the returned command then updates state.

Sync client use requires a configured URL. Specs with `url=None` are only safe
through async tool execution.
