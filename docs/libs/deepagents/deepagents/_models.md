# `deepagents/_models.py`

## High-Level Purpose

Shared helpers for resolving and inspecting LangChain chat models, and the **Profiles API** for registering provider-specific behaviors. This module centralizes model resolution logic, providing consistent handling of model strings (e.g., `"openai:gpt-4o"`) and already-instantiated `BaseChatModel` objects. It is used by `graph.py` when setting up the main agent and subagents.

## Dependencies

- `langchain.chat_models.init_chat_model` — instantiates models from provider/model strings
- `langchain_core.language_models.BaseChatModel` — base class for all LangChain chat models
- `deepagents.profiles.provider.provider_profiles` — built-in provider profile registry

## Functions

### `resolve_model(model: str | BaseChatModel) -> BaseChatModel`

Resolves a model identifier to a `BaseChatModel` instance.

**Parameters:**
- `model` — A `BaseChatModel` instance (returned as-is) or a provider-prefixed string like `"openai:gpt-4o"` or `"anthropic:claude-sonnet-4-6"`.

**Returns:** A `BaseChatModel` instance ready for use.

**Key Logic:**
- If `model` is already a `BaseChatModel`, returns it unchanged.
- For string specs, delegates to `init_chat_model(model)` with provider prefix resolution.
- After initialization, calls `apply_provider_profile(model)` to compose any registered provider-specific behavior (e.g., OpenAI Responses API opt-in, custom headers).

---

### `get_model_identifier(model: BaseChatModel) -> str | None`

Extracts the provider-native model identifier string (e.g., `"gpt-4o"`, `"claude-sonnet-4-6"`) from a `BaseChatModel` instance.

**Key Logic:**
- Tries `model.model_name` attribute first (used by Anthropic), then falls back to `model.model` attribute (used by OpenAI and others).
- Returns the first non-empty string found, or `None`.

---

### `get_model_provider(model: BaseChatModel) -> str | None`

Extracts the provider name (e.g., `"anthropic"`, `"openai"`) from a `BaseChatModel` instance.

**Key Logic:**
- Calls `model._get_ls_params()` and reads the `"ls_provider"` key.
- Logs an `INFO`-level message on failure (not `DEBUG`) so profile resolution misses surface in logs without requiring verbose logging.

---

### `model_matches_spec(model: BaseChatModel, spec: str) -> bool`

Checks whether a `BaseChatModel` instance corresponds to a given model spec string.

**Parameters:**
- `model` — A `BaseChatModel` instance.
- `spec` — A model spec string in `"provider:model-name"` format (e.g., `"openai:gpt-5"`).

**Returns:** `True` if the model matches the spec, `False` otherwise.

**Key Logic:**
- Performs two match attempts:
  1. Exact equality between `spec` and the model's current identifier.
  2. Model-name-only match: strips the `"provider:"` prefix from `spec` and compares the remainder. For example, `"openai:gpt-5"` matches a model with identifier `"gpt-5"`.

---

### `apply_provider_profile(model: BaseChatModel) -> BaseChatModel`

Applies any registered `ProviderProfile` behaviors to the model instance.

**Key Logic:**
- Looks up the model's provider via `get_model_provider()`.
- If a profile is registered for that provider, calls the profile's transform function (e.g., to enable `use_responses_api=True` for OpenAI, or inject custom HTTP headers for OpenRouter).
- Returns the (possibly modified) model instance.

---

### `register_provider_profile(provider: str, profile: ProviderProfile) -> None`

Registers a custom `ProviderProfile` for a provider name.

**Use case:** Extend the framework with custom provider behaviors (e.g., extra headers, API variants) without modifying core code.

---

## Profiles API

The Profiles API allows registering per-provider behaviors that are automatically applied whenever `resolve_model()` initializes a model for that provider.

### Built-in Provider Profiles

| Provider | Behavior |
|---|---|
| `"openai"` | Enables `use_responses_api=True` (OpenAI Responses API) |
| `"openrouter"` | Injects app attribution headers for OpenRouter routing |

### `ProviderProfile`

A callable (or class with `__call__`) that accepts a `BaseChatModel` and returns a (possibly modified) `BaseChatModel`. Registered via `register_provider_profile()`.

**Example:**

```python
from deepagents._models import register_provider_profile

def my_provider_profile(model):
    # Add custom headers or modify model config
    return model.bind(extra_headers={"X-Custom": "value"})

register_provider_profile("myprovider", my_provider_profile)
```
