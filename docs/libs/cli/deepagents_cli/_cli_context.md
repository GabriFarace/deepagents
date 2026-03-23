# `_cli_context.py`

## High-Level Purpose

This module defines a lightweight runtime context type for CLI model overrides. It is extracted from `configurable_model` so that hot-path modules (`app.py`, `textual_adapter.py`) can import `CLIContext` without pulling in the full LangChain middleware stack.

## Classes

### `CLIContext`

**Type:** `TypedDict(total=False)`

Runtime context passed via `context=` to the LangGraph graph. Carries per-invocation overrides that `ConfigurableModelMiddleware` reads from `request.runtime.context`.

| Key | Type | Description |
|---|---|---|
| `model` | `str \| None` | Model spec to swap at runtime (e.g., `'openai:gpt-4o'`) |
| `model_params` | `dict[str, Any]` | Invocation params (e.g., `temperature`, `max_tokens`) to merge into `model_settings` |

## Usage

```python
from deepagents_cli._cli_context import CLIContext

context: CLIContext = {
    "model": "anthropic:claude-opus-4-6",
    "model_params": {"temperature": 0.7},
}
```

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `typing_extensions.TypedDict` | `typing-extensions` | `TypedDict` with `total=False` support |
