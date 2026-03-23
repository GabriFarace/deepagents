# `deepagents/middleware/__init__.py`

## High-Level Purpose

Package initializer for `deepagents.middleware`. It re-exports all public middleware classes and explains the architectural rationale for the middleware approach.

## Design Philosophy (from module docstring)

The middleware package exists because LLM tools (plain callables in `tools=[]`) can only be invoked _by_ the LLM, not _before_ an LLM call. Middleware subclasses `AgentMiddleware` and overrides `wrap_model_call()`, which intercepts every LLM request. This enables middleware to:

- **Filter tools dynamically** — `FilesystemMiddleware` removes the `execute` tool at call-time when the backend doesn't support it.
- **Inject system-prompt context** — `MemoryMiddleware` and `SkillsMiddleware` add relevant instructions on every call.
- **Transform messages** — `SummarizationMiddleware` counts tokens, truncates old tool arguments, and replaces history with summaries.
- **Maintain cross-turn state** — Middleware can read/write typed state dicts that persist across agent turns.

**Use middleware when:** tools need to modify the system prompt/tool list, track state across turns, or be available to all SDK consumers.
**Use plain tools when:** the function is stateless, self-contained, and consumer-specific.

## Exports

| Symbol | Source | Purpose |
|---|---|---|
| `AsyncSubAgent` | `async_subagents` | TypedDict for remote LangGraph subagent specs |
| `AsyncSubAgentMiddleware` | `async_subagents` | Manages async background subagent tasks |
| `CompiledSubAgent` | `subagents` | TypedDict for pre-compiled runnable subagents |
| `FilesystemMiddleware` | `filesystem` | Provides ls/read/write/edit/glob/grep/execute tools |
| `MemoryMiddleware` | `memory` | Loads AGENTS.md files into the system prompt |
| `SkillsMiddleware` | `skills` | Loads skill metadata into the system prompt |
| `SubAgent` | `subagents` | TypedDict for declarative synchronous subagent specs |
| `SubAgentMiddleware` | `subagents` | Provides the `task` tool for subagent delegation |
| `SummarizationMiddleware` | `summarization` | Auto-compacts conversation when context fills |
| `SummarizationToolMiddleware` | `summarization` | Exposes `compact_conversation` tool |
| `create_summarization_tool_middleware` | `summarization` | Convenience factory for summarization middleware |
