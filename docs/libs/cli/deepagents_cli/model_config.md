# `model_config.py`

## High-Level Purpose

This module manages model configuration for the CLI. It handles:

- Parsing and validating `provider:model` format model specifications
- Loading and saving model configuration from/to `~/.deepagents/config.toml`
- Discovering available models from installed LangChain provider packages
- Managing default model persistence
- Checking provider credentials
- Loading model profiles (token limits, capabilities, etc.)
- Checking whether warnings are suppressed in config

## Classes

### `ModelConfigError`

Custom exception raised when model configuration or creation fails.

### `ModelSpec`

**Type:** `dataclass(frozen=True)`

A model specification in `provider:model` format.

| Attribute | Type | Description |
|---|---|---|
| `provider` | `str` | The provider name (e.g., `'anthropic'`, `'openai'`) |
| `model` | `str` | The model identifier (e.g., `'claude-sonnet-4-5'`, `'gpt-4o'`) |

**Methods:**

- `parse(spec: str) -> ModelSpec` — Class method that parses `'provider:model'` string. Raises `ValueError` if format is invalid.
- `try_parse(spec: str) -> ModelSpec | None` — Non-raising variant; returns `None` for invalid specs.
- `__str__() -> str` — Returns `'provider:model'` formatted string.
- `__post_init__()` — Validates that neither provider nor model is empty.

**Example:**
```python
spec = ModelSpec.parse("anthropic:claude-sonnet-4-5")
spec.provider  # 'anthropic'
spec.model     # 'claude-sonnet-4-5'
str(spec)      # 'anthropic:claude-sonnet-4-5'
```

### `ModelProfileEntry`

**Type:** `TypedDict`

Profile data for a model with override tracking.

| Key | Type | Description |
|---|---|---|
| `profile` | `dict[str, Any]` | Merged profile dict (defaults + config.toml overrides) |
| `overridden_keys` | `frozenset[str]` | Keys whose values came from config.toml |

### `ProviderConfig`

**Type:** `TypedDict(total=False)`

Configuration for a model provider read from `config.toml`.

| Key | Type | Description |
|---|---|---|
| `enabled` | `bool` | Whether this provider appears in the model switcher |
| `models` | `list[str]` | List of model identifiers |
| `api_key_env` | `str` | Environment variable name for the API key |
| `base_url` | `str` | Custom base URL |
| `class_path` | `str` | Full import path to a custom `BaseChatModel` subclass |

> **Warning:** `class_path` executes arbitrary Python code from the user's config file.

### `ModelConfig`

**Type:** `TypedDict`

Full model configuration loaded from `config.toml`.

## Module-Level Constants

| Constant | Description |
|---|---|
| `DEFAULT_CONFIG_DIR` | `Path("~/.deepagents")` — user config directory |
| `DEFAULT_CONFIG_PATH` | `Path("~/.deepagents/config.toml")` — default config file path |

## Functions

### `is_warning_suppressed(warning_name: str, config_path: Path | None = None) -> bool`

Checks whether a named warning is suppressed via `[warnings].suppress` in `config.toml`.

**Parameters:**
- `warning_name`: Warning identifier (e.g., `'ripgrep'`).
- `config_path`: Optional config file path.

**Returns:** `True` if the warning name is in the suppress list.

### `get_available_models(config_path: Path | None = None) -> dict[str, list[str]]`

Discovers available models from installed LangChain provider packages, merged with user configuration from `config.toml`. The config file can add providers, add models to existing providers, or set `enabled: false` to hide providers.

**Returns:** `dict[provider_name, list[model_id]]`.

### `get_model_profiles(spec_list: list[str]) -> dict[str, ModelProfileEntry]`

Loads profiles for a list of model specs. Profiles include token limits, tool-calling capability, and other model attributes.

**Returns:** `dict[model_spec_str, ModelProfileEntry]`.

### `save_default_model(spec: str) -> bool`

Persists a model spec as the default model in `config.toml`. Uses atomic write (temp file + rename).

**Returns:** `True` on success, `False` on error.

### `clear_default_model() -> bool`

Removes the default model from `config.toml`.

**Returns:** `True` on success, `False` on error.

### `has_provider_credentials(provider: str) -> bool | None`

Checks whether the required API key environment variable for a provider is set.

**Returns:** `True` if credentials are confirmed present, `False` if missing, `None` if unknown.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `tomllib` | stdlib (3.11+) | Config TOML reading |
| `tomli_w` | `tomli-w` package | Config TOML writing |
| `importlib.util` | stdlib | Dynamic provider package discovery |
| `MappingProxyType` | stdlib | Immutable config wrappers |
