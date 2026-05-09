# `libs/deepagents/deepagents/profiles/provider/_openai.py`

> Built-in provider profile for OpenAI chat models.

## Position in the system

`_builtin_profiles._ensure_builtin_profiles_loaded()` calls this module's
`register()` function during provider bootstrap. The profile is keyed to the
provider-wide `"openai"` entry, so it applies to `openai:*` model specs unless
an exact model profile overrides the same fields.

## Functions and classes

### `register() -> None`

Registers `ProviderProfile(init_kwargs={"use_responses_api": True})` under the
`"openai"` key via the internal registration primitive. This means OpenAI
models use the Responses API by default when `resolve_model()` constructs them.

The function mutates only the provider-profile registry. User code can layer on
top of this default with `register_provider_profile("openai", ...)`.

## Gotchas

The built-in uses `_register_provider_profile_impl()` rather than the public
registration helper because bootstrap coordination is already being handled by
`_builtin_profiles.py`.

