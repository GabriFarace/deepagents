# `examples/`

> Short guide to the example agents shipped with Deep Agents.

The examples are not meant to document every source block. They show common ways
to assemble the SDK, CLI, deploy configuration, skills, memory files, MCP
servers, subagents, and backends into working agents.

## Examples at a glance

| Example | Demonstrates |
| --- | --- |
| [`deep_research`](deep_research.md) | Web research orchestration with a dedicated research subagent, Tavily search, and reflection. |
| [`content-builder-agent`](content-builder-agent.md) | File-configured content agent using memory, skills, a filesystem backend, image tools, and a YAML-defined research subagent. |
| [`text-to-sql-agent`](text-to-sql-agent.md) | Natural-language SQL over Chinook using LangChain SQL toolkit tools and workflow skills. |
| [`deploy-coding-agent`](deploy-coding-agent.md) | `deepagents deploy` for a coding assistant running in a LangSmith sandbox. |
| [`deploy-content-writer`](deploy-content-writer.md) | Deployed content writer with skills, Supabase auth, and per-user memory. |
| [`deploy-mcp-docs-agent`](deploy-mcp-docs-agent.md) | Deployed documentation researcher that uses an MCP docs server before answering. |
| [`deploy-gtm-agent`](deploy-gtm-agent.md) | Deployed GTM strategist combining supervisor instructions, skills, MCP, and subagents. |
| [`async-subagent-server`](async-subagent-server.md) | Self-hosted Agent Protocol server used as an async subagent by a supervisor. |
| [`nvidia_deep_agent`](nvidia_deep_agent.md) | Multi-model agent with NVIDIA-hosted subagents and a routed GPU/CPU backend. |
| [`ralph_mode`](ralph_mode.md) | Autonomous CLI loop with fresh context per iteration and filesystem/git persistence. |
| [`rlm_agent`](rlm_agent.md) | Recursive compiled subagent chain plus QuickJS REPL parallel tool calls. |
| [`repl_swarm`](repl_swarm.md) | Skill-packaged TypeScript swarm helper that fans out `task` calls from the REPL. |
| [`downloading_agents`](downloading_agents.md) | Agent distribution as ordinary folders or zip files. |
| [`better-harness`](better-harness.md) | Eval-driven outer loop where one Deep Agent edits another agent's harness surfaces. |

## How to read this section

Start with `deep_research` for the canonical SDK pattern: a supervisor prompt,
custom tools, and a specialized subagent passed to `create_deep_agent`. Then read
`content-builder-agent` and `text-to-sql-agent` to see how skills and memory turn
that skeleton into task-specific workflows.

The `deploy-*` examples are configuration-first. They are best read after the
CLI and deploy docs because their main source of behavior is `AGENTS.md`,
`deepagents.toml`, `mcp.json`, `skills/`, and `subagents/`, not Python
construction code.

The remaining examples are advanced patterns: async Agent Protocol delegation,
recursive compiled agents, QuickJS skill modules, autonomous looping, GPU
backends, and eval-driven harness optimization.
