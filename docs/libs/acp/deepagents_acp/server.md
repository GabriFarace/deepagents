# `deepagents_acp/server.py`

## High-Level Purpose

The core ACP server implementation. Provides `AgentServerACP`, which bridges a LangGraph-based Deep Agent (or any `CompiledStateGraph`) with the Agent Client Protocol (ACP). It handles session lifecycle, mode and model switching, streaming responses, tool call display, human-in-the-loop interrupts, plan updates, and cancellation.

## Classes

### `AgentSessionContext`

**Purpose:** Immutable data class carrying the runtime context for a single agent session, used when constructing an agent via a factory function.

**Definition:** `@dataclass(frozen=True, slots=True)`

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| `cwd` | `str` | Working directory for the session |
| `mode` | `str` | Current mode identifier (e.g. `"auto"`, `"manual"`) |
| `model` | `str \| None` | LLM model identifier for this session, or `None` if not configured |

---

### `AgentServerACP`

**Purpose:** The main ACP server class. Extends `acp.Agent` and implements all ACP protocol methods. Manages session state, streams agent responses to the connected client, handles tool call display, HITL permission requests, and dynamic mode/model switching.

**Inherits from:** `acp.Agent` (from the `agent-client-protocol` package)

#### `__init__`

```python
def __init__(
    self,
    agent: CompiledStateGraph | Callable[[AgentSessionContext], CompiledStateGraph],
    *,
    modes: SessionModeState | None = None,
    models: list[dict[str, str]] | None = None,
) -> None
```

**Parameters:**
- `agent`: Either a pre-compiled LangGraph state graph, or a factory callable `(AgentSessionContext) -> CompiledStateGraph`. When a factory is provided, a new agent is created per session/mode/model change.
- `modes`: Optional `SessionModeState` defining available session modes (e.g., auto-approve, manual). Only valid when `agent` is a factory.
- `models`: Optional list of model dicts, each with `"value"`, `"name"`, and optionally `"description"`. Only valid when `agent` is a factory.

**Internal State:**
- `_agent`: The current `CompiledStateGraph` instance (lazily initialized).
- `_agent_factory`: The stored agent or factory.
- `_session_cwds`: Maps session ID to working directory.
- `_session_modes`: Maps session ID to current mode ID.
- `_session_mode_states`: Maps session ID to full `SessionModeState`.
- `_session_models`: Maps session ID to current model identifier.
- `_session_plans`: Maps session ID to the current TODO list.
- `_allowed_command_types`: Maps session ID to a `set[tuple[str, str | None]]` of pre-approved command signatures (for `approve_always` HITL outcomes).
- `_cancelled`: Flag to abort the current streaming prompt.

#### Key Methods

##### `on_connect(conn: Client) -> None`
Stores the ACP client connection for sending session updates.

##### `initialize(protocol_version, client_capabilities, client_info, **kwargs) -> InitializeResponse`
Returns server capabilities to the ACP client, declaring support for image input.

##### `new_session(cwd, mcp_servers, **kwargs) -> NewSessionResponse`
Creates a new session with a UUID session ID. Initializes mode and model state. Returns `config_options` if modes or models are configured.

##### `set_session_mode(mode_id, session_id, **kwargs) -> SetSessionModeResponse`
Switches the session to a different mode and calls `_reset_agent()` to recreate the agent with the new mode context.

##### `set_config_option(config_id, session_id, value, **kwargs) -> SetSessionConfigOptionResponse`
Unified config option handler supporting both `config_id="mode"` and `config_id="model"`. Validates the new value against configured options and calls `_reset_agent()` on success. Raises `RequestError` for invalid values.

##### `cancel(session_id, **kwargs) -> None`
Sets `_cancelled = True` to abort the current `prompt()` execution.

##### `prompt(prompt, session_id, **kwargs) -> PromptResponse`
Main streaming handler. Converts incoming ACP content blocks to LangChain format, then streams the agent graph, processing:
- Text message chunks → forwarded as `update_agent_message` session updates
- Tool call chunks → accumulated and started via `start_tool_call` / `start_edit_tool_call`
- Tool result messages → completed via `update_tool_call`
- `__interrupt__` updates → routed to `_handle_interrupts()` for HITL permission requests
- `write_todos` calls → forwarded as `AgentPlanUpdate` plan updates
- Cancellation checks at each chunk

Returns `PromptResponse(stop_reason="end_turn")` or `"cancelled"` on cancellation.

##### `_reset_agent(session_id: str) -> None`
Re-creates the agent. If the factory is a `CompiledStateGraph`, assigns it directly. If it is a callable factory, constructs `AgentSessionContext` from the current session's `cwd`, `mode`, and `model`, then calls the factory.

##### `_build_config_options(session_id: str) -> list[SessionConfigOptionSelect | SessionConfigOption]`
Builds the list of config option objects combining mode and model selectors, using the current session's selected values.

**ACP v0.9 compatibility:** `agent-client-protocol` v0.9.0 removed the `SessionConfigOption` wrapper type; config options are now bare `SessionConfigOptionSelect` instances. `deepagents-acp` uses a conditional import so it works with both v0.8.x (with wrapper) and v0.9.0+ (without). When `SessionConfigOption` is importable, options are wrapped as `SessionConfigOption(root=...)`. When not available (v0.9.0+), bare `SessionConfigOptionSelect` instances are appended directly.

##### `_handle_interrupts(current_state, session_id) -> list[dict]`
Processes LangGraph interrupt nodes. For each interrupt, reads `action_requests`, checks the `_allowed_command_types` allowlist, and either auto-approves or calls `client.request_permission()`. Maps the client's response (`approve`, `approve_always`, `reject`) to `{"type": "approve"}` / `{"type": "reject"}` decisions. On `approve_always`, stores the command signature in `_allowed_command_types[session_id]` for future auto-approval.

##### `_process_tool_call_chunks(session_id, message_chunk, active_tool_calls, tool_call_accumulator) -> None`
Accumulates streaming `tool_call_chunks` by index. When a complete set of arguments has been parsed (valid JSON), emits a `start_tool_call` or `start_edit_tool_call` session update via the client.

##### `_create_tool_call_start(tool_id, tool_name, tool_args) -> ToolCallStart`
Maps tool names to `ToolKind` values and builds the appropriate `ToolCallStart` update with human-readable titles (e.g., `"Read \`/path/file\`"`, `"Edit \`/path/file\`"`, `"Write \`/path/file\`"`). For `edit_file`, creates a diff view using `tool_diff_content`.

##### `_handle_todo_update(session_id, todos, *, log_plan) -> None`
Converts a `write_todos` tool call's list of todo dicts into `PlanEntry` objects and sends an `AgentPlanUpdate` session update. Optionally logs the plan as a text message.

##### `_all_tasks_completed(plan) -> bool`
Returns `True` if every todo in the plan has `status == "completed"`.

##### `_clear_plan(session_id) -> None`
Sends an empty `AgentPlanUpdate` to the client and clears the in-memory plan.

##### `_log_text(session_id, text) -> None`
Sends an `update_agent_message` session update with a text block.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `acp` (multiple symbols) | `agent-client-protocol` | ACP protocol types, helpers, and base class |
| `acp.exceptions.RequestError` | `agent-client-protocol` | Raised for invalid config values |
| `acp.schema` (multiple) | `agent-client-protocol` | Protocol schema types |
| `deepagents.create_deep_agent` | `deepagents` | Agent construction helper |
| `deepagents.backends` | `deepagents` | `CompositeBackend`, `FilesystemBackend`, `StateBackend` |
| `langgraph.checkpoint.memory.MemorySaver` | `langgraph` | Fallback checkpointer for stateless agents |
| `langgraph.graph.state.CompiledStateGraph` | `langgraph` | Type for compiled agent graphs |
| `langgraph.types.Command`, `StateSnapshot` | `langgraph` | Resume commands and state inspection |
| `deepagents_acp.utils` | local | Content block conversion and command utilities |
