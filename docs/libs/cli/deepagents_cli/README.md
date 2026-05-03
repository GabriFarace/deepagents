# deepagents-cli — Architecture Overview

`deepagents-cli` is the interactive user-facing layer of deepagents. It provides a Textual TUI (terminal user interface), manages a LangGraph server subprocess, and communicates with it over HTTP + SSE. The CLI does **not** embed the agent graph directly — it always talks to the agent through the LangGraph server API.

---

## High-Level Architecture

```
User
  │  keyboard / stdin
  ▼
┌─────────────────────────────────────────────────┐
│  Textual TUI  (app.py)                          │
│  ┌─────────┐  ┌──────────────┐  ┌────────────┐ │
│  │StatusBar│  │VerticalScroll│  │ ChatInput  │ │
│  └─────────┘  │  (messages)  │  └────────────┘ │
│               └──────────────┘                  │
│         ┌──────────────────────────┐            │
│         │ Modal overlays           │            │
│         │ ApprovalMenu, ModelSel…  │            │
│         └──────────────────────────┘            │
└──────────────────┬──────────────────────────────┘
                   │ HTTP + SSE  (RemoteAgent)
                   ▼
┌─────────────────────────────────────────────────┐
│  LangGraph Server Subprocess                    │
│  (langgraph dev — spawned by ServerProcess)     │
│                                                 │
│  server_graph.py::make_graph()                  │
│   └─ create_cli_agent()  (agent.py)             │
│       └─ create_deep_agent()  (SDK)             │
│           └─ middleware stack + tools           │
└─────────────────────────────────────────────────┘
         │ SQLite checkpoint
         ▼
~/.deepagents/.state/threads.db
```

This split is intentional: it gives the CLI persistence (via LangGraph checkpointing), hot-reload capability, and the ability to run remote or async subagents without coupling to any specific process.

---

## File Map

| File | Role |
|---|---|
| [`main.py`](main.md) | CLI entry point — argument parsing, mode dispatch |
| [`app.py`](app.md) | Textual `CLIApp` — widget hierarchy, message queue, input routing |
| [`agent.py`](agent.md) | `create_cli_agent()` — builds the agent graph for the CLI's server |
| [`server.py`](server.md) | `ServerProcess` — manages the `langgraph dev` subprocess |
| [`server_manager.py`](server_manager.md) | `start_server_and_get_agent()` — orchestrates startup sequence |
| [`server_graph.py`](server_graph.md) | `make_graph()` — wires up the agent inside the server process |
| [`remote_client.py`](remote_client.md) | `RemoteAgent` — HTTP+SSE client for the LangGraph server |
| [`sessions.py`](sessions.md) | Thread metadata — listing, resuming, creating |
| [`mcp_tools.py`](mcp_tools.md) | MCP server config loading, tool discovery |
| [`command_registry.py`](command_registry.md) | Slash command definitions and bypass tiers |
| [`hooks.py`](hooks.md) | Lifecycle hook dispatch (session.start, message.agent, …) |
| [`config.py`](config.md) | Settings singleton, dotenv loading, bootstrap |
| [`non_interactive.py`](non_interactive.md) | Headless `-n` mode |
| [`subagents.py`](subagents.md) | Loads subagent specs from `~/.deepagents/{agent}/agents/` |
| [`tools.py`](tools.md) | `web_search` and `fetch_url` tool definitions |
| [`input.py`](input.md) | `@file` mention parsing, media tracking |
| [`widgets/`](widgets/README.md) | All Textual widget classes |

---

## Key Design Decisions

**Why a subprocess?** Running `langgraph dev` as a subprocess isolates the agent graph from the TUI process. This enables: LangGraph's built-in SQLite checkpointing, hot-reload of graph code without restarting the TUI, and future remote-server support with no TUI changes.

**Why SSE streaming?** Server-Sent Events give real-time token-by-token streaming without WebSockets. Each state update from LangGraph is forwarded to the TUI as it arrives.

**Why a message queue?** The TUI processes user input asynchronously. A `deque[QueuedMessage]` ensures messages and commands execute in order and don't race with each other or with ongoing agent execution.

---

## Startup Sequence (summary)

1. `main.py::cli_main()` parses args and calls `run_textual_cli_async()`
2. `run_textual_cli_async()` creates a `CLIApp` and calls `app.run_async()`
3. `CLIApp.on_mount()` calls `start_server_and_get_agent()` (server_manager.py)
4. `start_server_and_get_agent()` scaffolds a temp workspace and starts `langgraph dev`
5. When the server is ready, a `RemoteAgent` client is created and returned to the TUI
6. The TUI enters its idle loop; user input is routed through `ChatInput` → `_handle_submit()`

Full detail: [main.md](main.md) → [server_manager.md](server_manager.md) → [app.md](app.md)

---

## Message Flow (summary)

1. User types in `ChatInput` and presses Enter
2. `app.py` enqueues the message; pops it when agent is idle
3. `RemoteAgent.astream()` sends the message to the LangGraph server via HTTP POST + SSE
4. Server streams state updates (AI tokens, tool calls, interrupts) back as SSE chunks
5. `StreamHandler` in `remote_client.py` translates chunks into widget mutations
6. `CLIApp` applies widget updates: appends text, creates `ToolCallMessage`, shows `ApprovalMenu`

Full detail: [app.md](app.md) → [remote_client.md](remote_client.md)

---

## See Also

- [CLI entry point: main.md](main.md)
- [TUI App: app.md](app.md)
- [Widgets: widgets/README.md](widgets/README.md)
- [SDK entry point: ../../deepagents/deepagents/graph.md](../../deepagents/deepagents/graph.md)
