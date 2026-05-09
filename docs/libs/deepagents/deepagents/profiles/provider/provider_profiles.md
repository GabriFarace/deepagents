# `libs/deepagents/deepagents/profiles/provider/provider_profiles.py`

> Registry and merge logic for provider-specific model-construction defaults.

## Position in the system

This module is consumed by `deepagents._models.resolve_model()`. When user code
passes a string model spec, the resolver uses `apply_provider_profile()` to
collect profile kwargs and then passes the result into LangChain's
`init_chat_model`.

Provider profiles are separate from harness profiles. They only influence how a
chat model object is built: constructor kwargs, version checks, headers,
provider-specific base URLs, and other pre-construction behavior.

## Imports and module-level state

`validate_profile_key()` is shared with harness profiles. `_PROVIDER_PROFILES`
is the in-memory registry from provider/model keys to immutable
`ProviderProfile` objects. Built-ins are not loaded until
`_ensure_provider_profiles_loaded()` calls `_builtin_profiles`.

## Functions and classes

### `ProviderProfile`

Immutable dataclass describing how to construct a chat model. `init_kwargs`
stores static kwargs, `pre_init` optionally runs before model construction, and
`init_kwargs_factory` optionally computes kwargs at resolution time. Static
kwargs are useful for stable defaults such as `use_responses_api=True`;
factories are useful for env-var-dependent values such as OpenRouter
attribution headers.

The class is deliberately scoped to model construction. Prompt assembly,
middleware, tool visibility, and default subagent behavior live in
`HarnessProfile`.

#### `ProviderProfile.__post_init__(self) -> None`

Defensively copies `init_kwargs` and wraps it in `MappingProxyType`. A frozen
dataclass prevents rebinding `profile.init_kwargs`, but it does not prevent
mutating a dict that was passed in. This hook prevents both mutation through
the original caller-owned dict and direct mutation through the profile object.

### `_ensure_provider_profiles_loaded() -> None`

Imports and calls the shared lazy bootstrap. This keeps provider registry code
decoupled from the concrete built-in modules while ensuring built-ins and
plugins are present before lookup or public registration.

### `_register_provider_profile_impl(key: str, profile: ProviderProfile) -> None`

Internal registration primitive. It validates the key and either inserts the
profile or merges it on top of an existing registration. Callers are expected
to handle bootstrap coordination, which is why built-in modules can call this
function during bootstrap without recursively starting another bootstrap.

### `register_provider_profile(key: str, profile: ProviderProfile) -> None`

Public beta registration API. It first ensures built-ins and plugins are loaded
so a user registration under a built-in key layers on top of the built-in
profile instead of being overwritten later.

Registration is additive. Incoming `init_kwargs` override existing keys,
`pre_init` callbacks chain, and factories chain so both are invoked on each
resolution with the newer factory winning on shared keys.

### `get_provider_profile(spec: str) -> ProviderProfile | None`

Looks up a profile for a model spec. It returns an exact model registration
when present, falls back to the provider prefix for `provider:model` specs, and
merges exact-on-top-of-provider when both exist. Malformed specs return `None`
rather than accidentally matching a provider-wide default.

This helper is best for inspection. Construction paths should prefer
`apply_provider_profile()` because it also invokes `pre_init` and resolves
dynamic kwargs.

### `apply_provider_profile(spec: str, kwargs: Mapping[str, Any] | None = None, *, run_pre_init: bool = True) -> dict[str, Any]`

Composes the final kwargs for `init_chat_model`. The function copies
caller-supplied kwargs, looks up the profile, optionally runs `pre_init`, merges
static kwargs, merges factory kwargs, then places caller kwargs on top.

The precedence is important: profile defaults sit below explicit user/config
kwargs. If no profile matches, the function returns a fresh copy of the caller
kwargs unchanged.

### `_merge_provider_profiles(base: ProviderProfile, override: ProviderProfile) -> ProviderProfile`

Builds a new profile representing `override` layered on `base`. Static kwargs
merge with override winning. Two `pre_init` hooks chain in order and log which
side failed before re-raising. Two factories also chain in order and merge
their output with override winning.

## Flow walk-through

1. `resolve_model()` calls `apply_provider_profile(spec, kwargs)`.
2. The provider registry ensures lazy bootstrap is complete.
3. Lookup checks exact model key first, then provider key.
4. Any matching provider and exact profiles are merged.
5. `pre_init` runs unless suppressed.
6. Static profile kwargs, factory kwargs, and caller kwargs are merged into a
   fresh dict for `init_chat_model`.

## Gotchas

Factories run every resolution, not once at registration. If a factory reads an
environment variable, changing the environment before the next model resolution
can change the resulting kwargs.

