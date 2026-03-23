# `_session_stats.py`

## High-Level Purpose

This module provides lightweight session statistics and token count formatting utilities. It is intentionally kept free of heavy dependencies (no pydantic, no config, no widget imports) so that `app.py` can import `SessionStats` and `format_token_count` at module level without pulling in the full `textual_adapter` dependency tree.

## Type Aliases

| Alias | Type | Description |
|---|---|---|
| `SpinnerStatus` | `Literal["Thinking", "Offloading"] \| None` | Valid spinner display states, or `None` to hide |

## Classes

### `ModelStats`

**Type:** `dataclass`

Token statistics for a single model within a session.

| Attribute | Type | Description |
|---|---|---|
| `request_count` | `int` | Number of LLM API requests made to this model |
| `input_tokens` | `int` | Cumulative input tokens sent |
| `output_tokens` | `int` | Cumulative output tokens received |

### `SessionStats`

**Type:** `dataclass`

Statistics accumulated over a single agent turn or full session.

| Attribute | Type | Description |
|---|---|---|
| `request_count` | `int` | Total LLM API requests made |
| `input_tokens` | `int` | Cumulative input tokens across all requests |
| `output_tokens` | `int` | Cumulative output tokens across all requests |
| `wall_time_seconds` | `float` | Wall-clock duration from stream start to end |
| `per_model` | `dict[str, ModelStats]` | Per-model breakdown keyed by model name |

**Methods:**

#### `record_request(model_name: str, input_toks: int, output_toks: int) -> None`

Accumulates token counts for one completed LLM request. Updates both session totals and per-model breakdown.

**Parameters:**
- `model_name`: Model that served this request (used as per-model key). Pass an empty string to skip per-model tracking.
- `input_toks`: Input tokens for this request.
- `output_toks`: Output tokens for this request.

#### `merge(other: SessionStats) -> None`

Merges another `SessionStats` into `self` (mutates in place). Used to accumulate per-turn stats into a session-level total.

**Parameters:**
- `other`: The stats to fold in.

## Functions

### `format_token_count(n: int) -> str`

Formats a token count for display in the status bar.

**Returns:** Human-readable string (e.g., `"1.2k"`, `"45.3k"`, `"1.2M"`).

## Important Imports and Dependencies

This module uses only the Python standard library (`dataclasses`, `typing`). No third-party imports.
