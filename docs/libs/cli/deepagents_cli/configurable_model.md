# `libs/cli/deepagents_cli/configurable_model.py`

> Middleware for applying runtime model overrides from LangGraph config.

## Position in the system

Installed in the CLI agent stack so model settings can change per run without
rebuilding the whole graph.

## Functions and classes

### `_is_anthropic_model(model)`

Detects Anthropic models for provider-specific override behavior.

### `_apply_overrides(request)`

Reads configurable values from the model request and returns an overridden
request.

### `ConfigurableModelMiddleware`

Wraps sync/async model calls and applies overrides before delegating.

## Gotchas

This affects one request path; persisted user defaults live in
`model_config.py`.
