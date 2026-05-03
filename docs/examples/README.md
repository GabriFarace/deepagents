# `examples/` — Example Agents

Each subdirectory is a self-contained example demonstrating a specific deepagents pattern. Examples are ordered from simple to complex.

---

## Example Catalog

| Directory | Pattern demonstrated | Complexity |
|---|---|---|
| `deep_research/` | Basic research agent with web search | Beginner |
| `content-builder-agent/` | Memory + skills + subagents for content creation | Intermediate |
| `text-to-sql-agent/` | NL-to-SQL query builder | Intermediate |
| `deploy-coding-agent/` | Autonomous coder with sandbox backend | Intermediate |
| `deploy-content-writer/` | Per-user memory via Supabase auth | Intermediate |
| `nvidia_deep_agent/` | Multi-model architecture (frontier + Nemotron) with GPU execution | Advanced |
| `deploy-gtm-agent/` | GTM strategist coordinating sync + async subagents | Advanced |
| `async-subagent-server/` | Self-hosted Agent Protocol server as async subagent | Advanced |
| `repl_swarm/` | Parallel subagent dispatch from REPL | Advanced |
| `rlm_agent/` | Recursive REPL Mode — nested agents with decreasing depth | Advanced |
| `better-harness/` | Autonomous harness optimization loop | Expert |

---

## Getting Started with an Example

```bash
cd examples/deep_research
uv run python main.py
```

Each example has its own `README.md` with setup instructions and a description of what it demonstrates.

---

## Common Patterns Illustrated

**Memory + skills (content-builder-agent):**
- `AGENTS.md` for project memory
- `SKILL.md` files for reusable prompt templates
- Subagents for parallel content generation

**Sandbox deployment (deploy-coding-agent):**
- Using a partner sandbox backend (Modal, Daytona) for untrusted code
- `create_deep_agent()` with a non-local backend

**Multi-model architecture (nvidia_deep_agent):**
- Passing different models to the parent agent and subagents
- GPU code execution via custom sandbox

**Async coordination (deploy-gtm-agent):**
- `AsyncSubAgent` for non-blocking background tasks
- `start_async_task` + `check_async_task` polling pattern

---

## See Also

- [../libs/deepagents/deepagents/graph.md](../libs/deepagents/deepagents/graph.md) — `create_deep_agent()` used in all examples
- [../libs/deepagents/deepagents/middleware/subagents.md](../libs/deepagents/deepagents/middleware/subagents.md) — subagent patterns
