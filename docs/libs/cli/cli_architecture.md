# Agent CLI Architecture

> A conceptual map of how coding-agent CLIs are built, and where
> deepagents-cli chooses a different shape from tools like Claude Code and
> Codex CLI.

## Public Reference Points

Claude Code's public docs describe an agentic coding tool available in the
terminal that reads a codebase, edits files, runs commands, integrates with
MCP, uses project instructions, supports hooks, and can spawn multiple agents
for parallel work. OpenAI's public Codex CLI docs describe a local coding agent
that can read, modify, and run code on the user's machine with approval modes.
Textual is the Python TUI framework deepagents-cli uses, and MCP is the open
protocol deepagents-cli uses for external tools and data sources.

Sources:

- [Claude Code overview](https://code.claude.com/docs/en/overview)
- [Codex CLI](https://developers.openai.com/codex/cli)
- [Textual documentation](https://textual.textualize.io/)
- [Model Context Protocol introduction](https://modelcontextprotocol.io/docs/getting-started/intro)

This chapter stays conceptual. It does not infer private implementation
details of Claude Code or Codex.

## The Shell of an Agent CLI

Most agent CLIs have the same outer shape:

```
entry command
  ├─ parse flags and subcommands
  ├─ load config, env, credentials, project context
  ├─ choose mode: interactive / print / server / utility command
  ├─ build or connect to an agent runtime
  ├─ stream events back to a renderer
  └─ persist session state and cleanup resources
```

In deepagents-cli, `main.py` owns the entry command. It handles utility
subcommands, `--version`, `-p` print mode, ACP mode, session resume, agent
selection, MCP trust checks, optional dependency warnings, and then either runs
the Textual app or the non-interactive loop.

## Single Process vs. Server Backed

A single-process CLI keeps the UI, agent loop, tools, and model calls in one
runtime. That is simple to launch and easy to debug. It is also the natural
shape for a local tool that owns its own event loop and directly calls tools.

deepagents-cli is server-backed. The parent process writes a temporary
LangGraph workspace, starts `langgraph dev`, and talks to that server through a
remote graph client. This buys a few things:

- The graph runs in the same shape locally and remotely.
- Checkpointing and thread state live behind LangGraph's server API.
- SSE streaming, interrupts, and subgraph namespaces use the LangGraph wire
  protocol instead of a CLI-private stream format.
- The TUI can restart or fail gracefully around a server subprocess.

The cost is orchestration complexity: env var handoff, generated
`langgraph.json`, health checks, subprocess logs, temp workspaces, and cleanup.

## TUI Runtime

Terminal UIs need to solve input, layout, redraws, focus, mouse/keyboard
events, and accessibility-ish terminal constraints. deepagents-cli uses Textual
instead of raw ANSI rendering, so the app is organized around widgets:
message transcript, input box, approval menu, model selector, thread selector,
notification center, MCP viewer, and tool renderers.

That means UI state is distributed across widgets. The app coordinates
high-level lifecycle and stream consumption; widgets own local rendering and
interaction behavior.

## Streaming

Agent CLIs feel alive because they stream. There are usually several event
classes mixed together:

- model token chunks,
- tool call starts and results,
- interrupt/approval payloads,
- custom status events,
- errors,
- checkpoint/thread updates.

deepagents-cli receives LangGraph stream tuples through `RemoteAgent.astream`.
`remote_client.py` delegates SSE parsing to `RemoteGraph`, then converts
serialized message dicts into LangChain message objects so the Textual adapter
can render them.

## Approvals and HITL

Approval systems sit between tool intent and tool execution. In LangGraph, this
is naturally represented as an interrupt: the graph pauses, the UI renders an
approval request, and the user's decision resumes the graph.

deepagents-cli has two approval modes:

- interactive UI approvals through widgets, backed by LangGraph interrupts;
- non-interactive allow-list logic that rejects unsafe shell commands without
  pausing the graph, useful for `-p` automation.

## Slash Commands

Slash commands keep the agent prompt channel from becoming the only control
surface. A registry maps command names to handlers, descriptions, argument
metadata, and bypass rules. Autocomplete and help views can then read the same
registry the dispatcher uses.

deepagents-cli also expands skills into command entries, so installed skills
can behave like first-class slash commands without being hard-coded in the TUI.

## MCP Tools

MCP adds external tools, prompts, and resources through a standard protocol.
For a CLI, the operational problems are config discovery, trust, auth,
transport lifecycle, tool-name namespacing, and error display.

deepagents-cli validates explicit MCP configs in the parent process before
server startup, but real MCP tool discovery happens in `server_graph.py` so the
tools exist in the graph process. Runtime sessions are managed by a server-side
`MCPSessionManager`.

## Sessions

A coding CLI needs resumability. deepagents-cli maps sessions to LangGraph
threads backed by the SQLite checkpointer used by the local `langgraph dev`
server. The CLI adds listing, sorting, display metadata, initial prompt
extraction, and message-count caches on top.

## Hooks

Hooks are the escape hatch for local automation. They let a user run commands
around lifecycle events without changing the CLI source. The core design issue
is isolation: hook commands should receive structured payloads, run with clear
timeouts, and fail without taking down the agent unless the event contract says
otherwise.

## How deepagents-cli Fits Together

```
main.py
  ├─ parse args / choose mode
  ├─ server_session(...)
  │    ├─ write env via ServerConfig
  │    ├─ scaffold temp LangGraph workspace
  │    ├─ start ServerProcess
  │    └─ return RemoteAgent
  ├─ Textual app or non-interactive loop
  └─ cleanup

server_graph.py (subprocess)
  ├─ read ServerConfig.from_env()
  ├─ create model
  ├─ load built-in + MCP tools
  ├─ create sandbox backend if configured
  └─ create_cli_agent(...)
```

The important mental model: the CLI process is the interface; the server
process is the agent runtime.
