# `libs/deepagents/deepagents/_models.py`

> Shared helpers for resolving model specs and extracting provider/model
> identity from `BaseChatModel` instances.

## Position in the system

`graph.py:create_deep_agent()` calls `resolve_model()` whenever the caller
passes a string model spec or a declarative subagent declares its own model.
The profile system also relies on `get_model_identifier()`,
`get_model_provider()`, and `model_matches_spec()` to decide which harness or
provider profile applies.

```
create_deep_agent()
  ├─ resolve_model("provider:model")
  │    └─ init_chat_model(..., **apply_provider_profile(...))
  └─ _harness_profile_for_model(...)
       ├─ get_model_provider(...)
       ├─ get_model_identifier(...)
       └─ model_matches_spec(...)
```

## Imports and module-level state

The file imports LangChain's `init_chat_model`, the abstract
`BaseChatModel`, and `apply_provider_profile()` from the provider-profile
registry. The only module state is a logger used when model-provider
introspection fails.

## Functions and classes

### `resolve_model(model)`

`resolve_model()` normalizes the model argument accepted by
`create_deep_agent()`. If the value is already a `BaseChatModel`, it returns
the object unchanged so caller-specified settings, callbacks, output versions,
and provider-specific constructor choices are preserved.

For string values, it calls `apply_provider_profile(model)` first and forwards
the returned keyword arguments into `init_chat_model()`. This is where
provider-wide defaults enter the system: built-in profiles can make OpenAI use
the Responses API or add OpenRouter attribution headers before LangChain
constructs the concrete chat model. The function does not catch import or
configuration errors; those should surface to the caller because the agent
cannot run without a usable model.

### `get_model_identifier(model)`

This helper returns the provider-native model id from a chat model instance. It
checks `model_name` first, then `model`, because providers disagree about which
attribute they expose. A missing or empty field returns `None` rather than
guessing.

The identifier is used by harness-profile matching and by
`model_matches_spec()`. It is intentionally model-name only; provider detection
is handled separately by `get_model_provider()`.

### `get_model_provider(model)`

This function asks the LangChain model for LangSmith parameters via
`model._get_ls_params()` and reads the `ls_provider` value. Most first-party
LangChain chat integrations fill this field with a stable provider name such
as `anthropic` or `openai`, which makes it more reliable than deriving a
provider from the Python class name.

The exception path is deliberately visible at INFO level. If a custom model
does not implement `_get_ls_params()` correctly, profile matching may silently
miss; logging the failure gives users a clue without requiring DEBUG logs. A
missing, falsey, or non-string provider returns `None`.

### `model_matches_spec(model, spec)`

`model_matches_spec()` answers "does this already-instantiated model correspond
to this string spec?" It first compares the full `spec` to the identifier. If
that fails, it splits on the first colon and compares only the model-name
portion. That lets `openai:gpt-5` match a model whose provider object reports
only `gpt-5`.

The function assumes the common `provider:model` convention and does not
attempt fuzzy matching or alias resolution. If the model has no identifier, it
returns `False` so callers can fall back to provider-level matching or default
profiles.

### `_string_attr(obj, attr)`

Private helper that reads an attribute and returns it only if it is a non-empty
string. It keeps `get_model_identifier()` compact and avoids treating provider
objects, empty strings, or missing attributes as valid model ids.

## Flow walk-through

1. `graph.py:458` records the original string model spec, if there was one.
2. `graph.py:460` handles the deprecated `None` model path; otherwise
   `graph.py:478` calls `resolve_model()`.
3. `resolve_model()` either returns a model object unchanged or calls
   `init_chat_model()` with provider-profile kwargs.
4. `graph.py:479` passes the resolved model and optional original spec into
   harness-profile selection.

## Gotchas

Passing a prebuilt `BaseChatModel` bypasses provider-profile constructor
defaults because the object is already constructed. That is intentional: use a
model object when you want exact control over provider options.
