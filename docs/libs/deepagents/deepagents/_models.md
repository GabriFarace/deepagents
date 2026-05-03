# `libs/deepagents/deepagents/_models.py`

## High-Level Purpose

`_models.py` resolves model specifications (strings like `"claude-sonnet-4-6"` or `"openai:gpt-4o"`) into `BaseChatModel` instances. It handles provider-specific profile injection (e.g., setting OpenAI Responses API defaults, adding OpenRouter headers) and provides utility functions for extracting provider/identifier metadata from a model object.

---

## Key Functions

### `resolve_model(model) → BaseChatModel`

The main entry point. Accepts either a string spec or an already-constructed `BaseChatModel`.

- If `model` is already a `BaseChatModel`: returns it unchanged
- If `model` is a string: calls `langchain.chat_models.init_chat_model(model)` then applies provider profiles

**String format:** `"provider:model_id"` (e.g., `"anthropic:claude-sonnet-4-6"`) or bare `"model_id"` (provider auto-detected).

**Provider profiles applied after resolution:**

| Provider | Profile behavior |
|---|---|
| `openai` | Enables Responses API defaults (streaming, tool use mode) |
| `openrouter` | Injects required `HTTP-Referer` and `X-Title` headers |
| Others | No profile applied |

### `get_model_identifier(model) → str | None`

Returns the model's string identifier by checking `model.model_name` then `model.model` attributes. Returns `None` if neither is found.

### `get_model_provider(model) → str | None`

Returns the provider name (e.g., `"anthropic"`, `"openai"`) by calling `model._get_ls_params()`. Logs at INFO level (not DEBUG) if `_get_ls_params()` is unavailable, so users can see when provider detection fails.

### `model_matches_spec(model, spec) → bool`

Returns `True` if `model` matches `spec`. Checks:
1. Exact string equality on the model identifier
2. `"provider:model"` suffix match (e.g., `spec="anthropic:claude-sonnet-4-6"` matches a model with provider `"anthropic"` and identifier `"claude-sonnet-4-6"`)

---

## Architecture Notes

**Provider detection:** LangChain's `_get_ls_params()` is a private method used internally for LangSmith trace metadata. Using it for provider detection is a pragmatic choice — there's no public equivalent. The INFO-level log on failure ensures visibility without verbose debug output.

**`init_chat_model` dependency:** The SDK delegates all model construction to LangChain's `init_chat_model()`. This means any LangChain-supported provider works out-of-the-box, and the SDK doesn't need to maintain its own provider registry.

---

## See Also

- [graph.md](graph.md) — passes `model` parameter to `resolve_model()`
