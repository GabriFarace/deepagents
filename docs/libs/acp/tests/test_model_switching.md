# `tests/test_model_switching.py`

## High-Level Purpose

Tests for the model switching feature of `AgentServerACP`. Verifies that sessions expose model configuration options, that switching models updates session state and triggers agent recreation, that invalid model values raise errors, and that mode + model config options are correctly combined.

## Classes

### `MockClient`

A simple async-compatible test double for the ACP `Client`. Both `session_update` and `request_permission` are no-ops.

## Fixtures

| Fixture | Description |
|---------|-------------|
| `agent_factory` | Returns a factory function `build_agent(context)` that creates a `create_deep_agent` using `context.model` (falling back to `"anthropic:claude-sonnet-4"`) and `FilesystemBackend` in `virtual_mode`. |
| `models` | Returns a list of 3 model dicts: Claude Opus 4, Claude Sonnet 4, Claude Haiku 4. |

## Test Functions

| Test | What It Verifies |
|------|-----------------|
| `test_new_session_returns_config_options` | `new_session()` with models returns a `NewSessionResponse` with 1 config option of category `"model"` and the first model as `current_value`. |
| `test_set_config_option_switches_model` | `set_config_option(config_id="model", value=...)` updates `_session_models[session_id]` and returns updated `config_options`. |
| `test_set_config_option_invalid_model_raises_error` | Switching to an unrecognized model value raises `RequestError` containing `"Invalid model"`. |
| `test_config_options_with_modes_and_models` | When both `modes` and `models` are provided, `new_session()` returns 2 config options: mode first, model second. |
| `test_switching_mode_via_config_option` | Mode can be switched via `set_config_option(config_id="mode", ...)` and is reflected in `_session_modes`. |
| `test_model_passed_to_agent_context` | After switching model and calling `_reset_agent()`, `_session_models[session_id]` holds the new model value. |
| `test_default_model_when_none_configured` | Without a `models` list, `new_session()` returns no config options and `context.model` is `None` in the factory. |

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `AgentServerACP`, `AgentSessionContext` | `deepagents_acp.server` | System under test |
| `create_deep_agent` | `deepagents` | Agent construction |
| `FilesystemBackend` | `deepagents.backends` | Sandbox backend for tests |
| `ChatAnthropic` | `langchain_anthropic` | Present in imports for context |
| `NewSessionResponse`, `SetSessionConfigOptionResponse` | `acp.schema` | Response type assertions |
