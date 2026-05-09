# `libs/deepagents/deepagents/profiles/provider/_openrouter.py`

> Built-in OpenRouter provider profile for version checks and attribution kwargs.

## Position in the system

This module is registered by the shared profile bootstrap under the
`"openrouter"` provider key. Its profile runs a pre-init dependency check and
uses a factory so environment variables are read at model-resolution time.

## Imports and module-level state

`OPENROUTER_MIN_VERSION` is the minimum `langchain-openrouter` version required
for app attribution support. `_OPENROUTER_APP_URL` and
`_OPENROUTER_APP_TITLE` are SDK defaults for OpenRouter attribution.
`_OPENROUTER_ALLOW_AZURE_ENV` names the opt-in variable that allows Azure back
into OpenRouter routing.

## Functions and classes

### `_openrouter_attribution_kwargs() -> dict[str, Any]`

Builds kwargs for `init_chat_model`. It injects `app_url` and `app_title` only
when `OPENROUTER_APP_URL` or `OPENROUTER_APP_TITLE` are absent from the
environment, so explicit env vars take precedence over SDK defaults. An empty
string still counts as set, allowing callers to suppress the default.

The function also injects `openrouter_provider={"ignore": ["azure"]}` unless
`DEEPAGENTS_OPENROUTER_ALLOW_AZURE` is truthy (`1`, `true`, `yes`, or `on`).
That default avoids OpenRouter's stateless `/responses` beta routing reasoning
model calls through Azure in a way that can break multi-turn reasoning item
lookup.

### `check_openrouter_version() -> None`

Checks the installed `langchain-openrouter` package version. Missing package is
ignored so LangChain can raise the normal missing-dependency error later, but a
present package below `0.2.0` raises `ImportError` with an install command.

Non-PEP-440 version strings are not rejected. The function logs a warning and
skips the comparison because a development build or fork might still work.

### `register() -> None`

Registers the provider-wide `"openrouter"` profile. The profile uses
`pre_init=lambda _spec: check_openrouter_version()` and
`init_kwargs_factory=_openrouter_attribution_kwargs`, so the version check runs
before model construction and attribution/provider-routing kwargs are computed
fresh for each resolution.

## Gotchas

Because attribution kwargs are factory-driven, environment variables are read
when a model is resolved, not when the module is imported or when bootstrap
runs.

