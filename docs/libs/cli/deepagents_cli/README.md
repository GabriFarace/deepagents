# `deepagents_cli/` — CLI Package

This is the main Python package for `deepagents-cli`. It provides a full-featured interactive terminal interface (TUI) for the Deep Agents AI coding assistant, built on LangGraph and Textual.

## Architecture Overview

```
deepagents_cli/
├── Entry Points
│   ├── __main__.py          # python -m deepagents_cli
│   └── main.py              # cli_main() — parse args, route, launch TUI
│
├── TUI Application
│   ├── app.py               # DeepAgentsApp (Textual App subclass)
│   ├── theme.py             # Brand colors, ThemeEntry registry
│   └── widgets/             # All Textual UI widget components
│
├── Agent & Model
│   ├── agent.py             # Agent creation with middleware stack
│   ├── model_config.py      # ModelSpec, provider discovery, profile loading
│   └── configurable_model.py # ConfigurableModelMiddleware for runtime switching
│
├── Session & State
│   ├── sessions.py          # Thread persistence via SQLite checkpoints
│   ├── config.py            # Settings, lazy bootstrap, dotenv loading
│   └── _session_stats.py    # Token tracking dataclasses
│
├── Server Infrastructure
│   ├── server.py            # langgraph dev subprocess lifecycle
│   ├── server_graph.py      # Server-side graph definition
│   ├── server_manager.py    # Server start/restart management
│   ├── _server_config.py    # DA_SERVER_* env var protocol
│   └── _server_constants.py # Shared constants
│
├── Tools & Integrations
│   ├── mcp_tools.py         # MCP server loading and tool discovery
│   ├── mcp_trust.py         # Project MCP trust prompting
│   ├── hooks.py             # External hook dispatch
│   └── integrations/        # Sandbox provider abstractions
│
├── Skills & Subagents
│   ├── skills/              # Skill management commands and loading
│   ├── subagents.py         # Subagent loader from AGENTS.md files
│   └── built_in_skills/     # Skills shipped with the package
│
├── Input & Output
│   ├── editor.py            # External editor integration
│   ├── file_ops.py          # File diff computation for HITL previews
│   ├── output.py            # JSON output helpers
│   ├── input.py             # Media input parsing and placeholders
│   └── clipboard.py         # Clipboard integration
│
├── Security & Utilities
│   ├── unicode_security.py  # Dangerous Unicode detection
│   ├── update_check.py      # PyPI update checking
│   ├── project_utils.py     # Git root and project context detection
│   ├── local_context.py     # LocalContextMiddleware
│   └── media_utils.py       # Image/video processing utilities
│
└── Private Modules (prefixed with _)
    ├── _version.py          # Version constants
    ├── _cli_context.py      # CLIContext TypedDict
    ├── _session_stats.py    # SessionStats, ModelStats
    ├── _server_config.py    # ServerConfig dataclass
    ├── _server_constants.py # DA_SERVER_ prefix constant
    ├── _ask_user_types.py   # AskUser type definitions
    ├── _debug.py            # Debug utilities
    └── _testing_models.py   # Test-only model helpers
```

## Key Concepts

### Startup Sequence

1. `cli_main()` in `main.py` checks dependencies and parses args.
2. Model is resolved (from `--model`, config default, or environment detection).
3. In **server mode** (default): a `langgraph dev` subprocess is started via `server.py`. The Textual app connects to it via `remote_client.py`.
4. In **direct mode**: The agent is built inline via `agent.py` and run in-process.
5. `DeepAgentsApp` (from `app.py`) launches the Textual TUI.

### Input Modes

The chat input supports three modes, selected by the first character:
- **Normal** (`no prefix`) — message sent to the AI agent
- **Shell** (`!`) — command run in the local shell
- **Command** (`/`) — slash command (e.g., `/model`, `/threads`, `/new`)

### Human-in-the-Loop (HITL)

Tool calls that modify files, run shell commands, or access the web require approval. The `ApprovalMenu` widget (in `widgets/approval.py`) presents the tool details and captures a `yes/auto/no` decision.

### Thread Persistence

Conversations are persisted as LangGraph checkpoints in `~/.deepagents/checkpoints.db` (SQLite). Each conversation has a UUID7 thread ID. The `/threads` command and `-r` flag manage thread resumption.

### MCP Integration

MCP (Model Context Protocol) servers extend the agent with external tools. Configs are discovered from `~/.deepagents/.mcp.json` and `.deepagents/.mcp.json`. The `/mcp` command shows loaded servers and tools.

## Subpackages

| Package | Description |
|---|---|
| [`widgets/`](widgets/README.md) | All Textual UI widget components |
| [`skills/`](skills/README.md) | Skill management CLI commands and discovery |
| [`integrations/`](integrations/README.md) | Sandbox provider abstractions |
| [`built_in_skills/`](built_in_skills/README.md) | Skills shipped with the package |
