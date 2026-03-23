# `tests/test_agent.py`

## High-Level Purpose

Integration tests for `AgentServerACP`. Covers the full ACP session lifecycle: text streaming, cancellation, multimodal input, HITL permission requests, plan updates, `approve_always` auto-approval, nested agent tool calls, and compatibility with prebuilt LangChain agents.

## Classes

### `FakeACPClient`

**Purpose:** A test double for the ACP `Client` interface that records all events and provides configurable responses to permission requests.

**Attributes:**
- `events: list[dict]` — Accumulated events from `session_update` and `request_permission` calls.
- `permission_outcomes: list[Literal["approve", "reject"]]` — Queue of outcomes to return, popped in FIFO order.

**Methods:**
- `session_update(session_id, update, source)` — Records the call as `{"type": "session_update", ...}`.
- `request_permission(options, session_id, tool_call, **kwargs)` — Records the call and returns `RequestPermissionResponse` with the next outcome from the queue (defaults to `"approve"` when queue is exhausted).

## Test Functions

| Test | Description |
|------|-------------|
| `test_acp_agent_prompt_streams_text` | Verifies that a simple text response is streamed as a `session_update` with `update_agent_message`. |
| `test_acp_agent_cancel_stops_prompt` | Verifies that calling `cancel()` during a prompt returns `stop_reason="cancelled"` or `"end_turn"`. |
| `test_acp_agent_prompt_streams_list_content_blocks` | Verifies that list-typed message content (multiple text blocks) is concatenated and sent as a single text update. |
| `test_acp_agent_initialize_and_modes` | Verifies `initialize()` returns correct capabilities and `new_session()` without modes returns `modes=None`. |
| `test_acp_agent_hitl_requests_permission_via_public_api` | Verifies HITL triggers a `request_permission` event with the correct tool call title for a custom tool. |
| `test_acp_deep_agent_hitl_interrupt_on_edit_file_requests_permission` | Verifies that `edit_file` HITL triggers a permission request with title `"Edit \`/tmp/x.txt\`"`. |
| `test_acp_agent_tool_call_chunk_starts_tool_call` | Unit tests `_process_tool_call_chunks()` directly to verify it populates `active_tool_calls` correctly. |
| `test_acp_agent_tool_result_completes_tool_call` | Verifies that a `ToolMessage` after a tool call chunk results in a `session_update` referencing the correct `tool_call_id`. |
| `test_acp_agent_multimodal_prompt_blocks_do_not_error` | Verifies that a prompt containing text, image, resource, and embedded resource blocks completes without error. |
| `test_acp_agent_end_to_end_clears_plan` | Verifies that rejecting a `write_todos` HITL sends plan updates (including a final empty-plan clear). |
| `test_acp_agent_hitl_approve_always_execute_auto_approves_next_time` | Verifies `approve_always` for an `execute` command stores the command signature in `_allowed_command_types` and auto-approves the same signature in subsequent interrupts. |
| `test_acp_agent_hitl_approve_always_tool_auto_approves_next_time` | Same as above but for a non-execute tool (`write_file`). |
| `test_acp_agent_hitl_client_cancel_raises_request_error` | Verifies that a `RequestError` raised by `request_permission` propagates out of `prompt()`. |
| `test_acp_agent_nested_agent_tool_call_returns_final_text` | Verifies that a tool which internally calls a subagent streams the subagent's final text response. |
| `test_acp_agent_with_prebuilt_langchain_agent_end_to_end` | Verifies `AgentServerACP` works with LangChain's prebuilt `create_agent()` (not just `create_deep_agent()`). |
| `test_acp_langchain_create_agent_nested_agent_tool_call_messages` | Verifies that a prebuilt LangChain agent with an interrupt-on interrupt raises `RequestError` for unsupported patterns. |
| `test_set_session_mode_resets_agent_with_new_mode` | Verifies that `set_session_mode()` re-creates the agent using the new mode context and the same CWD. |
| `test_reset_agent_with_compiled_state_graph` | Verifies `_reset_agent()` assigns the compiled graph directly when no factory is used. |
| `test_reset_agent_preserves_session_cwd` | Verifies the factory receives the correct CWD when the agent is reset after a mode change. |
| `test_acp_agent_hitl_requests_permission_only_once` | Regression test ensuring HITL permission is requested exactly once per interrupt (not twice due to double-handling). |

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `AgentServerACP`, `AgentSessionContext` | `deepagents_acp.server` | System under test |
| `GenericFakeChatModel` | `tests.chat_model` | Fake LLM |
| `create_deep_agent` | `deepagents` | Agent factory |
| `HumanInTheLoopMiddleware` | `langchain.agents.middleware` | HITL middleware |
| `acp` (multiple) | `agent-client-protocol` | ACP types for test assertions |
