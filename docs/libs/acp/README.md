# `libs/acp/` — deepagents-acp Package

`deepagents-acp` bridges the deepagents SDK with the **Agent Client Protocol (ACP)**, allowing any compiled agent to be exposed as an ACP server. ACP clients can then control the agent via sessions, send prompts, stream responses, and configure options.

---

## High-Level Architecture

```
ACP Client
    │  ACP wire protocol (HTTP + SSE)
    ▼
AgentServerACP
    │  bridges ACP ↔ LangGraph
    ▼
CompiledStateGraph  (any create_deep_agent() result)
```

---

## Key Class

### `AgentServerACP` (`deepagents_acp/server.py`)

Wraps a `CompiledStateGraph` (or a factory function) and exposes it over ACP.

**Key methods:**

| Method | ACP operation | Description |
|---|---|---|
| `initialize()` | `initialize` | Returns server capabilities and available config options |
| `new_session()` | `new_session` | Creates a session with cwd and MCP config |
| `prompt(session_id, message)` | `prompt` | Streams a response to a user message |
| `set_session_config_option()` | `config` | Changes mode or model for a session |

**Session tracking:**
- Each session has an isolated `AgentSessionContext`: `cwd`, `mode`, `model`
- Modes and models are only offered as options when the agent was created from a factory (not pre-compiled), allowing dynamic selection

**HITL in ACP mode:**
- When an interrupt is triggered, `AgentServerACP` calls `client.request_permission()` on the ACP client
- The ACP client prompts the user and returns the decision
- `AgentServerACP` resumes the graph with the decision

---

## Safety Features

`AgentServerACP` includes a pattern-based command filter that rejects obviously dangerous shell commands before they reach the agent:
- `rm -rf /`
- `sudo` commands
- Output redirection to `/dev/null 2>&1` (hiding output)
- Other patterns flagged as high-risk

---

## Usage Example

```python
from deepagents import create_deep_agent
from deepagents_acp import AgentServerACP

graph = create_deep_agent()
server = AgentServerACP(agent=graph, host="0.0.0.0", port=8080)
await server.run()
```

Or with CLI:
```bash
deepagents --acp
```

---

## See Also

- [../../libs/deepagents/deepagents/graph.md](../deepagents/deepagents/graph.md) — creates the graph wrapped by ACP
- [../../libs/cli/deepagents_cli/main.md](../cli/deepagents_cli/main.md) — `--acp` flag in CLI
