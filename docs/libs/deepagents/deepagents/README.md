# `deepagents/` — Core SDK Package

## Overview

The `deepagents` package is the root of the deepagents SDK. It provides everything needed to create and run AI agents with built-in capabilities for file management, subagent delegation, memory, skills, and conversation compaction.

## File and Directory Descriptions

| Path | Description |
|---|---|
| `__init__.py` | Top-level public API: re-exports `create_deep_agent` and key middleware classes |
| `_version.py` | Single-source version string (`"0.5.6"`) |
| `_models.py` | Helpers for resolving model strings to `BaseChatModel` instances |
| `graph.py` | `create_deep_agent()` — the primary entry point for building a configured agent |
| `backends/` | Pluggable storage and execution backends (state, filesystem, store, sandbox) |
| `middleware/` | Agent middleware providing tools, system prompt injection, and request transformation |

## Package Architecture

```
create_deep_agent() [graph.py]
    │
    ├── Model resolution [_models.py]
    │
    ├── Backend [backends/]
    │   ├── StateBackend (default — ephemeral, in LangGraph state)
    │   ├── FilesystemBackend (direct filesystem access)
    │   ├── StoreBackend (persistent cross-thread via LangGraph BaseStore)
    │   ├── LangSmithSandbox (LangSmith sandbox execution)
    │   ├── LocalShellBackend (local shell + filesystem)
    │   └── CompositeBackend (routes by path prefix)
    │
    └── Middleware stack [middleware/]
        ├── TodoListMiddleware (from langchain)
        ├── SkillsMiddleware — loads SKILL.md skill catalogs
        ├── FilesystemMiddleware — ls/read/write/edit/glob/grep/execute tools
        ├── SubAgentMiddleware — task tool for synchronous subagent delegation
        ├── SummarizationMiddleware — automatic context compaction
        ├── PatchToolCallsMiddleware — fixes dangling tool calls
        ├── AsyncSubAgentMiddleware — async tasks on remote LangGraph deployments
        ├── AnthropicPromptCachingMiddleware (from langchain-anthropic)
        ├── MemoryMiddleware — AGENTS.md memory injection
        └── HumanInTheLoopMiddleware (from langchain)
```

## Quick Start

```python
from deepagents import create_deep_agent

# Create a deep agent with defaults (Claude Sonnet 4.6, StateBackend)
agent = create_deep_agent()

# Invoke with a thread config
result = agent.invoke(
    {"messages": [{"role": "user", "content": "Hello!"}]},
    config={"configurable": {"thread_id": "my-thread"}}
)
```

## Key Design Decisions

- **`StateBackend` is the default:** Files are stored in LangGraph agent state, which is checkpointed automatically. Files persist within a thread but not across threads.
- **Middleware-first architecture:** Capabilities are added through middleware rather than custom tool implementations. This ensures consistent behavior across different agent configurations.
- **Subagent isolation:** Sub-agents run with isolated message histories but share the parent's filesystem state. Results are returned as a single final message.
- **Progressive disclosure for skills:** Skill metadata is injected at the start; full instructions are read on demand via `read_file`. This conserves context window space.
