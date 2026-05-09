# `libs/deepagents/deepagents/middleware/summarization.py`

> Conversation compaction middleware. It summarizes old messages, offloads full
> history to a backend, tracks compaction events without deleting raw state, and
> optionally exposes a `compact_conversation` tool.

## Position in the system

Deep agents can run long enough to exceed provider context windows. This module
adds two layers:

```
automatic layer:
  SummarizationMiddleware.wrap_model_call()
    ├─ reconstruct effective history from prior summary event
    ├─ optionally truncate old write/edit tool arguments
    ├─ summarize old messages when thresholds are crossed
    ├─ offload full old history to backend
    └─ update private _summarization_event

manual layer:
  SummarizationToolMiddleware
    ├─ exposes compact_conversation
    └─ reuses the same summarization engine and state key
```

`create_deep_agent()` uses these pieces to keep the model request small while
preserving the raw `state["messages"]` log for replay, evals, and recovery.

## Imports and module-level state

The module wraps LangChain's own `SummarizationMiddleware` helper for token
counting, cutoff selection, and summary generation. Deep Agents adds backend
offload, `ContextOverflowError` retry behavior, pre-summary tool-argument
truncation, and manual compaction tools.

Important constants include `SUMMARIZATION_SYSTEM_PROMPT`, the compact-tool
prompt nudge, and the imported LangChain defaults for messages-to-keep and
trim limits. `SummarizationMiddleware` is a public alias for the private
implementation class `_DeepAgentsSummarizationMiddleware`.

## Functions and classes

### `CompactConversationSchema`

Empty Pydantic schema for the `compact_conversation` tool. The current tool
creation leaves explicit schema wiring commented out, so the tool takes no
model arguments through inferred schema behavior.

### `SummarizationEvent`

TypedDict stored privately under `_summarization_event`. It records the
absolute cutoff index in raw state, the generated summary `HumanMessage`, and
the backend file path where old conversation history was written, if offload
succeeded.

### `TruncateArgsSettings`

Configuration for a lightweight pre-compaction step. It can trigger based on
the same context-size formats as summarization, keep recent messages intact,
and replace long argument values with a short prefix plus truncation text.

Only old `write_file` and `edit_file` tool-call arguments are truncated. This
often reduces context pressure without summarizing the conversation.

### `SummarizationState`

Extends `AgentState` with private `_summarization_event`. The event is private
because it is middleware bookkeeping, not user-facing output.

### `SummarizationDefaults`

TypedDict returned by `compute_summarization_defaults()`. It packages
automatic trigger, keep policy, and argument-truncation settings.

### `compute_summarization_defaults(model)`

Chooses default compaction settings from the resolved chat model. If
`model.profile["max_input_tokens"]` exists and is an integer, the defaults are
fraction-based: summarize at 85 percent of context and keep 10 percent. If the
profile lacks a max input window, the function falls back to conservative
fixed thresholds: 170,000 tokens, keep six messages, and truncate old args
after 20 messages.

### `_DeepAgentsSummarizationMiddleware`

The implementation behind the public `SummarizationMiddleware` alias. It
contains a LangChain summarization helper plus Deep Agents state/offload logic.
Its `serialized_name` and `name` property report `SummarizationMiddleware` so
middleware exclusion by class name or string targets the public alias.

#### `__init__(model, backend, trigger, keep, token_counter, summary_prompt, trim_tokens_to_summarize, truncate_args_settings, **deprecated_kwargs)`

Initializes the LangChain helper with model, trigger, keep, token counter, and
summary prompt settings, then stores the backend and computes the history path
prefix. For `CompositeBackend`, history is written under its `artifacts_root`;
otherwise the default prefix is `/conversation_history`.

The deprecated `history_path_prefix` kwarg still works with a deprecation
warning and overrides the computed prefix. `truncate_args_settings` is unpacked
into private trigger/keep/max-length fields; when absent, argument truncation
is disabled but defaults are still stored.

#### Delegated helper methods and properties

`model`, `token_counter`, `_get_profile_limits()`, `_should_summarize()`,
`_determine_cutoff_index()`, `_partition_messages()`, `_create_summary()`, and
`_acreate_summary()` delegate to the LangChain helper. Deep Agents keeps these
methods available because the rest of the file, and the manual compaction
tool, need a single summarization engine.

#### `_get_backend(state, runtime)`

Resolves the backend. Direct instances are returned unchanged. Callable
backend factories receive a synthetic `ToolRuntime` built from current state,
runtime context, store, stream writer, and runtime config.

#### `_get_thread_id()`

Reads LangGraph's current runnable config via `get_config()` and returns
`configurable.thread_id` when available. Outside a runnable context, it
generates a fallback ID like `session_a1b2c3d4`. The thread ID determines the
history offload file.

#### `_get_history_path()`

Returns `{history_path_prefix}/{thread_id}.md`, producing one append-only
conversation-history file per thread.

#### `_is_summary_message(msg)` and `_filter_summary_messages(messages)`

Identify and remove previous summary `HumanMessage` objects before offloading.
This prevents chained summarization from repeatedly storing summaries whose
underlying original messages were already offloaded.

#### `_build_new_messages_with_path(summary, file_path)`

Builds the summary `HumanMessage` inserted at the front of the effective model
history. If offload succeeded, the message tells the model where the full
conversation history was saved and wraps the summary in `<summary>` tags. If
offload failed, it emits a simpler "summary to date" message. The message is
tagged with `additional_kwargs={"lc_source": "summarization"}`.

#### `_get_effective_messages(request)`

Applies any prior summarization event in request state to the raw message list.
This is the entry point used by automatic compaction before token counting.

#### `_apply_event_to_messages(messages, event)`

Static helper that reconstructs the effective conversation. With no event, it
returns a copy of the raw messages. With an event, it returns the stored
summary message followed by raw messages from `cutoff_index` onward. Malformed
events or out-of-bounds cutoffs are logged and handled defensively.

#### `_compute_state_cutoff(event, effective_cutoff)`

Translates a cutoff index in the effective message list back to an absolute
index in raw state. When a prior event exists, the effective list starts with a
synthetic summary message, so the method adds the prior raw cutoff and
subtracts one.

#### `_should_truncate_args(messages, total_tokens)`

Checks the configured argument-truncation trigger. Message and token triggers
compare directly. Fraction triggers require a model profile max input token
limit; without one, fraction-based argument truncation does not run.

#### `_determine_truncate_cutoff_index(messages)`

Determines which older messages are eligible for argument truncation. A
message-count keep policy preserves the most recent N messages. Token and
fraction policies walk backward from the end until the retained messages fit
the target token budget. Messages before the returned index may be truncated.

#### `_truncate_tool_call(tool_call)`

Returns a copied tool call whose string argument values longer than the
configured max length are replaced by the first 20 characters plus the
truncation suffix. If no argument changes, it returns the original tool call.

#### `_truncate_args(messages, system_message, tools)`

Counts tokens for the effective request, decides whether argument truncation
should run, computes a cutoff, and rewrites old `AIMessage.tool_calls` for
`write_file` and `edit_file` only. It returns `(messages, modified)`, where the
message list is copied only when a change is needed.

#### `_offload_to_backend(backend, messages)` and `_aoffload_to_backend(backend, messages)`

Append the messages being summarized to the per-thread history markdown file.
Each event adds a timestamped section containing `get_buffer_string()` output.
The methods use `download_files()`/`adownload_files()` to fetch raw existing
content, then `edit()`/`aedit()` or `write()`/`awrite()` to persist the
combined file.

Offload failure is non-fatal and returns `None`; summarization can still
proceed, but older raw messages may not be recoverable from backend storage.

#### `wrap_model_call(request, handler)`

Automatic sync compaction hook. It reconstructs effective messages, truncates
old write/edit args if configured, counts tokens, and if summarization is not
needed it calls the model once with the truncated messages. If that call raises
`ContextOverflowError`, it falls through into the summarization path.

When summarization runs, it chooses a cutoff, partitions messages, offloads the
summarized portion, generates a summary, builds a new summary message, computes
the raw-state cutoff, calls the model with summary plus preserved messages, and
returns an `ExtendedModelResponse` that writes the new `_summarization_event`.
It does not overwrite `state["messages"]`.

#### `awrap_model_call(request, handler)`

Async automatic compaction hook. It mirrors `wrap_model_call()` but awaits the
handler and runs offload plus summary generation concurrently with
`asyncio.gather()` because those operations are independent.

### `SummarizationMiddleware`

Public alias for `_DeepAgentsSummarizationMiddleware`. External callers should
import and configure this name.

### `create_summarization_middleware(model, backend)`

Factory for an automatic `SummarizationMiddleware` with model-aware defaults.
It requires an already resolved `BaseChatModel`, computes defaults with
`compute_summarization_defaults()`, disables summary-input trimming by passing
`trim_tokens_to_summarize=None`, and enables the default argument-truncation
settings.

Passing a model string here is an error; use
`create_summarization_tool_middleware()` for string resolution.

### `create_summarization_tool_middleware(model, backend)`

Convenience factory for the manual tool layer. If `model` is a string, it
resolves it with `deepagents._models.resolve_model()`, creates an automatic
summarization engine with `create_summarization_middleware()`, and wraps that
engine in `SummarizationToolMiddleware`.

The returned value is only the tool middleware. Registering it gives the agent
the `compact_conversation` tool and prompt nudge; automatic compaction still
requires a `SummarizationMiddleware` in the stack.

### `SummarizationToolMiddleware`

Provides the `compact_conversation` structured tool and a system-prompt nudge.
It reuses a `_DeepAgentsSummarizationMiddleware` instance for backend
resolution, cutoff logic, offload, and summary generation. Manual and
automatic compaction share `_summarization_event`, so either path can build on
the other's latest summary.

#### `__init__(summarization)`

Stores the summarization engine and creates the single tool with
`_create_compact_tool()`. Its state schema is `SummarizationState`.

#### `_resolve_backend(runtime)`

Resolves the summarization engine's backend for tool execution. Callable
backends receive the actual `ToolRuntime` from the compact tool call.

#### `_create_compact_tool()`

Builds the `compact_conversation` `StructuredTool` with sync and async
implementations. The tool description says it takes no arguments and should be
used proactively when the conversation is getting long.

#### `_build_compact_result(runtime, to_summarize, summary, file_path, event, cutoff)`

Constructs the successful `Command` from a manual compact call. It builds a
new summary message, translates the cutoff to raw state, updates
`_summarization_event`, and appends a confirmation `ToolMessage` naming how
many messages were summarized.

#### `_nothing_to_compact(tool_call_id)`

Returns a `Command` with a `ToolMessage` explaining that the conversation is
within token budget. It is used when reported usage is below the manual
eligibility gate or cutoff selection finds no compactable prefix.

#### `_compact_error(tool_call_id, exc)`

Converts compaction exceptions into a `ToolMessage` instead of raising out of
the tool node. The message includes exception type and message and states that
no messages were summarized or removed.

#### `_is_eligible_for_compaction(messages)`

Checks whether the manual tool is allowed to compact yet. It requires reported
token usage to reach roughly 50 percent of the automatic trigger. Token
triggers use half of the token threshold; fraction triggers use half of the
fraction of model max input tokens. If no trigger conditions exist, the tool is
not eligible.

#### `_run_compact(runtime)` and `_arun_compact(runtime)`

Sync and async tool implementations. They reconstruct effective messages from
raw state and prior event, check eligibility, determine cutoff, partition old
messages, create a summary, offload the old history, and return the compact
result command. Exceptions are logged and converted into `_compact_error()`.

#### `wrap_model_call(request, handler)` and `awrap_model_call(request, handler)`

Append `SUMMARIZATION_SYSTEM_PROMPT` to the system message so the model knows
when it may call `compact_conversation`. These hooks do not trigger compaction
themselves.

## System prompts and tool descriptions

`SUMMARIZATION_SYSTEM_PROMPT`:

```text
## Compact conversation Tool `compact_conversation`

You have access to a `compact_conversation` tool. This tool refreshes your context window to reduce context bloat and costs.

You should use the tool when:
- The user asks to move on to a completely new task for which previous context is likely irrelevant.
- You have finished extracting or synthesizing a result and previous working context is no longer needed.
```

`compact_conversation` tool description:

```text
Compact the conversation by summarizing older messages into a concise summary. Use this proactively when the conversation is getting long to free up context window space. This tool takes no arguments.
```

Summary message template when history offload succeeds:

```text
You are in the middle of a conversation that has been summarized.

The full conversation history has been saved to {file_path} should you need to refer back to it for details.

A condensed summary follows:

<summary>
{summary}
</summary>
```

Summary message template when history offload fails:

```text
Here is a summary of the conversation to date:

{summary}
```

## Flow walk-through

1. `wrap_model_call()` starts from raw state messages and applies any previous
   `_summarization_event`.
2. Old `write_file`/`edit_file` arguments may be truncated before token
   counting.
3. If below threshold, the model is called normally; provider
   `ContextOverflowError` forces the summarization path.
4. The middleware partitions messages into old-to-summarize and recent-to-keep.
5. Old messages are appended to `/conversation_history/{thread_id}.md` or the
   composite backend's artifacts root.
6. A summary message replaces the old prefix only in the model request, while
   raw state remains intact.
7. The model response is wrapped in `ExtendedModelResponse` so LangGraph stores
   the new private summary event.

## Gotchas

Summarization does not delete old messages from `state["messages"]`. The model
sees an effective compacted history, while raw state keeps the full transcript.

Manual compaction eligibility uses reported token usage through LangChain's
helper. If usage metadata is absent, `compact_conversation` may say there is
nothing to compact even when approximate token counting would be high.

Offload failure warns but does not block summarization. In that case the model
gets a summary, but the full evicted history has no backend recovery path.
