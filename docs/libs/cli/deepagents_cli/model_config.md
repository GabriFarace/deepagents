# `libs/cli/deepagents_cli/model_config.py`

> Provider/model catalog, credential status, defaults/recent selections, and
> thread-list preferences.

## Position in the system

Model selectors, `main.py`, and `config.py:create_model()` use this module to
load available model profiles and persist user choices.

## Functions and classes

### Credential helpers

`resolved_env_var_name()`, `resolve_env_var()`,
`resolve_provider_credential()`, `get_provider_auth_status()`,
`has_provider_credentials()`, `get_credential_env_var()`, and
`apply_stored_credentials()` normalize auth state.

### Catalog types and loaders

`ModelSpec`, `ModelProfileEntry`, `ProviderConfig`, `_get_builtin_providers()`,
`_load_provider_profiles()`, `get_available_models()`, and
`get_model_profiles()` build the model catalog.

### `ModelConfig`

TOML-backed configuration object for model/provider choices.

### Persistence helpers

Default/recent model and agent helpers, warning suppression helpers, and thread
column/sort/time preference helpers edit focused TOML fields.

## Gotchas

Use structured auth status helpers instead of checking a single environment
variable; local endpoints and stored credentials complicate the answer.
