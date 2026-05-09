# `deepagents` Documentation

> **New here?** Open [`ROADMAP.md`](./ROADMAP.md) — it's a 14-stage learning
> path that orders everything in this tree by dependency. Don't read this
> file linearly.

---

## What this tree contains

This directory documents the `deepagents` codebase **block by block**: every
public function and class in `libs/` gets a 1–3 paragraph explanation that
covers what it computes, what it mutates, who calls it, and any non-obvious
behaviour. System prompts and tool descriptions are quoted in full.

The tree is organised in three layers:

```
docs/
├── ROADMAP.md            ← start here
│
├── prerequisites/        ← LangChain + LangGraph primers (self-contained)
│   ├── langchain.md      ← ChatModel, messages, @tool, AgentMiddleware
│   └── langgraph.md      ← StateGraph, MessagesState, checkpointers,
│                            streaming modes, interrupts, subgraphs
│
└── libs/                 ← mirror of the source tree, file-for-file
    ├── deepagents/       ← core SDK (create_deep_agent, backends, middleware)
    ├── cli/              ← deepagents-cli (heaviest focus)
    │   ├── cli_architecture.md   ← how agent CLIs are built (Claude Code,
    │   │                            Codex, deepagents-cli — comparison)
    │   └── deepagents_cli/       ← per-file docs
    ├── acp/              ← ACP server adapter
    ├── evals/            ← evaluation harness + Harbor benchmarks
    ├── partners/         ← sandbox adapters: daytona, modal, quickjs, runloop
    └── repl/             ← langchain-repl interpreter
```

`examples/` (the 14 example agents) is documented under `docs/examples/`
with one short doc per agent.

---

## Quick navigation

### By layer (top to bottom)

| Layer | Doc |
|---|---|
| External: `LangChain` | [`prerequisites/langchain.md`](./prerequisites/langchain.md) |
| External: `LangGraph` | [`prerequisites/langgraph.md`](./prerequisites/langgraph.md) |
| Core SDK assembly | [`libs/deepagents/deepagents/graph.md`](./libs/deepagents/deepagents/graph.md) |
| Backend protocol | [`libs/deepagents/deepagents/backends/`](./libs/deepagents/deepagents/backends/README.md) |
| Middleware stack | [`libs/deepagents/deepagents/middleware/`](./libs/deepagents/deepagents/middleware/README.md) |
| Generic CLI architecture | [`libs/cli/cli_architecture.md`](./libs/cli/cli_architecture.md) |
| CLI startup | [`libs/cli/deepagents_cli/main.md`](./libs/cli/deepagents_cli/main.md) |
| CLI server lifecycle | [`libs/cli/deepagents_cli/server.md`](./libs/cli/deepagents_cli/server.md) |
| CLI TUI | [`libs/cli/deepagents_cli/app.md`](./libs/cli/deepagents_cli/app.md) |
| ACP adapter | [`libs/acp/`](./libs/acp/README.md) |
| Sandboxes | [`libs/partners/`](./libs/partners/README.md) |
| Evals | [`libs/evals/`](./libs/evals/README.md) |

### By concept ("how do I…?")

| Question | Doc |
|---|---|
| Build my first deep agent | [`libs/deepagents/deepagents/graph.md`](./libs/deepagents/deepagents/graph.md) |
| Add a custom tool | [`libs/deepagents/deepagents/_tools.md`](./libs/deepagents/deepagents/_tools.md) |
| Add a custom middleware | [`libs/deepagents/deepagents/middleware/README.md`](./libs/deepagents/deepagents/middleware/README.md) |
| Pick or write a backend | [`libs/deepagents/deepagents/backends/README.md`](./libs/deepagents/deepagents/backends/README.md) |
| Understand the system prompts | [`libs/deepagents/deepagents/middleware/`](./libs/deepagents/deepagents/middleware/README.md) (one file per middleware) |
| Understand the agent loop | [`prerequisites/langgraph.md`](./prerequisites/langgraph.md) §10 |
| Understand sub-agents (`task` tool) | [`libs/deepagents/deepagents/middleware/subagents.md`](./libs/deepagents/deepagents/middleware/subagents.md) |
| Approve / reject tool calls (HITL) | [`libs/deepagents/deepagents/middleware/permissions.md`](./libs/deepagents/deepagents/middleware/permissions.md) |
| Run an agent in a sandbox | [`libs/partners/`](./libs/partners/README.md) |
| Boot the CLI | [`libs/cli/deepagents_cli/main.md`](./libs/cli/deepagents_cli/main.md) + [`server_manager.md`](./libs/cli/deepagents_cli/server_manager.md) |
| Stream tokens to the TUI | [`libs/cli/deepagents_cli/remote_client.md`](./libs/cli/deepagents_cli/remote_client.md) |
| Render a tool call in the TUI | [`libs/cli/deepagents_cli/widgets/tool_widgets.md`](./libs/cli/deepagents_cli/widgets/tool_widgets.md) |
| Add a slash command | [`libs/cli/deepagents_cli/command_registry.md`](./libs/cli/deepagents_cli/command_registry.md) |
| Persist sessions | [`libs/cli/deepagents_cli/sessions.md`](./libs/cli/deepagents_cli/sessions.md) |
| Add an MCP server | [`libs/cli/deepagents_cli/mcp_tools.md`](./libs/cli/deepagents_cli/mcp_tools.md) |
| Compare deepagents-cli to Claude Code / Codex | [`libs/cli/cli_architecture.md`](./libs/cli/cli_architecture.md) |

---

## Conventions used in this tree

- Every directory has a `README.md` (the conceptual index for that
  directory).
- Every documented source file `<package>/<file>.py` has a corresponding
  `docs/<package>/<file>.md`.
- Each file doc is structured as: **summary → position in system → imports →
  per-block walkthroughs → flow diagrams (if any) → gotchas**.
- System prompts and tool descriptions are quoted **verbatim** in fenced
  code blocks. They are part of the design and you cannot understand the
  agent without them.
- Internal links are relative; `prerequisites/langchain.md` is referenced
  from anywhere as `../../prerequisites/langchain.md` (or wherever the
  level-up path leads).

---

## What's *not* covered (yet)

The following are explicitly out of scope for the current pass:

- `.github/workflows/` (CI/CD pipelines)
- `action.yml` and release infrastructure
- `release-please-config.json`
- Tests (other than where they reveal usage patterns)

These will be addressed in a future pass. See `book_diary.md` at the repo
root for the rolling status of the documentation effort.
