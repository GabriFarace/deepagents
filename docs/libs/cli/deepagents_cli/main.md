# `libs/cli/deepagents_cli/main.py`

## High-Level Purpose

`main.py` is the CLI entry point. It is the first Python code that runs when the user types `deepagents`. Its job is narrow: parse arguments, bootstrap configuration, and dispatch to one of three execution modes (interactive TUI, non-interactive headless, or ACP server). It intentionally defers heavy imports until after the fast-path checks (e.g., `--version`) complete, keeping cold-start time low.

---

## Key Functions

### `cli_main() → None`

The top-level entry point registered as the `deepagents` console script in `pyproject.toml`.

Execution steps:

1. **Fast version check** — if `--version` is the only argument, prints the version and exits without importing any heavy dependencies.
2. **Dependency validation** — calls `check_cli_dependencies()` which ensures optional extras (`requests`, `python-dotenv`, `tavily-python`, `textual`) are installed. Prints a helpful error and exits if any are missing.
3. **Argument parsing** — `parse_args()` returns a `Namespace` with all CLI flags.
4. **Stdin piping** — `apply_stdin_pipe()` converts piped stdin into a `-n` prompt so `echo "explain this" | deepagents` works without special flags.
5. **Mode dispatch**:
   - `-n` / `--non-interactive` → `run_non_interactive()`
   - `--acp` → `_run_acp_cli_async()` (starts an ACP server)
   - default → `run_textual_cli_async()`

### `run_textual_cli_async(args) → None`

Sets up configuration and runs the interactive TUI.

1. Resolves the model spec cheaply (no full config load) for the status bar display.
2. Calls `apply_model_config()` to propagate model and provider settings into environment variables that the LangGraph server subprocess will inherit.
3. Calls `run_textual_app()` from `app.py` with server kwargs (MCP preload settings, model config, thread ID for resume, etc.).

### `parse_args() → argparse.Namespace`

Defines all CLI flags. Key flags:

| Flag | Effect |
|---|---|
| `--version` | Print version and exit |
| `-n "prompt"` | Non-interactive mode |
| `--acp` | Start ACP server |
| `-r [thread_id]` | Resume a previous thread (omit ID for most recent) |
| `--model MODEL` | Override model for this session |
| `--mcp-config PATH` | Path to MCP config file |
| `--shell-allow-list CMDS` | Comma-separated allowed shell commands (non-interactive) |
| `--max-turns N` | Limit agentic loop iterations (non-interactive) |
| `--no-stream` | Buffer full response before printing (non-interactive) |
| `--quiet` | Print only final response to stdout; progress to stderr |
| `--agent NAME` | Select a named agent from `~/.deepagents/agents/` |

### `apply_stdin_pipe(args) → None`

If stdin is not a TTY (e.g., piped input), reads stdin and sets `args.non_interactive = True` and `args.prompt = <stdin content>`. This normalizes the two ways of providing a non-interactive prompt.

### `_run_acp_cli_async(args) → None`

Starts the ACP server mode. Creates a `deepagents_acp.AgentServerACP` instance and runs its async server loop. Useful for integrating the agent into external ACP-compatible clients.

---

## Architecture Notes

The startup code is split across `main.py` and `config.py` deliberately. `main.py` owns argument parsing and mode dispatch. `config.py` owns the settings singleton, dotenv loading, and lazy bootstrap. The split means tests can import `config.py` without triggering argument parsing side effects.

The heavy Textual import (`from deepagents_cli.app import run_textual_app`) is deferred until after argument parsing, so `--version` and `--help` remain near-instant.

---

## See Also

- [config.md](config.md) — settings bootstrap details
- [app.md](app.md) — what `run_textual_app()` does
- [non_interactive.md](non_interactive.md) — headless mode
- [server_manager.md](server_manager.md) — server startup called from app
