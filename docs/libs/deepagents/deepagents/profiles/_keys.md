# `libs/deepagents/deepagents/profiles/_keys.py`

> Shared registry-key validation for provider and harness profiles.

## Position in the system

Both profile registries accept either a provider-wide key such as `"openai"` or
an exact model key such as `"openai:gpt-5.4"`. This helper centralizes that
grammar so `register_provider_profile` and `register_harness_profile` reject
the same malformed input before mutating their registries.

## Functions and classes

### `validate_profile_key(key: str) -> None`

Validates the `provider` or `provider:model` shape used by profile
registration. The function mutates no state and returns `None` on success; all
useful behavior is in its exceptions.

The guard rejects empty keys, leading or trailing whitespace, multiple colons,
empty provider/model halves, and whitespace adjacent to the colon. Lookup
helpers are more permissive and return `None` for malformed specs, but
registration must fail loudly because a bad key would otherwise leave a
registry entry that never matches.

## Gotchas

The helper does not validate provider or model existence. It only validates the
string shape needed by the two-level lookup algorithm.

