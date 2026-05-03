# `deepagents/middleware/summarization.py`

## High-Level Purpose

`SummarizationMiddleware` automatically compacts conversation history when the context window fills up. It replaces old messages with an LLM-generated summary, stores the summary in the backend, and re-injects it at the start of every subsequent model call. This lets agents run indefinitely long sessions without hitting token limits.

---

## Key Classes

### `SummarizationMiddleware(AgentMiddleware)`

**Constructor parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `trigger_fraction` | `float` | `0.85` | Compact when token usage exceeds this fraction of context window |
| `keep_fraction` | `float` | `0.10` | Keep this fraction of tokens (most recent messages) after compaction |
| `summary_storage_path` | `str` | `"/conversation_history/{thread_id}.md"` | Path to write summaries |
| `backend` | `BackendProtocol` | required | Backend used to store summaries |
| `model` | `BaseChatModel \| None` | `None` | Model used to generate summary (defaults to agent's model) |

### `SummarizationToolMiddleware(AgentMiddleware)`

Exposes a `compact_conversation` tool that the agent (or user via `/compact`) can call explicitly, without waiting for the automatic threshold. Useful for proactively managing context before a long task.

### `create_summarization_middleware(backend, **kwargs) → SummarizationMiddleware`

Convenience factory with sensible defaults.

---

## Compaction Process

When triggered:

1. **Identify old messages** — everything except the `keep_fraction` most recent messages
2. **Generate summary** — calls the LLM with the old messages and prompt: "Summarize this conversation history preserving all important facts, decisions, and context"
3. **Store summary** — writes to `summary_storage_path` (replacing any previous summary)
4. **Update state** — replaces old messages with a single `SystemMessage` containing a pointer to the summary and the summary text itself
5. **Log event** — adds a compaction event to state metadata (visible in trace)

On each subsequent model call, the summary is injected at the top of the system prompt.

---

## Token Counting

Uses LangChain's `count_tokens_approximately()`. This is model-agnostic and not perfectly precise, but accurate enough to trigger compaction before the model hits its limit.

The trigger is checked at the start of each model call (via `wrap_model_call()`). If usage is below threshold, the middleware is a no-op.

---

## Architecture Notes

**Why store the summary externally?** The summary file at `/conversation_history/{thread_id}.md` persists across sessions. If the agent resumes a thread after a long gap, the previous summary is still accessible even if the LangGraph state was reset.

**Summary quality:** The quality of compaction depends on the LLM. Using a weaker/faster model for summarization (via the `model` parameter) is a common optimization — summaries don't need frontier-level intelligence.

---

## See Also

- [README.md](README.md) — middleware stack
- [../graph.md](../graph.md) — passes `backend` to this middleware indirectly
