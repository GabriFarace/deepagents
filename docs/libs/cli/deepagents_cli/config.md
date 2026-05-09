# `libs/cli/deepagents_cli/config.py`

> Broad CLI configuration module: settings, env bootstrap, glyphs, shell
> safety, LangSmith links, and model construction.

## Position in the system

Used by both parent process and server subprocess. It centralizes user config
and model creation.

## Functions and classes

### Bootstrap and display helpers

`_find_dotenv_from_start_path()`, `_load_dotenv()`, `_ensure_bootstrap()`,
`CharsetMode`, `Glyphs`, `_detect_charset_mode()`, `get_glyphs()`,
`newline_shortcut()`, and `get_banner()` prepare environment and terminal UI
presentation.

### `Settings` and `SessionState`

Store global config paths, provider settings, feature flags, session metadata,
and runtime options.

### Shell and LangSmith helpers

`contains_dangerous_patterns()`, `is_shell_command_allowed()`,
`get_langsmith_project_name()`, `fetch_langsmith_project_url()`, and
`build_langsmith_thread_url()` support safety and trace links.

### Model construction

`detect_provider()`, `_get_default_model_spec()`, provider kwarg helpers,
`ModelResult`, `_apply_profile_overrides()`, `create_model()`, and
`validate_model_capabilities()` turn config into a usable chat model.

### Lazy globals

`_get_console()`, `_get_settings()`, and `__getattr__()` defer expensive global
construction until needed.

## Gotchas

Before changing this file, check whether the caller is parent CLI process,
server subprocess, or both.
