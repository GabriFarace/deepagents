# `deepagents/_models.py`

## High-Level Purpose

Shared helpers for resolving and inspecting LangChain chat models. This module centralizes model resolution logic, providing consistent handling of model strings (e.g., `"openai:gpt-4o"`) and already-instantiated `BaseChatModel` objects. It is used by `graph.py` when setting up the main agent and subagents.

## Dependencies

- `langchain.chat_models.init_chat_model` — used to instantiate a model from a provider/model string
- `langchain_core.language_models.BaseChatModel` — base class for all LangChain chat models

## Functions

### `resolve_model(model: str | BaseChatModel) -> BaseChatModel`

Resolves a model identifier to a `BaseChatModel` instance.

**Parameters:**
- `model` — Either a `BaseChatModel` instance (returned as-is) or a provider-prefixed string like `"openai:gpt-4o"` or `"anthropic:claude-sonnet-4-6"`.

**Returns:** A `BaseChatModel` instance ready for use.

**Key Logic:**
- If `model` is already a `BaseChatModel`, returns it unchanged.
- If `model` starts with `"openai:"`, initializes via `init_chat_model(..., use_responses_api=True)`, opting into the OpenAI Responses API by default.
- For all other strings, delegates to `init_chat_model(model)` which handles provider prefix resolution.

---

### `get_model_identifier(model: BaseChatModel) -> str | None`

Extracts the provider-native model identifier string (e.g., `"gpt-4o"`, `"claude-sonnet-4-6"`) from a `BaseChatModel` instance.

**Parameters:**
- `model` — A `BaseChatModel` instance.

**Returns:** The model's identifier string, or `None` if it cannot be determined.

**Key Logic:**
- Calls `model.model_dump()` to get the serialized config.
- Checks for `"model_name"` first (used by Anthropic), then `"model"` (used by OpenAI and others).
- Returns the first non-empty string found, or `None`.

---

### `model_matches_spec(model: BaseChatModel, spec: str) -> bool`

Checks whether a `BaseChatModel` instance already corresponds to a given model spec string. Used to avoid re-initializing a model that is already configured correctly.

**Parameters:**
- `model` — A `BaseChatModel` instance.
- `spec` — A model spec string in `"provider:model-name"` format (e.g., `"openai:gpt-5"`).

**Returns:** `True` if the model's identifier matches the spec, `False` otherwise.

**Key Logic:**
- Gets the current model identifier via `get_model_identifier`.
- Performs two match attempts:
  1. Exact equality: `spec == current`
  2. Model-name-only match: strips the `"provider:"` prefix from `spec` and compares the remainder to `current`. For example, `"openai:gpt-5"` matches a model with identifier `"gpt-5"`.

---

### `_string_value(config: dict, key: str) -> str | None` (private)

Helper that safely retrieves a non-empty string from a dict.

**Parameters:**
- `config` — Dictionary (typically from `model.model_dump()`).
- `key` — Key to look up.

**Returns:** The string value if present and non-empty, otherwise `None`.
