# `libs/cli/deepagents_cli/remote_client.py`

## High-Level Purpose

`remote_client.py` provides `RemoteAgent`, the HTTP client that the TUI uses to talk to the LangGraph server. It wraps the `langgraph-sdk` client with CLI-specific streaming logic: it sends user messages to the server, iterates the SSE event stream, and translates raw LangGraph state updates into TUI widget mutations via a `StreamHandler`. This file is the communication backbone of the CLI.

---

## Key Classes

### `RemoteAgent`

The TUI's handle on the running LangGraph server. One instance is created per server startup and reused for all messages in the session.

**Constructor parameters:**

| Parameter | Type | Purpose |
|---|---|---|
| `url` | `str` | Base URL of the LangGraph server |
| `graph_id` | `str` | Graph name in the server (default `"agent"`) |
| `on_update` | `Callable` | Callback invoked for each widget mutation |

**Key methods:**

#### `async astream(message, thread_id, config=None) → AsyncIterator[StreamChunk]`

Sends `message` to the server and yields parsed stream chunks until the graph run completes or is interrupted.

Internally calls the LangGraph SDK's `client.runs.stream()` with `stream_mode=["updates", "messages"]`, which gives both state-level updates (tool calls, tool results) and token-level streaming (partial AI text).

Returns control to the caller when:
- The graph returns a `__end__` event (agent finished)
- The graph returns an `__interrupt__` event (HITL gate hit)
- An exception is raised (network error, server crash)

#### `async resume(thread_id, decision) → AsyncIterator[StreamChunk]`

Resumes a paused graph run after a HITL decision. Calls `client.runs.stream()` with the interrupt decision in the config. Used by `app.py` after the user approves/rejects a tool call.

#### `async create_thread() → str`

Creates a new LangGraph thread via the server API and returns the `thread_id`. Called from `CLIApp._start_server()` for each new session.

---

### `StreamHandler`

Translates raw SSE event data from the LangGraph server into TUI widget operations. Owned by `RemoteAgent`, called once per stream chunk.

#### `handle_chunk(chunk) → list[UIAction]`

Parses a raw LangGraph stream chunk and returns a list of `UIAction` objects that describe widget mutations. The TUI applies these actions in order.

**Chunk types and resulting actions:**

| LangGraph event | UIAction produced |
|---|---|
| `messages` update with `AIMessageChunk` | Append text tokens to the current `AssistantMessage` widget |
| `updates` with new `AIMessage` (no streaming) | Create or finalize `AssistantMessage` |
| `updates` with `ToolCall` | Create a `ToolCallMessage` widget with "pending" status |
| `updates` with `ToolMessage` | Attach the result to the matching `ToolCallMessage` widget |
| `__interrupt__` event | Emit `InterruptAction` with interrupt data |
| `ask_user` tool call | Emit `AskUserAction` with the question text |
| Error event | Create an `ErrorMessage` widget |

**UIAction types:**

| Action | Effect in TUI |
|---|---|
| `AppendTextAction` | Append tokens to the current `AssistantMessage` |
| `CreateToolCallAction` | Add a new `ToolCallMessage` to the transcript |
| `UpdateToolCallAction` | Update an existing `ToolCallMessage` with result |
| `InterruptAction` | Pause streaming, show `ApprovalMenu` modal |
| `AskUserAction` | Show `AskUserMenu` modal |
| `ErrorAction` | Show `ErrorMessage` in transcript |

---

## Architecture Notes

**Token streaming:** LangGraph's `stream_mode=["messages"]` delivers token-by-token `AIMessageChunk` events. Each chunk's `content` is appended to the current `AssistantMessage` widget without waiting for the full response. This is what makes the "typewriter" effect in the TUI.

**Interrupt handling:** When the LangGraph graph hits an interrupt node (HITL gate), it emits a special `__interrupt__` event with the interrupt payload (tool name, args, description). `StreamHandler` converts this to an `InterruptAction`. The TUI's `_run_agent()` sees the action, shows the approval modal, awaits the decision, and calls `remote_agent.resume(thread_id, decision)` to continue.

**Thread safety:** All `UIAction` callbacks from the stream handler are dispatched to the Textual main loop via `app.call_from_thread()`. The SSE iteration runs in an asyncio task, while Textual's event loop owns widget state.

---

## See Also

- [app.md](app.md) — `_run_agent()` and `_handle_interrupt()` that consume this client
- [server_manager.md](server_manager.md) — creates the `RemoteAgent` instance
- [widgets/messages.md](widgets/messages.md) — the widget classes mutated by UIActions
- [widgets/approval.md](widgets/approval.md) — the `ApprovalMenu` modal shown on `InterruptAction`
