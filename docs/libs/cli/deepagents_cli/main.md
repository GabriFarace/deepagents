# `libs/cli/deepagents_cli/main.py`

> Main CLI entry point. It parses commands, checks dependencies, resolves mode
> and agent/session choices, starts the server when needed, and hands control to
> Textual, non-interactive, ACP, or utility flows.

## Position in the system

`cli_main()` is the console-script target. It is the parent-process control
plane: it does not run the agent graph itself, but decides which mode should
run and supplies the arguments needed by `server_manager.py`.

## Functions and classes

### `_resolve_agent_arg(args)` and `_recent_agent_is_valid(name)`

Resolve the configured agent name. Explicit `-a` wins; resume mode defers
agent inference to saved thread metadata; config default and recent agent come
next; the hard-coded default is last. `_recent_agent_is_valid()` guards stale
config entries whose agent directory no longer exists.

### Dependency and warning helpers

`check_cli_dependencies()` exits on missing required CLI extras.
`_ripgrep_install_hint()`, `check_optional_tools()`,
`build_missing_tool_notification()`, and `format_tool_warning_cli()` turn
missing recommended tools into either TUI notifications or non-interactive
warnings.

### Argument parsing and early command handling

`_show_bare_command_group_help()` and `parse_args()` build the CLI command
surface. The parser covers utility commands, interactive mode, `-p` print mode,
ACP mode, model/agent/session flags, sandbox flags, MCP config/trust flags, and
debug options.

### Runtime mode functions

`run_textual_cli_async()` runs the interactive Textual app around a server
session. `_run_acp_cli_async()` exposes the graph through ACP. `apply_stdin_pipe()`
folds piped stdin into the prompt. `_print_session_stats()` formats stats
output. `_check_mcp_project_trust()` centralizes project MCP trust decisions.

### `cli_main()`

Top-level synchronous entry. It checks dependencies, parses args, dispatches
utility commands, resolves agent/model/session context, handles stdin and MCP
trust, then runs the chosen async mode.

## Flow walk-through

1. User runs `deepagents`.
2. `cli_main()` parses args and handles immediate utility commands.
3. Interactive or print mode resolves agent, model, session, and MCP trust.
4. The selected async runner opens `server_session()`.
5. Stream output is rendered by Textual or the non-interactive console loop.

## Gotchas

Imports are intentionally delayed in hot paths. Avoid moving heavy Textual,
LangGraph, or optional-provider imports to module import time unless startup
cost has been considered.
