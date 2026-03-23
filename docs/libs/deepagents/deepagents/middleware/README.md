# `deepagents/middleware/` — Agent Middleware

## Overview

The `middleware` package provides the behavior-level components that augment the agent with capabilities beyond bare LLM tool calling. Each middleware subclasses `AgentMiddleware` and gets to intercept every LLM request via `wrap_model_call()` before it is sent, and optionally run logic before each agent step via `before_agent()`.

## File Descriptions

| File | Class | Purpose |
|---|---|---|
| `__init__.py` | — | Re-exports all public middleware; explains middleware vs. plain tools |
| `_utils.py` | — | `append_to_system_message` helper for all middleware |
| `filesystem.py` | `FilesystemMiddleware` | Provides ls/read/write/edit/glob/grep/execute tools; handles large result eviction |
| `memory.py` | `MemoryMiddleware` | Loads AGENTS.md files and injects into system prompt; teaches agent to update memories |
| `skills.py` | `SkillsMiddleware` | Loads skill metadata from SKILL.md files; injects skills catalog with progressive disclosure |
| `subagents.py` | `SubAgentMiddleware` | Provides `task` tool for delegating to ephemeral synchronous sub-agents |
| `async_subagents.py` | `AsyncSubAgentMiddleware` | Provides 5 tools for managing background tasks on remote LangGraph deployments |
| `summarization.py` | `SummarizationMiddleware`, `SummarizationToolMiddleware` | Automatic and on-demand conversation compaction |
| `patch_tool_calls.py` | `PatchToolCallsMiddleware` | Fixes dangling tool calls when user messages interrupt before tool results arrive |

## How Files Relate

All middleware files follow the same pattern:
1. Define a state schema extending `AgentState` (optional, for stateful middleware)
2. Implement `before_agent()` for pre-run initialization (e.g., loading skills or memory)
3. Implement `wrap_model_call()` to intercept LLM requests and inject tools/system prompt
4. Use `_utils.append_to_system_message` for system prompt injection

### Dependency Graph

```
filesystem.py
  ├── deepagents.backends.*  (all backends)
  └── deepagents.backends.utils

memory.py
  └── deepagents.backends.protocol

skills.py
  └── deepagents.backends.protocol

subagents.py
  └── deepagents.backends.protocol

async_subagents.py
  └── langgraph_sdk

summarization.py
  └── deepagents.backends.protocol
  └── langchain.agents.middleware.summarization

patch_tool_calls.py
  └── (no deepagents deps; only langchain/langgraph)

All middleware:
  └── _utils.append_to_system_message
```

## Default Middleware Stack

`create_deep_agent()` assembles this stack for the main agent (in order):
1. `TodoListMiddleware` (from langchain)
2. `SkillsMiddleware` (if `skills` provided)
3. `FilesystemMiddleware`
4. `SubAgentMiddleware`
5. `SummarizationMiddleware`
6. `PatchToolCallsMiddleware`
7. `AsyncSubAgentMiddleware` (if async subagents provided)
8. User-provided `middleware`
9. `AnthropicPromptCachingMiddleware`
10. `MemoryMiddleware` (if `memory` provided)
11. `HumanInTheLoopMiddleware` (if `interrupt_on` provided)

Memory and caching are placed last so memory updates do not invalidate the Anthropic prompt cache prefix.
