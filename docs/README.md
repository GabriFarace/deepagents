# deepagents — Documentation Index

> Auto-generated documentation for the deepagents monorepo.
> Each file in `docs/` mirrors the source file it documents, maintaining the same directory hierarchy.

## What is deepagents?

**deepagents** is an open-source, batteries-included AI agent framework built on [LangGraph](https://github.com/langchain-ai/langgraph). It provides:

- A Python SDK (`deepagents`) for building LLM agents with file access, shell execution, memory, skills, and subagents
- An interactive TUI CLI (`deepagents-cli`) for using those agents from the terminal
- An ACP protocol adapter (`deepagents-acp`) for remote/embedded use
- A benchmark suite (`deepagents-evals`) for quality measurement
- Pluggable sandbox integrations for Modal, Daytona, Runloop, and QuickJS

**Think of it as a Claude Code-inspired, open-source, provider-agnostic agent harness.**

---

## Documentation Structure

```
docs/
├── README.md                    ← You are here
├── ROADMAP.md                   ← Learning roadmap (start here!)
├── libs/
│   ├── README.md                ← Overview of all library packages
│   ├── deepagents/              ← Core SDK documentation
│   ├── cli/                     ← CLI tool documentation
│   ├── acp/                     ← ACP adapter documentation
│   ├── evals/                   ← Evaluation suite documentation
│   └── partners/                ← Sandbox provider documentation
├── examples/                    ← Example agent documentation
└── devops/                      ← CI/CD and DevOps documentation
```

---

## Quick Navigation

### By Layer (top to bottom)

| Layer | Package | Docs |
|---|---|---|
| User Interface | `deepagents-cli` | [docs/libs/cli/README.md](libs/cli/README.md) |
| Protocol Adapter | `deepagents-acp` | [docs/libs/acp/README.md](libs/acp/README.md) |
| Core SDK | `deepagents` | [docs/libs/deepagents/README.md](libs/deepagents/README.md) |
| Backends | SDK backends + partners | [docs/libs/deepagents/deepagents/backends/README.md](libs/deepagents/deepagents/backends/README.md) |
| Middleware | SDK middleware | [docs/libs/deepagents/deepagents/middleware/README.md](libs/deepagents/deepagents/middleware/README.md) |
| Sandboxes | Partner packages | [docs/libs/partners/README.md](libs/partners/README.md) |
| Evaluation | `deepagents-evals` | [docs/libs/evals/README.md](libs/evals/README.md) |
| Examples | `examples/` | [docs/examples/README.md](examples/README.md) |
| DevOps | CI/CD | [docs/devops/README.md](devops/README.md) |

### By Concept

| Concept | Where to Look |
|---|---|
| Creating your first agent | [deepagents/graph.md](libs/deepagents/deepagents/graph.md) |
| Backends (file/shell access) | [deepagents/backends/README.md](libs/deepagents/deepagents/backends/README.md) |
| Middleware (extending agents) | [deepagents/middleware/README.md](libs/deepagents/deepagents/middleware/README.md) |
| Subagents | [deepagents/middleware/subagents.md](libs/deepagents/deepagents/middleware/subagents.md) |
| Skills (reusable workflows) | [deepagents/middleware/skills.md](libs/deepagents/deepagents/middleware/skills.md) |
| Memory (AGENTS.md) | [deepagents/middleware/memory.md](libs/deepagents/deepagents/middleware/memory.md) |
| TUI Application | [cli/deepagents_cli/app.md](libs/cli/deepagents_cli/app.md) |
| Model configuration | [cli/deepagents_cli/model_config.md](libs/cli/deepagents_cli/model_config.md) |
| MCP tools | [cli/deepagents_cli/mcp_tools.md](libs/cli/deepagents_cli/mcp_tools.md) |
| Human-in-the-loop | [cli/deepagents_cli/widgets/approval.md](libs/cli/deepagents_cli/widgets/approval.md) |
| ACP protocol | [acp/deepagents_acp/server.md](libs/acp/deepagents_acp/server.md) |
| CI/CD pipeline | [devops/github-workflows/ci.md](devops/github-workflows/ci.md) |
| Release process | [devops/github-workflows/release.md](devops/github-workflows/release.md) |

---

## Key Design Principles

1. **Provider-agnostic** — Works with any LangChain-supported LLM (Anthropic, OpenAI, Google, Ollama, etc.)
2. **Uniform backend protocol** — All storage accessed via `BackendProtocol`, making middleware storage-agnostic
3. **Middleware for extensibility** — New capabilities added as middleware, not by modifying the core graph
4. **Subagent isolation** — Sub-agents share filesystem state but have independent message histories
5. **Progressive disclosure for skills** — Skill metadata always visible; full instructions read on demand
6. **Batteries included** — The CLI ships with everything needed for a full coding assistant experience

---

## Learning Path

See **[ROADMAP.md](ROADMAP.md)** for a structured guide on how to learn this codebase from the ground up.
