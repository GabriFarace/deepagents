# `main.py`

## High-Level Purpose

This is the primary entry point and CLI argument parser for `deepagents-cli`. It handles:

- Parsing all command-line arguments via `argparse`
- Checking for required dependencies and optional external tools (e.g., `ripgrep`)
- Routing to subcommands (`help`, `list`, `reset`, `skills`, `threads`)
- Starting the Textual TUI or non-interactive mode
- Preloading MCP server metadata for server mode
- Performing the startup sequence (dependency checks, model resolution, server startup)

## Functions

### `check_cli_dependencies() -> None`

Checks that all required Python packages for the CLI are installed (`requests`, `python-dotenv`, `tavily-python`, `textual`). If any are missing, prints an install hint and calls `sys.exit(1)`.

### `_ripgrep_install_hint() -> str`

Returns a platform-appropriate install command for `ripgrep`. Detects the OS and available package managers (brew, apt-get, dnf, pacman, zypper, apk, nix-env, choco, scoop, winget, cargo, conda) and returns the most suitable install command. Falls back to the GitHub URL if no package manager is found.

**Returns:** A platform-specific install command string or GitHub URL.

### `check_optional_tools(*, config_path: Path | None = None) -> list[str]`

Checks for recommended but optional external tools. Currently checks for `ripgrep` (`rg`). Respects the `[warnings].suppress` list in `config.toml` to silence specific tool warnings.

**Parameters:**
- `config_path`: Path to the config file. Defaults to `~/.deepagents/config.toml`.

**Returns:** A list of missing tool names (e.g., `["ripgrep"]`).

### `format_tool_warning_tui(tool: str) -> str`

Formats a missing-tool warning for display in the TUI as a toast notification.

**Parameters:**
- `tool`: Name of the missing tool.

**Returns:** Plain-text warning suitable for `App.notify`.

### `format_tool_warning_cli(tool: str) -> str`

Formats a missing-tool warning for non-interactive console output using Rich markup.

**Parameters:**
- `tool`: Name of the missing tool.

**Returns:** Warning string with optional Rich link markup for URLs.

### `_preload_session_mcp_server_info(*, mcp_config_path, no_mcp, trust_project_mcp) -> list[MCPServerInfo] | None`

Async function that preloads MCP server metadata for the interactive TUI when running in server mode. In server mode, MCP tools are created inside the LangGraph server process, but the local Textual app needs metadata for the welcome banner and `/mcp` viewer.

**Parameters:**
- `mcp_config_path`: Optional explicit MCP config path.
- `no_mcp`: Whether MCP loading is disabled.
- `trust_project_mcp`: Project-level MCP trust decision.

**Returns:** List of `MCPServerInfo` objects, or `None` when MCP is disabled.

**Key Logic:** Opens a temporary MCP session, collects metadata, then immediately cleans up the session.

### `parse_args() -> argparse.Namespace`

Builds the full argument parser and parses `sys.argv`. Uses a custom `_make_help_action` factory to wire `-h` flags to Rich-formatted help screens instead of argparse's default output.

**Subcommands registered:**
- `help` — show help information
- `list` — list available agents
- `reset` — reset an agent (with `--agent` and optional `--target`)
- `skills` — skill management subcommands (see `skills/commands.py`)
- `threads` — manage conversation threads with `list`/`delete` sub-subcommands

**Key flags (root parser):**
| Flag | Description |
|---|---|
| `-r / --resume` | Resume a thread (most recent or by ID) |
| `-a / --agent` | Agent name to use |
| `-M / --model` | Model spec (`provider:model` or bare name) |
| `--model-params` | JSON object of extra model kwargs |
| `--profile-override` | JSON object to override model profile fields |
| `--default-model` | Set or show the default model |
| `--clear-default-model` | Clear the persisted default model |
| `-m / --message` | Initial prompt to auto-submit |
| `-n / --non-interactive` | Run a single task non-interactively and exit |
| `--max-turns N` | Maximum number of agentic turns before stopping (requires `-n` or piped stdin). Clamped to `_MAX_HITL_ITERATIONS = 50`. Exits with code 2 if used without non-interactive mode. Useful for CI/CD pipelines to prevent runaway agents. |
| `-q / --quiet` | Clean output for piping |
| `--no-stream` | Buffer full response before writing to stdout |
| `-y / --auto-approve` | Auto-approve all tool calls |
| `--sandbox` | Remote sandbox type (none/agentcore/modal/daytona/runloop/langsmith) |
| `--sandbox-id` | Existing sandbox ID to reuse |
| `--sandbox-setup` | Path to setup script for sandbox |
| `-S / --shell-allow-list` | Shell commands to auto-approve |
| `--mcp-config` | Path to MCP servers JSON config |
| `--no-mcp` | Disable all MCP tool loading |
| `--trust-project-mcp` | Trust project-level MCP configs |
| `--update` | Check for and install updates |
| `--acp` | Run as an ACP server over stdio |
| `-v / --version` | Show version |

### `cli_main() -> None`

The top-level entry point called by `__main__.py` and by the `deepagents` console script. Orchestrates the entire startup sequence:

1. Calls `check_cli_dependencies()`.
2. Calls `parse_args()`.
3. Routes to subcommand handlers or starts the TUI/non-interactive session.
4. Performs model resolution, agent creation, server startup, and MCP loading.
5. Launches `DeepAgentsApp` (Textual TUI) or the non-interactive runner.

## Internal Helpers

### `_extract_model_params_flag(raw_arg: str) -> tuple[str, dict[str, Any] | None]`

Parses the `--model-params` flag value from a `/model` command's raw argument string. Handles quoted (`'...'` / `"..."`), bare `{...}` JSON values with balanced braces, and simple whitespace-delimited tokens.

**Parameters:**
- `raw_arg`: The argument string after `/model `.

**Returns:** `(remaining_args, parsed_dict | None)`.

**Raises:** `ValueError` if JSON is missing, has unclosed quotes, unbalanced braces, or is invalid JSON. `TypeError` if the JSON is not a dict.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `argparse`, `asyncio`, `json`, `os`, `sys` | stdlib | CLI parsing, async, config |
| `shutil` | stdlib | Which-checking for external tools |
| `DeepAgentsApp` | `deepagents_cli.app` | Textual TUI application |
| `MCPServerInfo` | `deepagents_cli.mcp_tools` | MCP server metadata type |
| `__version__` | `deepagents_cli._version` | Version string |
| `add_json_output_arg` | `deepagents_cli.output` | JSON output flag helper |
| `setup_skills_parser` | `deepagents_cli.skills` | Skills subcommand parser setup |
