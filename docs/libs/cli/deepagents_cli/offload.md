# `libs/cli/deepagents_cli/offload.py`

> Business logic for the /offload command.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from langchain_core.messages import get_buffer_string`

- `from langchain_core.messages.utils import count_tokens_approximately`

- `from deepagents_cli.config import create_model`

- `from deepagents_cli.textual_adapter import format_token_count`


## Functions and classes

### `OffloadResult`

Successful offload result.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `OffloadThresholdNotMet`

Offload was a no-op — conversation is within the retention budget.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `OffloadModelError`

Raised when the model cannot be created for offloading.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `format_offload_limit(keep: tuple[str, int | float], context_limit: int | None)`

Format offload retention settings into a human-readable limit string.

Additional notes from the source docstring:

```text
Args:
    keep: Retention policy tuple `(type, value)` from summarization
        defaults, where `type` is one of `"messages"`, `"tokens"`, or
        `"fraction"`.
    context_limit: Model context limit when available.

Returns:
    A short display string describing the offload retention limit.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `offload_messages_to_backend(messages: list[Any], middleware: SummarizationMiddleware, *, thread_id: str, backend: BackendProtocol)`

Write messages to backend storage before offloading.

Additional notes from the source docstring:

```text
Appends messages as a timestamped markdown section to the conversation
history file, matching the `SummarizationMiddleware` offload pattern.

Filters out prior summary messages using the middleware's
`_filter_summary_messages` to avoid storing summaries-of-summaries.

Args:
    messages: Messages to offload.
    middleware: `SummarizationMiddleware` instance for filtering.
    thread_id: Thread identifier used to derive the storage path.
    backend: Backend to persist conversation history to.

Returns:
    File path where history was stored, `""` (empty string) if there were no
        non-summary messages to offload (not an error), or `None` if the
        write failed.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `perform_offload(*, messages: list[Any], prior_event: SummarizationEvent | None, thread_id: str, model_spec: str, profile_overrides: dict[str, Any] | None, context_limit: int | None, total_context_tokens: int, backend: BackendProtocol | None)`

Execute the offload workflow: summarize old messages and free context.

Additional notes from the source docstring:

```text
Args:
    messages: Current conversation messages from agent state.

        May be LangChain message objects or serialized dicts (the latter
        when read from a remote HTTP state snapshot).
    prior_event: Existing `_summarization_event` if any.

        In server mode `summary_message` may be a serialized message dict.
    thread_id: Thread identifier for backend storage.
    model_spec: Model specification string (e.g. "openai:gpt-4").
    profile_overrides: Optional profile overrides from CLI flags.
    context_limit: Model context limit from settings.
    total_context_tokens: Current total context token count, or `0` when
        no token tracker is available.
    backend: Backend for persisting offloaded history.

Returns:
    `OffloadResult` on success, `OffloadThresholdNotMet` when the
        conversation is within the retention budget.

Raises:
    OffloadModelError: If the model cannot be created.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
