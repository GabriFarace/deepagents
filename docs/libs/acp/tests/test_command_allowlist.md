# `tests/test_command_allowlist.py`

## High-Level Purpose

Unit and integration tests for the command-type allowlist system. Tests two areas: (1) correctness of `extract_command_types()` for parsing shell commands into security-relevant signatures, and (2) the per-session command allowlist data structure on `AgentServerACP`.

## Classes

### `TestExtractCommandTypes`

Tests the `extract_command_types` utility function from `deepagents_acp.utils`.

| Test Method | Behavior Verified |
|-------------|-------------------|
| `test_simple_non_sensitive_command` | `ls`, `pwd`, `cat` return just the base command name |
| `test_npm_commands_with_subcommands` | `npm install`, `npm test`, `npm run build`, `npm start` include subcommand |
| `test_python_with_module_flag` | `python -m pytest tests/` → `["python -m pytest"]` |
| `test_python_with_code_flag` | `python -c '...'` → `["python -c"]` regardless of code content |
| `test_python_script_execution` | `python script.py` → `["python"]` |
| `test_node_commands` | `-e`/`-p` flags produce `"node -e"`/`"node -p"`, scripts produce `"node"` |
| `test_npx_with_package` | `npx jest` → `["npx jest"]` |
| `test_yarn_commands` | yarn mirrors npm subcommand behavior |
| `test_uv_commands` | `uv run pytest` → `["uv run pytest"]`, `uv pip install` → `["uv pip"]` |
| `test_command_with_and_operator` | `&&`-chained commands produce multiple entries |
| `test_command_with_pipes_and_and_operator` | Both pipes and `&&` are handled |
| `test_empty_command` | Empty/whitespace input returns `[]` |
| `test_command_with_trailing_and_operator` | Handles varying whitespace around `&&` |
| `test_duplicate_commands_preserved` | Repeated commands appear multiple times |
| `test_complex_real_world_command` | `cd /path && python -m pytest tests/ -v` → `["cd", "python -m pytest"]` |
| `test_security_python_different_modules` | `python -m pytest` and `python -m pip` are different signatures |
| `test_security_npm_different_subcommands` | `npm install` and `npm test` are different signatures |

### `TestCommandTypeAllowlist`

Tests the `_allowed_command_types` dict on `AgentServerACP`.

| Test Method | Behavior Verified |
|-------------|-------------------|
| `test_allowed_command_types_initialized` | Dict exists and is empty on construction |
| `test_can_add_allowed_command_type` | Tuples can be added; lookup works; unrelated commands are not auto-allowed |
| `test_command_types_are_session_specific` | Separate sessions have independent allowlists |
| `test_multiple_command_types_in_single_command` | All command types in a `&&`-command must be individually allowed |
| `test_security_python_pytest_vs_pip` | Allowing `python -m pytest` does not allow `python -m pip` or `python -c` |
| `test_security_npm_install_vs_run` | Allowing `npm install` does not allow `npm run arbitrary-script` |
| `test_security_uv_run_pytest_vs_python` | Allowing `uv run pytest` does not allow `uv run python` or `uv pip` |

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `extract_command_types` | `deepagents_acp.utils` | Function under test |
| `AgentServerACP` | `deepagents_acp.server` | Server class with allowlist state |
| `GenericFakeChatModel` | `tests.chat_model` | Fake LLM for agent construction |
