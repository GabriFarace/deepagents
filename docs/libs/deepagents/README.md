# deepagents SDK — Documentation

## Overview

`deepagents` is a Python SDK for building powerful AI agents on top of [LangGraph](https://github.com/langchain-ai/langgraph). It provides a batteries-included agent factory with built-in capabilities for file management, subagent spawning, memory, skills, conversation compaction, and pluggable storage backends.

**Version:** `0.5.0a2`
**License:** MIT
**Python:** `>=3.11`
**Homepage:** https://docs.langchain.com/oss/python/deepagents/overview

## Quick Start

```python
from deepagents import create_deep_agent

agent = create_deep_agent()
result = agent.invoke(
    {"messages": [{"role": "user", "content": "Hello, what can you do?"}]},
    config={"configurable": {"thread_id": "session-1"}}
)
```

## Documentation Structure

```
docs/libs/deepagents/
├── README.md                          # This file
├── pyproject.toml.md                  # Package config, dependencies, tooling
├── Makefile.md                        # Developer workflow commands
└── deepagents/
    ├── README.md                      # Core package overview
    ├── __init__.md                    # Public API exports
    ├── _version.md                    # Version constant
    ├── _models.md                     # Model resolution helpers
    ├── graph.md                       # create_deep_agent() factory
    ├── backends/
    │   ├── README.md                  # Backend architecture overview
    │   ├── __init__.md                # Backend exports
    │   ├── protocol.md                # BackendProtocol and all result types
    │   ├── state.md                   # StateBackend (ephemeral, default)
    │   ├── filesystem.md              # FilesystemBackend (direct disk access)
    │   ├── sandbox.md                 # BaseSandbox (shell-based file ops)
    │   ├── langsmith.md               # LangSmithSandbox
    │   ├── local_shell.md             # LocalShellBackend
    │   ├── store.md                   # StoreBackend (persistent cross-thread)
    │   ├── composite.md               # CompositeBackend (path-prefix routing)
    │   └── utils.md                   # Shared helpers for all backends
    └── middleware/
        ├── README.md                  # Middleware architecture overview
        ├── __init__.md                # Middleware exports and design rationale
        ├── _utils.md                  # append_to_system_message helper
        ├── filesystem.md              # FilesystemMiddleware (file tools + eviction)
        ├── memory.md                  # MemoryMiddleware (AGENTS.md loading)
        ├── skills.md                  # SkillsMiddleware (skill catalog)
        ├── subagents.md               # SubAgentMiddleware (task tool)
        ├── async_subagents.md         # AsyncSubAgentMiddleware (remote tasks)
        ├── summarization.md           # SummarizationMiddleware (context compaction)
        └── patch_tool_calls.md        # PatchToolCallsMiddleware (dangling tool fixes)
```

## Core Concepts

### `create_deep_agent()`

The single entry point. Takes a model, tools, backend, subagents, middleware, skills, and memory configuration and returns a compiled LangGraph `CompiledStateGraph` ready for invocation.

See [`deepagents/graph.md`](deepagents/graph.md) for full parameter documentation.

### Backends

Backends define where files are stored and whether shell execution is available. The key design pattern: all backends implement `BackendProtocol` with a uniform interface (ls, read, write, edit, grep, glob, upload, download), making storage transparent to the middleware and tool layers.

| Backend | Persistence | Execution | Best For |
|---|---|---|---|
| `StateBackend` | Per-thread | No | Default, web APIs, testing |
| `FilesystemBackend` | Permanent | No | Local dev CLIs |
| `LocalShellBackend` | Permanent | Yes (unsafe) | Trusted local dev |
| `LangSmithSandbox` | Sandbox | Yes (isolated) | Production sandboxed execution |
| `StoreBackend` | Cross-thread | No | Agent memories, persistent docs |
| `CompositeBackend` | Mixed | Depends on default | Combining storage strategies |

See [`deepagents/backends/README.md`](deepagents/backends/README.md) for architecture details.

### Middleware

Middleware wraps every LLM call, enabling dynamic tool injection, system prompt modification, and cross-turn state management. The middleware layer is the primary extension point for the SDK.

Default middleware stack (in order of application in `create_deep_agent`):
1. `TodoListMiddleware` — planning and task list management
2. `SkillsMiddleware` — skill catalog injection (optional)
3. `FilesystemMiddleware` — all file and execution tools
4. `SubAgentMiddleware` — the `task` delegation tool
5. `SummarizationMiddleware` — automatic context compaction
6. `PatchToolCallsMiddleware` — conversation consistency fixes
7. `AsyncSubAgentMiddleware` — remote background task tools (optional)
8. User-provided middleware
9. `AnthropicPromptCachingMiddleware` — Anthropic prompt caching
10. `MemoryMiddleware` — AGENTS.md context loading (optional)
11. `HumanInTheLoopMiddleware` — HITL approval gates (optional)

See [`deepagents/middleware/README.md`](deepagents/middleware/README.md) for architecture details.

### Subagents

Three forms of subagents are supported:
- **`SubAgent`** (TypedDict) — declarative synchronous subagents, compiled at call-time
- **`CompiledSubAgent`** (TypedDict) — pre-compiled runnables used directly
- **`AsyncSubAgent`** (TypedDict) — remote agents on LangGraph deployments, non-blocking

A `general-purpose` subagent is always available (added automatically if not overridden), giving the main agent an isolated context window for complex tasks.

### Skills

Skills are reusable workflows stored as `SKILL.md` files in backend directories. They follow the [Agent Skills specification](https://agentskills.io/specification) with YAML frontmatter (name, description, license, compatibility). The agent discovers skills via the system prompt and reads full instructions on demand (progressive disclosure).

### Memory

Memory is loaded from `AGENTS.md` files via `MemoryMiddleware`. It is always injected into the system prompt and persists across conversations when using a persistent backend. The agent is instructed to update its memory files when it learns new user preferences.

## Configuration Files

- [`pyproject.toml.md`](pyproject.toml.md) — Package metadata, dependencies, and development tooling configuration
- [`Makefile.md`](Makefile.md) — Developer workflow commands for testing, linting, and formatting

## Key Design Principles

1. **Uniform backend protocol** — All storage is accessed through `BackendProtocol`, making the middleware and tool layers storage-agnostic.
2. **Middleware for extensibility** — New capabilities are added as middleware, not by modifying the core graph.
3. **Subagent isolation** — Sub-agents share filesystem state but have independent message histories, preventing context leakage.
4. **Factory pattern for backends** — Backends like `StateBackend` that need `ToolRuntime` are passed as factory classes and instantiated lazily at tool-call time.
5. **Progressive disclosure for skills** — Skill metadata (name + description) is always visible; full instructions are read on demand.
6. **Memory last** — Memory middleware is placed after caching middleware to avoid invalidating Anthropic prompt cache prefixes when memories are updated.
