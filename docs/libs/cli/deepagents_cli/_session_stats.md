# `libs/cli/deepagents_cli/_session_stats.py`

> Lightweight session statistics and token formatting utilities.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `ModelStats`

Token stats for a single model within a session.

Additional notes from the source docstring:

```text
Attributes:
    request_count: Number of LLM API requests made to this model.
    input_tokens: Cumulative input tokens sent to this model.
    output_tokens: Cumulative output tokens received from this model.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `SessionStats`

Stats accumulated over a single agent turn (or full session).

Additional notes from the source docstring:

```text
Attributes:
    request_count: Total LLM API requests made (each chunk with
        usage_metadata counts as one completed request).
    input_tokens: Cumulative input tokens across all LLM requests.
    output_tokens: Cumulative output tokens across all LLM requests.
    wall_time_seconds: Wall-clock duration from stream start to end.
    per_model: Per-model breakdown keyed by model name.
        Populated only when `record_request` receives a non-empty
        `model_name`. Empty dict means no named-model requests were
        recorded; `print_usage_table` omits the model table in that case and
        shows only the wall-time line (if applicable).
```

Methods worth reading inside this class:

- `record_request(self, model_name: str, input_toks: int, output_toks: int)`: Accumulate token counts for one completed LLM request.

- `merge(self, other: SessionStats)`: Merge another `SessionStats` into this one (mutates *self*).


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `format_token_count(count: int)`

Format a token count into a human-readable short string.

Additional notes from the source docstring:

```text
Args:
    count: Number of tokens.

Returns:
    Formatted string like `'12.5K'`, `'1.2M'`, or `'500'`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
