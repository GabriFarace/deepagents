# deepagents — Documentation Hub

deepagents is an open-source, provider-agnostic AI agent framework built on [LangGraph](https://github.com/langchain-ai/langgraph). It ships a Python SDK (`deepagents`), a batteries-included CLI (`deepagents-cli`) with a Textual TUI, an ACP server adapter, and an evaluation suite.

**New here? Start with [ROADMAP.md](ROADMAP.md)** — a dependency-ordered learning path from orientation to advanced topics.

---

## Directory Tree

```
docs/
├── README.md                    ← this file (navigation hub)
├── ROADMAP.md                   ← ordered learning path
├── libs/
│   ├── README.md                ← package dependency graph & selection guide
│   ├── deepagents/              ← Core SDK
│   │   └── deepagents/
│   │       ├── README.md
│   │       ├── graph.md         ← create_deep_agent() — main entry point
│   │       ├── _models.md       ← model resolution
│   │       ├── backends/
│   │       │   ├── README.md    ← backend selection guide
│   │       │   ├── protocol.md  ← BackendProtocol interface
│   │       │   ├── state.md
│   │       │   ├── filesystem.md
│   │       │   ├── store.md
│   │       │   ├── composite.md
│   │       │   ├── sandbox.md
│   │       │   └── local_shell.md
│   │       └── middleware/
│   │           ├── README.md    ← middleware stack & ordering
│   │           ├── filesystem.md
│   │           ├── skills.md
│   │           ├── memory.md
│   │           ├── subagents.md
│   │           ├── async_subagents.md
│   │           ├── summarization.md
│   │           └── human_in_the_loop.md
│   ├── cli/                     ← deepagents-cli (TUI + LangGraph server)
│   │   └── deepagents_cli/
│   │       ├── README.md        ← CLI architecture overview
│   │       ├── main.md          ← entry point & startup flow
│   │       ├── app.md           ← Textual TUI App
│   │       ├── agent.md         ← agent factory for CLI
│   │       ├── server.md        ← LangGraph server process
│   │       ├── server_manager.md
│   │       ├── server_graph.md  ← graph wiring inside server
│   │       ├── remote_client.md ← HTTP+SSE client
│   │       ├── sessions.md      ← thread persistence
│   │       ├── mcp_tools.md     ← MCP tool loading
│   │       ├── command_registry.md ← slash commands
│   │       ├── hooks.md         ← lifecycle hooks
│   │       ├── config.md        ← settings & config file
│   │       ├── non_interactive.md
│   │       ├── subagents.md
│   │       ├── tools.md
│   │       ├── input.md
│   │       └── widgets/
│   │           ├── README.md
│   │           ├── chat_input.md
│   │           ├── messages.md
│   │           ├── message_store.md
│   │           ├── approval.md
│   │           ├── status.md
│   │           └── welcome.md
│   ├── acp/
│   │   └── README.md            ← ACP bridge
│   └── evals/
│       └── README.md            ← evaluation suite
└── examples/
    └── README.md                ← example agents
```

---

## Quick Navigation

### By Layer (top to bottom)

| Layer | What to read |
|---|---|
| User entry point (CLI) | [main.md](libs/cli/deepagents_cli/main.md) |
| TUI rendering | [app.md](libs/cli/deepagents_cli/app.md), [widgets/README.md](libs/cli/deepagents_cli/widgets/README.md) |
| Server lifecycle | [server_manager.md](libs/cli/deepagents_cli/server_manager.md), [server.md](libs/cli/deepagents_cli/server.md) |
| Agent wiring | [agent.md](libs/cli/deepagents_cli/agent.md), [server_graph.md](libs/cli/deepagents_cli/server_graph.md) |
| LangGraph comms | [remote_client.md](libs/cli/deepagents_cli/remote_client.md) |
| SDK core | [graph.md](libs/deepagents/deepagents/graph.md) |
| Middleware pipeline | [middleware/README.md](libs/deepagents/deepagents/middleware/README.md) |
| File/shell access | [middleware/filesystem.md](libs/deepagents/deepagents/middleware/filesystem.md) |
| Storage backends | [backends/README.md](libs/deepagents/deepagents/backends/README.md) |
| External tooling (MCP) | [mcp_tools.md](libs/cli/deepagents_cli/mcp_tools.md) |
| ACP protocol | [acp/README.md](libs/acp/README.md) |

### By Concept

| Question | Where to look |
|---|---|
| How does the CLI start up? | [main.md](libs/cli/deepagents_cli/main.md) → [server_manager.md](libs/cli/deepagents_cli/server_manager.md) |
| How does a message go from user to agent and back? | [app.md](libs/cli/deepagents_cli/app.md) → [remote_client.md](libs/cli/deepagents_cli/remote_client.md) |
| How do slash commands work? | [command_registry.md](libs/cli/deepagents_cli/command_registry.md) |
| How are MCP tools loaded? | [mcp_tools.md](libs/cli/deepagents_cli/mcp_tools.md) |
| How does session persistence work? | [sessions.md](libs/cli/deepagents_cli/sessions.md) |
| How do HITL approvals work? | [widgets/approval.md](libs/cli/deepagents_cli/widgets/approval.md), [middleware/human_in_the_loop.md](libs/deepagents/deepagents/middleware/human_in_the_loop.md) |
| How do hooks work? | [hooks.md](libs/cli/deepagents_cli/hooks.md) |
| How do custom skills work? | [middleware/skills.md](libs/deepagents/deepagents/middleware/skills.md) |
| How is the agent assembled? | [graph.md](libs/deepagents/deepagents/graph.md) |
| What backend should I use? | [backends/README.md](libs/deepagents/deepagents/backends/README.md) |
| How does context compaction work? | [middleware/summarization.md](libs/deepagents/deepagents/middleware/summarization.md) |
| How do subagents work? | [middleware/subagents.md](libs/deepagents/deepagents/middleware/subagents.md) |
| How does non-interactive mode work? | [non_interactive.md](libs/cli/deepagents_cli/non_interactive.md) |
