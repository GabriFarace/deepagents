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

This prompt is always appended to any custom `system_prompt` provided by the caller.

## Functions

### `get_default_model() -> ChatAnthropic`

Returns the default LLM used when no `model` is passed to `create_deep_agent`.

**Returns:** A `ChatAnthropic` instance configured with `claude-sonnet-4-6`.

---

### `create_deep_agent(...) -> CompiledStateGraph`

The main factory function. Assembles a complete deep agent graph.

**Parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str \| BaseChatModel \| None` | `None` | LLM to use. Defaults to Claude Sonnet 4.6. Accepts provider-prefixed strings like `"openai:gpt-4o"`. |
| `tools` | `Sequence[BaseTool \| Callable \| dict] \| None` | `None` | Additional tools to give the agent beyond built-ins. |
| `system_prompt` | `str \| SystemMessage \| None` | `None` | Custom instructions prepended before the base prompt. |
| `middleware` | `Sequence[AgentMiddleware]` | `()` | Extra middleware inserted after the base stack but before caching and memory. |
| `subagents` | `Sequence[SubAgent \| CompiledSubAgent \| AsyncSubAgent] \| None` | `None` | Subagent specifications. Auto-adds a `general-purpose` subagent if none provided. |
| `skills` | `list[str] \| None` | `None` | POSIX paths to skill source directories. |
| `memory` | `list[str] \| None` | `None` | Paths to `AGENTS.md` memory files. |
| `response_format` | `ResponseFormat \| None` | `None` | Structured output schema for the agent's final response. |
| `context_schema` | `type[Any] \| None` | `None` | Schema for the agent's context object. |
| `checkpointer` | `Checkpointer \| None` | `None` | LangGraph checkpointer for persisting state between runs. |
| `store` | `BaseStore \| None` | `None` | Persistent store (required by `StoreBackend`). |
| `backend` | `BackendProtocol \| BackendFactory \| None` | `None` | Storage/execution backend. Defaults to `StateBackend`. |
| `interrupt_on` | `dict[str, bool \| InterruptOnConfig] \| None` | `None` | Tool names to pause execution on for HITL review. |
| `debug` | `bool` | `False` | Enables debug mode in the underlying `create_agent` call. |
| `name` | `str \| None` | `None` | Name for the agent graph. |
| `cache` | `BaseCache \| None` | `None` | LangGraph cache instance. |

**Returns:** A `CompiledStateGraph` configured with a `recursion_limit` of `1000` and metadata tagging the agent as a `deepagents` integration.

**Key Logic (assembly order):**

1. **Model resolution** — `resolve_model` converts string specs to `BaseChatModel` instances. Falls back to `get_default_model()`.
2. **Backend setup** — Defaults to `StateBackend` if not provided.
3. **General-purpose subagent** — A `general-purpose` subagent is built with the full base middleware stack (TodoList, Filesystem, Summarization, PatchToolCalls, Skills if applicable, AnthropicPromptCaching, HITL if applicable). It is inserted unless a subagent named `"general-purpose"` is already in the provided list.
4. **Subagent separation** — Iterates `subagents` and splits them into `inline_subagents` (sync: `SubAgent` + `CompiledSubAgent`) and `async_subagents` (`AsyncSubAgent` identified by a `"graph_id"` key).
5. **Subagent middleware** — Each `SubAgent` gets its own copy of the base middleware stack prepended before any user-specified `middleware`.
6. **Main agent middleware stack** is assembled in this order:
   - `TodoListMiddleware`
   - `SkillsMiddleware` (if `skills` provided)
   - `FilesystemMiddleware`
   - `SubAgentMiddleware`
   - `SummarizationMiddleware`
   - `PatchToolCallsMiddleware`
   - `AsyncSubAgentMiddleware` (if async subagents present)
   - User-provided `middleware`
   - `AnthropicPromptCachingMiddleware`
   - `MemoryMiddleware` (if `memory` provided)
   - `HumanInTheLoopMiddleware` (if `interrupt_on` provided)
7. **System prompt** — Merges the user's `system_prompt` with `BASE_AGENT_PROMPT`. Strings are concatenated with `\n\n`; `SystemMessage` objects have a new text content block appended.
8. **Graph creation** — Delegates to LangChain's `create_agent(...)` and wraps the result with `with_config({"recursion_limit": 1000, "metadata": {...}})`.

## Dependencies

- `langchain.agents.create_agent` — core agent graph builder
- `langchain.agents.middleware.*` — `HumanInTheLoopMiddleware`, `TodoListMiddleware`, `InterruptOnConfig`
- `langchain_anthropic.middleware.AnthropicPromptCachingMiddleware` — Anthropic-specific prompt caching
- `langgraph.store.base.BaseStore`, `langgraph.cache.base.BaseCache`, `langgraph.types.Checkpointer`
- `deepagents._models.resolve_model`
- `deepagents.backends.*` — `StateBackend`, `BackendProtocol`, `BackendFactory`
- `deepagents.middleware.*` — all middleware classes
