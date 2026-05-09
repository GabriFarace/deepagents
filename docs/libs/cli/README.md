# `libs/cli/`

> The `deepagents-cli` package: a Textual terminal UI that starts a LangGraph
> server subprocess, talks to it over HTTP/SSE, and wraps SDK agents with
> sessions, approvals, MCP tools, skills, hooks, and deployment helpers.

## Position in the system

The CLI is intentionally split into two processes:

```
terminal process
  ├─ parses args and config
  ├─ renders Textual UI / non-interactive output
  ├─ starts langgraph dev in a temp workspace
  └─ talks to RemoteAgent over HTTP/SSE

server subprocess
  ├─ imports server_graph.py
  ├─ reads DEEPAGENTS_CLI_SERVER_* env vars
  ├─ builds model/tools/backend/MCP/sandbox
  └─ serves the compiled deep agent graph
```

This split is the main architectural difference from a simple single-process
agent CLI. It gives the CLI LangGraph persistence, remote-client parity,
server-side streaming, and a clean place to run deployed or ACP-like graph
surfaces, at the cost of startup orchestration.

## Files

| Path | Doc | Role |
|---|---|---|
| `cli_architecture.md` | [`cli_architecture.md`](./cli_architecture.md) | Generic agent CLI architecture and deepagents-cli tradeoffs. |
| `deepagents_cli/main.py` | [`deepagents_cli/main.md`](./deepagents_cli/main.md) | CLI entry point, args, modes, startup flow. |
| `deepagents_cli/server_manager.py` | [`deepagents_cli/server_manager.md`](./deepagents_cli/server_manager.md) | Parent-side server workspace/env orchestration. |
| `deepagents_cli/server.py` | [`deepagents_cli/server.md`](./deepagents_cli/server.md) | Low-level `langgraph dev` process lifecycle. |
| `deepagents_cli/server_graph.py` | [`deepagents_cli/server_graph.md`](./deepagents_cli/server_graph.md) | Server-side graph factory loaded by LangGraph. |
| `deepagents_cli/agent.py` | [`deepagents_cli/agent.md`](./deepagents_cli/agent.md) | CLI-specific `create_deep_agent()` wrapper. |
| `deepagents_cli/remote_client.py` | [`deepagents_cli/remote_client.md`](./deepagents_cli/remote_client.md) | HTTP/SSE client wrapper around `RemoteGraph`. |
| `deepagents_cli/app.py` | [`deepagents_cli/app.md`](./deepagents_cli/app.md) | Textual app runtime and layout. |
| `deepagents_cli/command_registry.py` | [`deepagents_cli/command_registry.md`](./deepagents_cli/command_registry.md) | Slash command registry and skill command expansion. |
| `deepagents_cli/sessions.py` | [`deepagents_cli/sessions.md`](./deepagents_cli/sessions.md) | SQLite checkpoint/session listing and metadata. |
| `deepagents_cli/hooks.py` | [`deepagents_cli/hooks.md`](./deepagents_cli/hooks.md) | User-configurable lifecycle hooks. |
| `deepagents_cli/mcp_tools.py` | [`deepagents_cli/mcp_tools.md`](./deepagents_cli/mcp_tools.md) | MCP config validation, sessions, and tool wrapping. |
| `deepagents_cli/widgets/` | [`deepagents_cli/widgets/`](./deepagents_cli/widgets/README.md) | Textual widgets for chat, approvals, messages, tools, selectors, and notifications. |

## Reading order

Read [`cli_architecture.md`](./cli_architecture.md), then the bootstrap path:
`main.py` → `server_manager.py` → `server.py` → `server_graph.py` →
`agent.py` → `remote_client.py` → `app.py`. After that, read widgets and
extension systems (`command_registry`, `sessions`, `hooks`, `mcp_tools`).
