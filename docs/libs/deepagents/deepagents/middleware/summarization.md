# `deepagents/middleware/summarization.py`

## High-Level Purpose

This module provides automatic and on-demand conversation compaction. When the conversation context fills up (measured in tokens, messages, or fraction of model context), older messages are summarized via an LLM call and the full history is offloaded to the backend for later retrieval. This prevents context overflow errors and reduces API costs.

Two middleware classes plus a factory:
- **`SummarizationMiddleware`** — Automatic compaction triggered by a threshold.
- **`SummarizationToolMiddleware`** — Exposes a `compact_conversation` tool for manual triggering.
- **`create_summarization_middleware`** — Internal factory used by `create_deep_agent`.

## Key Types

### `ContextSize` (from `langchain.agents.middleware.summarization`)
A union type specifying a compaction threshold:
- `("tokens", int)` — Trigger when conversation exceeds N tokens
- `("messages", int)` — Trigger when conversation exceeds N messages
- `("fraction", float)` — Trigger when conversation uses this fraction of model's context window

### `SummarizationEvent`
TypedDict capturing a single summarization occurrence.

| Field | Type | Description |
|---|---|---|
| `cutoff_index` | `int` | Index in messages where compaction occurred |
| `summary_message` | `HumanMessage` | The generated summary |
| `file_path` | `str \| None` | Path where history was offloaded, or None on failure |

### `TruncateArgsSettings`
Settings for the pre-summarization optimization that truncates large tool-call arguments in old messages (lighter than full summarization).

| Field | Type | Default | Description |
|---|---|---|---|
| `trigger` | `ContextSize \| None` | — | When to activate argument truncation |
| `keep` | `ContextSize` | — | How many recent messages to leave intact |
| `max_length` | `int` | — | Max chars per argument before clipping |
| `truncation_text` | `str` | — | Replacement suffix after truncation |

### `SummarizationState(AgentState)`
State schema with `_summarization_event: PrivateStateAttr` storing the most recent compaction event.

### `SummarizationDefaults`
TypedDict returned by `compute_summarization_defaults`.

## Functions

### `compute_summarization_defaults(model: BaseChatModel) -> SummarizationDefaults`

Computes default thresholds based on the model's profile.

**Returns:**
- If the model has `profile.max_input_tokens`: fraction-based settings (trigger at 85%, keep 10%).
- Otherwise: conservative fixed settings (trigger at 170K tokens or 20 messages; keep 6 messages).

### `create_summarization_middleware(model, backend) -> SummarizationMiddleware`

Convenience factory used internally by `create_deep_agent`. Creates a `SummarizationMiddleware` instance with model-aware defaults.

## Class: `SummarizationMiddleware`

Extends `langchain.agents.middleware.summarization.SummarizationMiddleware` (from LangChain) with backend-backed history offloading.

### Constructor

```python
SummarizationMiddleware(
    model: str | BaseChatModel,
    backend: BACKEND_TYPES,
    trigger: ContextSize | None = None,
    keep: ContextSize | None = None,
    summary_prompt: str = DEFAULT_SUMMARY_PROMPT,
    truncate_args_settings: TruncateArgsSettings | None = None,
)
```

**Parameters:**
- `model` — LLM used to generate summaries (may differ from the main agent model).
- `backend` — Storage backend for offloading conversation history.
- `trigger` — When to trigger compaction. Defaults to model-aware values.
- `keep` — How much of the recent context to retain after compaction. Defaults to model-aware values.
- `summary_prompt` — Prompt template for summarization.
- `truncate_args_settings` — Settings for argument pre-truncation.

### Behavior

When the conversation exceeds the `trigger` threshold:
1. Generates a summary of the messages that will be evicted.
2. Offloads the evicted messages to `backend.write()` at `/conversation_history/{thread_id}.md`.
3. Replaces the conversation history with the summary message and the retained recent messages.
4. Records a `SummarizationEvent` in private state.

## Class: `SummarizationToolMiddleware`

Wraps a `SummarizationMiddleware` instance and exposes a `compact_conversation` tool that triggers manual compaction.

### Constructor

```python
SummarizationToolMiddleware(summarization_middleware: SummarizationMiddleware)
```

### System Prompt

`SUMMARIZATION_SYSTEM_PROMPT` is injected when this middleware is active. It tells the agent to use `compact_conversation` when:
- The user moves to a completely new task.
- Previous working context is no longer needed.

### `wrap_model_call` / `awrap_model_call`
Adds `compact_conversation` tool to the request and injects the system prompt.

## Storage

Offloaded messages are stored as Markdown at:
```
/conversation_history/{thread_id}.md
```
Each summarization event appends a new section, creating a running log.

## Dependencies

- `langchain.agents.middleware.summarization` — base class and types
- `langchain_core.messages` — message types
- `langchain_core.exceptions.ContextOverflowError`
- `langgraph.config.get_config`
- `deepagents.middleware._utils.append_to_system_message`
- `deepagents.backends.protocol` — `BACKEND_TYPES`
