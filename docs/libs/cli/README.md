# `deepagents-cli` — CLI Package Documentation

This directory contains comprehensive documentation for the `deepagents-cli` Python package — the interactive terminal interface for the Deep Agents AI coding assistant.

## What is deepagents-cli?

`deepagents-cli` provides a full-featured terminal UI (TUI) built on [Textual](https://textual.textualize.io/) for interacting with AI agents powered by LangGraph. It supports:

- Interactive chat with AI agents (streaming responses with Markdown rendering)
- Multiple AI providers (Anthropic, OpenAI, Google, and 15+ others)
- File operations with Human-in-the-Loop approval
- Shell command execution (with optional auto-approve)
- MCP (Model Context Protocol) tool integration
- Reusable agent skills
- Custom subagents
- Conversation thread persistence
- Remote sandbox execution (Modal, Daytona, Runloop, etc.)
- Non-interactive mode for scripting (`-n`)
- ACP server mode (`--acp`)

## Quick Start

```bash
pip install deepagents-cli
deepagents                          # Launch interactive TUI
deepagents -n "Write a hello world" # Non-interactive single task
deepagents -M anthropic:claude-opus-4-6  # Specify model
deepagents --help                   # Full help
```

## Documentation Index

### Configuration Files

| File | Description |
|---|---|
| [`pyproject.toml.md`](pyproject.toml.md) | Package metadata, all dependencies, optional extras, entry point |
| [`Makefile.md`](Makefile.md) | Development commands (test, lint, format, type check) |

### Core Package (`deepagents_cli/`)

#### Entry Points

| File | Description |
|---|---|
| [`deepagents_cli/__main__.md`](deepagents_cli/__main__.md) | `python -m deepagents_cli` entry point |
| [`deepagents_cli/main.md`](deepagents_cli/main.md) | CLI argument parsing, startup sequence, `cli_main()` |

#### Application

| File | Description |
|---|---|
| [`deepagents_cli/app.md`](deepagents_cli/app.md) | `DeepAgentsApp` — main Textual TUI application |
| [`deepagents_cli/theme.md`](deepagents_cli/theme.md) | Brand colors, `ThemeEntry` registry, user-defined themes |

#### Agent and Model

| File | Description |
|---|---|
| [`deepagents_cli/agent.md`](deepagents_cli/agent.md) | Agent creation with full middleware stack |
| [`deepagents_cli/model_config.md`](deepagents_cli/model_config.md) | `ModelSpec`, provider discovery, profile loading, default model |

#### Session and Configuration

| File | Description |
|---|---|
| [`deepagents_cli/config.md`](deepagents_cli/config.md) | `Settings`, lazy bootstrap, dotenv loading, shared `console` |
| [`deepagents_cli/sessions.md`](deepagents_cli/sessions.md) | Thread persistence via SQLite, `ThreadInfo`, `generate_thread_id` |

#### Server Infrastructure

| File | Description |
|---|---|
| [`deepagents_cli/server.md`](deepagents_cli/server.md) | `langgraph dev` subprocess lifecycle, `generate_langgraph_json` |
| [`deepagents_cli/_server_config.md`](deepagents_cli/_server_config.md) | `ServerConfig` — CLI↔server env var protocol |

#### Tools and Integrations

| File | Description |
|---|---|
| [`deepagents_cli/mcp_tools.md`](deepagents_cli/mcp_tools.md) | MCP server loading, `MCPServerInfo`, config validation |
| [`deepagents_cli/hooks.md`](deepagents_cli/hooks.md) | External hook dispatch for session events |

#### Skills and Subagents

| File | Description |
|---|---|
| [`deepagents_cli/subagents.md`](deepagents_cli/subagents.md) | `SubagentMetadata`, AGENTS.md parsing, multi-directory loading |

#### Input, Output, and Editing

| File | Description |
|---|---|
| [`deepagents_cli/editor.md`](deepagents_cli/editor.md) | External editor integration (`$VISUAL`/`$EDITOR`) |
| [`deepagents_cli/file_ops.md`](deepagents_cli/file_ops.md) | File diff computation for HITL previews |
| [`deepagents_cli/output.md`](deepagents_cli/output.md) | JSON output helpers, `write_json`, `OutputFormat` |

#### Private/Internal Modules

| File | Description |
|---|---|
| [`deepagents_cli/_version.md`](deepagents_cli/_version.md) | Version string and URL constants |
| [`deepagents_cli/_cli_context.md`](deepagents_cli/_cli_context.md) | `CLIContext` TypedDict for runtime model overrides |
| [`deepagents_cli/_session_stats.md`](deepagents_cli/_session_stats.md) | `SessionStats`, `ModelStats` — token tracking |

### Subpackages

| Package | Description |
|---|---|
| [`deepagents_cli/widgets/`](deepagents_cli/widgets/README.md) | All Textual UI widget components |
| [`deepagents_cli/skills/`](deepagents_cli/skills/README.md) | Skill management CLI commands and discovery |
| [`deepagents_cli/integrations/`](deepagents_cli/integrations/README.md) | Sandbox provider abstractions |
| [`deepagents_cli/built_in_skills/`](deepagents_cli/built_in_skills/README.md) | Skills shipped with the package |

### Widget Documentation (`deepagents_cli/widgets/`)

| File | Widget | Description |
|---|---|---|
| [`widgets/__init__.md`](deepagents_cli/widgets/__init__.md) | — | Package overview |
| [`widgets/approval.md`](deepagents_cli/widgets/approval.md) | `ApprovalMenu` | HITL tool approval widget |
| [`widgets/ask_user.md`](deepagents_cli/widgets/ask_user.md) | `AskUserMenu` | Interactive question widget |
| [`widgets/autocomplete.md`](deepagents_cli/widgets/autocomplete.md) | `MultiCompletionManager` | Slash command and file mention autocomplete |
| [`widgets/chat_input.md`](deepagents_cli/widgets/chat_input.md) | `ChatInput` | Primary multi-line input widget |
| [`widgets/diff.md`](deepagents_cli/widgets/diff.md) | `compose_diff_lines` | Theme-aware unified diff renderer |
| [`widgets/history.md`](deepagents_cli/widgets/history.md) | `HistoryManager` | Input history with file persistence |
| [`widgets/loading.md`](deepagents_cli/widgets/loading.md) | `LoadingWidget`, `Spinner` | Animated thinking indicator |
| [`widgets/mcp_viewer.md`](deepagents_cli/widgets/mcp_viewer.md) | `MCPViewerScreen` | MCP server/tool viewer modal |
| [`widgets/message_store.md`](deepagents_cli/widgets/message_store.md) | `MessageStore`, `MessageData` | Virtualized chat history |
| [`widgets/messages.md`](deepagents_cli/widgets/messages.md) | `AssistantMessage`, `ToolCallMessage`, etc. | Message display widgets |
| [`widgets/model_selector.md`](deepagents_cli/widgets/model_selector.md) | `ModelSelectorScreen` | Interactive model picker modal |
| [`widgets/status.md`](deepagents_cli/widgets/status.md) | `StatusBar`, `ModelLabel` | Bottom status bar |
| [`widgets/theme_selector.md`](deepagents_cli/widgets/theme_selector.md) | `ThemeSelectorScreen` | Theme picker with live preview |
| [`widgets/thread_selector.md`](deepagents_cli/widgets/thread_selector.md) | `ThreadSelectorScreen` | Thread browser and selector modal |
| [`widgets/tool_renderers.md`](deepagents_cli/widgets/tool_renderers.md) | `get_renderer()` | Registry pattern for tool approval widgets |
| [`widgets/tool_widgets.md`](deepagents_cli/widgets/tool_widgets.md) | `WriteFileApprovalWidget`, etc. | Tool-specific HITL preview widgets |
| [`widgets/welcome.md`](deepagents_cli/widgets/welcome.md) | `WelcomeBanner` | Startup banner with tips |

## Key CLI Commands

```
deepagents [options]            Interactive TUI session
deepagents -n "task"            Non-interactive single task
deepagents -r                   Resume most recent thread
deepagents -r <id>              Resume specific thread
deepagents -a researcher        Use a named agent
deepagents -M anthropic:claude-opus-4-6  Specify model
deepagents list                 List available agents
deepagents skills list          List available skills
deepagents skills create <name> Create a new skill
deepagents threads list         List conversation threads
deepagents threads delete <id>  Delete a thread
deepagents --sandbox modal      Use Modal sandbox
deepagents --mcp-config path    Load MCP servers from config
deepagents --acp                Run as ACP server
```

## Slash Commands (In-TUI)

| Command | Description |
|---|---|
| `/model` | Switch AI model |
| `/threads` | Browse and resume threads |
| `/new` | Start a new thread |
| `/theme` | Change color theme |
| `/mcp` | View loaded MCP servers and tools |
| `/skills` | List available skills |
| `/skill:<name>` | Invoke a skill directly |
| `/offload` | Compact the conversation when it gets long |
| `/remember` | Save learnings from the conversation |
| `/docs` | Open documentation |
| `/changelog` | Open changelog |
| `/update` | Check for and install updates |
| `/feedback` | Open GitHub issues |

## Configuration

The CLI reads configuration from `~/.deepagents/config.toml`:

```toml
[ui]
theme = "langchain"           # or "langchain-light"

[model]
default = "anthropic:claude-sonnet-4-6"

[warnings]
suppress = ["ripgrep"]

[async_subagents.researcher]
description = "Research agent"
url = "https://my-deployment.langsmith.dev"
graph_id = "agent"

[themes.my-theme]
label = "My Custom Theme"
dark = true
primary = "#7AA2F7"
```
