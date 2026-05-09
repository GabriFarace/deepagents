# `libs/cli/deepagents_cli/textual_adapter.py`

> Textual UI adapter for agent execution.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._ask_user_types import AskUserRequest`

- `from deepagents_cli._cli_context import CLIContext`

- `from deepagents_cli._debug import configure_debug_logging`

- `from deepagents_cli._session_stats import ModelStats as ModelStats, SessionStats as SessionStats, SpinnerStatus as SpinnerStatus, format_token_count as format_token_count`

- `from deepagents_cli.config import build_stream_config`

- `from deepagents_cli.file_ops import FileOpTracker`

- `from deepagents_cli.formatting import format_duration`

- `from deepagents_cli.hooks import dispatch_hook`

- `from deepagents_cli.input import MediaTracker, parse_file_mentions`

- `from deepagents_cli.media_utils import create_multimodal_content`

- `from deepagents_cli.tool_display import format_tool_message_content`

- `from deepagents_cli.widgets.messages import AppMessage, AssistantMessage, DiffMessage, SummarizationMessage, ToolCallMessage`


## Functions and classes

### `_get_hitl_request_adapter(hitl_request_type: type)`

Return a cached `TypeAdapter(HITLRequest)`.

Additional notes from the source docstring:

```text
Avoids re-compiling the pydantic schema on every `execute_task_textual` call.

Args:
    hitl_request_type: The `HITLRequest` class (passed in because
        it is imported locally by the caller).

Returns:
    Shared `TypeAdapter` instance.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `print_usage_table(stats: SessionStats, wall_time: float, console: Console)`

Print a model-usage stats table to a Rich console.

Additional notes from the source docstring:

```text
When the session spans multiple models each gets its own row with a
totals row appended; single-model sessions show one row.

Args:
    stats: Cumulative session stats.
    wall_time: Total wall-clock time in seconds.
    console: Rich console for output.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_get_ask_user_adapter()`

Return a cached `TypeAdapter(AskUserRequest)`.

Additional notes from the source docstring:

```text
Returns:
    Shared `TypeAdapter` instance.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_is_summarization_chunk(metadata: dict | None)`

Check if a message chunk is from summarization middleware.

Additional notes from the source docstring:

```text
The summarization model is invoked with
`config={"metadata": {"lc_source": "summarization"}}`
(see `langchain.agents.middleware.summarization`), which
LangChain's callback system merges into the stream metadata dict.

Args:
    metadata: The metadata dict from the stream chunk.

Returns:
    Whether the chunk is from summarization and should be filtered.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `TextualUIAdapter`

Adapter for rendering agent output to Textual widgets.

Additional notes from the source docstring:

```text
This adapter provides an abstraction layer between the agent execution and the
Textual UI, allowing streaming output to be rendered as widgets.
```

Methods worth reading inside this class:

- `finalize_pending_tools_with_error(self, error: str)`: Mark all pending/running tool widgets as error and clear tracking.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_build_interrupted_ai_message(pending_text_by_namespace: dict[tuple, str], current_tool_messages: dict[str, Any])`

Build an AIMessage capturing interrupted state (text + tool calls).

Additional notes from the source docstring:

```text
Args:
    pending_text_by_namespace: Dict of accumulated text by namespace
    current_tool_messages: Dict of tool_id -> ToolCallMessage widget

Returns:
    AIMessage with accumulated content and tool calls, or None if empty.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_read_mentioned_file(file_path: Path, max_embed_bytes: int)`

Read a mentioned file for inline embedding (sync, for use with to_thread).

Additional notes from the source docstring:

```text
Args:
    file_path: Resolved path to the file.
    max_embed_bytes: Size threshold; larger files get a reference only.

Returns:
    Markdown snippet with the file content or a size-exceeded reference.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `execute_task_textual(user_input: str, agent: Any, assistant_id: str | None, session_state: Any, adapter: TextualUIAdapter, backend: Any=None, image_tracker: MediaTracker | None=None, context: CLIContext | None=None, *, sandbox_type: str | None=None, message_kwargs: dict[str, Any] | None=None, turn_stats: SessionStats | None=None)`

Execute a task with output directed to Textual UI.

Additional notes from the source docstring:

```text
This is the Textual-compatible version of execute_task() that uses
the TextualUIAdapter for all UI operations.

Args:
    user_input: The user's input message
    agent: The LangGraph agent to execute
    assistant_id: The agent identifier
    session_state: Session state with auto_approve flag
    adapter: The TextualUIAdapter for UI operations
    backend: Optional backend for file operations
    image_tracker: Optional tracker for images
    context: Optional `CLIContext` with model override and params, passed
        to the graph via `context=`.
    sandbox_type: Sandbox provider name for trace metadata, or `None`
        if no sandbox is active.
    message_kwargs: Extra fields merged into the stream input message
        dict (e.g., `additional_kwargs` for persisting skill metadata
        in the checkpoint).
    turn_stats: Pre-created `SessionStats` to accumulate into.

        When the caller holds a reference to the same object, stats are
        available even if this coroutine is cancelled before it can return.

        If `None`, a new instance is created internally.

Returns:
    Stats accumulated over this turn (request count, token counts,
        wall-clock time).

Raises:
    ValidationError: If HITL request validation fails (re-raised).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_handle_interrupt_cleanup(*, adapter: TextualUIAdapter, agent: Any, config: RunnableConfig, pending_text_by_namespace: dict[tuple, str], captured_input_tokens: int, captured_output_tokens: int, turn_stats: SessionStats, start_time: float)`

Shared cleanup for CancelledError and KeyboardInterrupt.

Additional notes from the source docstring:

```text
Args:
    adapter: UI adapter with display callbacks.
    agent: The LangGraph agent.
    config: Runnable config with `thread_id`.
    pending_text_by_namespace: Accumulated text per namespace.
    captured_input_tokens: Input tokens captured before interrupt.
    captured_output_tokens: Output tokens captured before interrupt.
    turn_stats: Stats for the current turn.
    start_time: Monotonic timestamp when the turn began.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_persist_context_tokens(agent: Any, config: RunnableConfig, tokens: int)`

Best-effort persist of the context token count into graph state.

Additional notes from the source docstring:

```text
Args:
    agent: The LangGraph agent (must support `aupdate_state`).
    config: Runnable config with `thread_id`.
    tokens: Total context tokens to persist.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_report_and_persist_tokens(adapter: TextualUIAdapter, agent: Any, config: RunnableConfig, captured_input_tokens: int, captured_output_tokens: int, *, shield: bool=False, approximate: bool=False)`

Update the token display and best-effort persist to graph state.

Additional notes from the source docstring:

```text
Args:
    adapter: UI adapter with token callbacks.
    agent: The LangGraph agent.
    config: Runnable config with `thread_id` in its configurable dict.
    captured_input_tokens: Total input tokens captured during the turn.
    captured_output_tokens: Total output tokens captured during the turn.
    shield: When `True`, suppress exceptions and `CancelledError` from the
        persist call so that interrupt handlers can safely await this.
    approximate: When `True`, signal to the UI that the count is stale
        (e.g. after an interrupted generation) by appending "+".
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_flush_assistant_text_ns(adapter: TextualUIAdapter, text: str, ns_key: tuple, assistant_message_by_namespace: dict[tuple, Any])`

Flush accumulated assistant text for a specific namespace.

Additional notes from the source docstring:

```text
Finalizes the streaming by stopping the MarkdownStream.
If no message exists yet, creates one with the full content.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
