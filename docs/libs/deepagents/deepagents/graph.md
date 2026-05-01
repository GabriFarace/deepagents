# `deepagents/graph.py`

## High-Level Purpose

This is the primary entry point for building a deep agent. It exports `create_deep_agent`, a factory function that assembles a fully configured `CompiledStateGraph` from LangGraph — wiring together the model, tools, backends, and all middleware layers (filesystem, subagents, summarization, skills, memory, caching, and human-in-the-loop).

## Constants

### `BASE_AGENT_PROMPT`

A large string constant that defines the default behavioral instructions for every deep agent. It covers:
- Core behavior (conciseness, no preamble)
- Professional objectivity (accuracy over validation)
- Task execution protocol (understand → act → verify)
- Progress update guidance

When a harness profile is active for the selected model, the profile may supply its own `base_system_prompt` (replacing `BASE_AGENT_PROMPT`) and/or a `system_prompt_suffix` appended after the base. The user-supplied `system_prompt` always comes first regardless.

## Functions

### `create_deep_agent(...) -> CompiledStateGraph`

The main factory function. Assembles a complete deep agent graph.

**Parameters (grouped by category):**

**Model / Output**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str \| BaseChatModel \| None` | `None` | LLM to use. Accepts provider-prefixed strings like `"openai:gpt-4o"` or pre-initialized `BaseChatModel` instances. **Deprecated:** passing `None` emits a `DeprecationWarning` and defaults to `claude-sonnet-4-6`; will be required in 1.0.0. |
| `response_format` | `ResponseFormat \| None` | `None` | Structured output schema for the agent's final response. |
| `context_schema` | `type[Any] \| None` | `None` | Schema for the agent's context object. |

**Behavior / Tools**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `tools` | `Sequence[BaseTool \| Callable \| dict] \| None` | `None` | Additional tools to give the agent beyond built-ins. |
| `system_prompt` | `str \| SystemMessage \| None` | `None` | Custom instructions placed **before** the base prompt. When a `SystemMessage`, preserves `cache_control` markers on content blocks (important for Anthropic prompt cache breakpoints). |
| `middleware` | `Sequence[AgentMiddleware]` | `()` | User middleware inserted between the base stack and the tail stack. |
| `subagents` | `Sequence[SubAgent \| CompiledSubAgent \| AsyncSubAgent] \| None` | `None` | Three forms: declarative sync (`SubAgent`), pre-compiled (`CompiledSubAgent`), remote async (`AsyncSubAgent` identified by a `"graph_id"` key). Subagents inherit `interrupt_on` and `permissions` from parent unless they override them. |
| `skills` | `list[str \| tuple[str, str]] \| None` | `None` | Skill source directories. Each entry is a path string or a `(path, label)` tuple for a labelled source. Later entries override earlier ones for same-named skills ("last wins"). |
| `memory` | `list[str] \| None` | `None` | Paths to `AGENTS.md` memory files. Display names are auto-derived from paths. |
| `permissions` | `list[FilesystemPermission] \| None` | `None` | Filesystem permission rules enforced at the tool level. First match wins; unmatched paths are allowed. Subagents inherit unless they specify their own (which fully replaces parent rules). |

**Infrastructure**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `backend` | `BackendProtocol \| BackendFactory \| None` | `None` | Storage/execution backend. Defaults to `StateBackend()` (ephemeral). For shell access, use a backend implementing `SandboxBackendProtocol`. |
| `interrupt_on` | `dict[str, bool \| InterruptOnConfig] \| None` | `None` | Tool names to pause on for human-in-the-loop review. |
| `checkpointer` | `Checkpointer \| None` | `None` | LangGraph checkpointer for persisting state between runs. |
| `store` | `BaseStore \| None` | `None` | Persistent store (required by `StoreBackend`). |
| `cache` | `BaseCache \| None` | `None` | LangGraph cache instance. |
| `debug` | `bool` | `False` | Enables debug mode in the underlying `create_agent` call. |
| `name` | `str \| None` | `None` | Name for the agent graph. |

**Returns:** A `CompiledStateGraph` configured with a `recursion_limit` of 1000 and metadata tagging the agent as a `deepagents` integration.

## Middleware Stack Assembly

The middleware stack is divided into three sections assembled in this order:

### 1. Base Stack

| Middleware | Condition |
|---|---|
| `TodoListMiddleware` | Always |
| `SkillsMiddleware` | If `skills` provided |
| `FilesystemMiddleware` | Always |
| `SubAgentMiddleware` | If inline subagents present |
| `SummarizationMiddleware` | Always |
| `PatchToolCallsMiddleware` | Always |
| `AsyncSubAgentMiddleware` | If async subagents present |

### 2. User Middleware

User-supplied `middleware` is inserted here, between base and tail.

### 3. Tail Stack

| Middleware | Condition |
|---|---|
| Profile `extra_middleware` | If harness profile provides extra middleware |
| `_ToolExclusionMiddleware` | If profile has `excluded_tools` |
| `AnthropicPromptCachingMiddleware` | Always (unconditional) |
| `MemoryMiddleware` | If `memory` provided |
| `HumanInTheLoopMiddleware` | If `interrupt_on` provided |

## Harness Profiles

`_harness_profile_for_model()` returns a `_HarnessProfile` for the resolved model. A profile can supply:
- `base_system_prompt` — replaces `BASE_AGENT_PROMPT` for that model
- `system_prompt_suffix` — appended after the base prompt
- `tool_description_overrides` — per-tool description text overrides
- `extra_middleware` — additional tail-stack middleware
- `excluded_tools` — tool names to drop from the tool list
- `excluded_middleware` — middleware to exclude from the assembled stack (validated; raises `ValueError` if nothing matches)

> **Required middleware:** `FilesystemMiddleware` and `SubAgentMiddleware` cannot be excluded; attempting to do so raises `ValueError`.

## General-Purpose Subagent Auto-Add

Unless the harness profile disables it, or the user already provides a subagent named `"general-purpose"`, a default `general-purpose` subagent is automatically inserted. It receives the same base middleware stack (TodoList, Filesystem, Summarization, PatchToolCalls, Skills if applicable, AnthropicPromptCaching, HITL if applicable).

## System Prompt Assembly Order

1. User-supplied `system_prompt` (first)
2. `BASE_AGENT_PROMPT` (or profile `base_system_prompt`)
3. Profile `system_prompt_suffix` (last, if any)

`str` prompts are concatenated with `"\n\n"`. `SystemMessage` objects have new text content blocks appended (preserving any existing `cache_control` markers).

## Dependencies

- `langchain.agents.create_agent` — core agent graph builder
- `langchain.agents.middleware.*` — `HumanInTheLoopMiddleware`, `TodoListMiddleware`, `InterruptOnConfig`
- `langchain_anthropic.middleware.AnthropicPromptCachingMiddleware`
- `langgraph.store.base.BaseStore`, `langgraph.cache.base.BaseCache`, `langgraph.types.Checkpointer`
- `deepagents._models.resolve_model`
- `deepagents.backends.*` — `StateBackend`, `BackendProtocol`, `BackendFactory`
- `deepagents.middleware.*` — all middleware classes
- `deepagents.middleware.filesystem.FilesystemPermission`
