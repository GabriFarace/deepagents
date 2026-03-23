# `deepagents/middleware/memory.py`

## High-Level Purpose

`MemoryMiddleware` implements support for the [AGENTS.md specification](https://agents.md/). It loads memory/context files from configurable backend paths and injects their content into the system prompt on every LLM call. Unlike skills (on-demand workflows), memory is always loaded and provides persistent context for the agent.

## Key Concepts

- **Memory sources** — Simple file paths to `AGENTS.md`-style Markdown files. Multiple sources are loaded in order and concatenated.
- **Lazy loading** — Memory is loaded once per conversation (in `before_agent`) and cached in `MemoryState`. Subsequent turns skip the load.
- **System prompt injection** — Formatted memory is appended to the system message on every `wrap_model_call`. The agent is instructed to update memories when it learns new preferences.

## TypedDicts and State

### `MemoryState(AgentState)`
State schema for `MemoryMiddleware`.
- `memory_contents: NotRequired[Annotated[dict[str, str], PrivateStateAttr]]` — Dict mapping source paths to their loaded content. Marked as `PrivateStateAttr` so it is not included in the output passed to parent agents.

### `MemoryStateUpdate`
State update shape: `{"memory_contents": dict[str, str]}`.

## `MEMORY_SYSTEM_PROMPT`

A large prompt template injected into the system message. Key content:
- Wraps memory in `<agent_memory>` XML tags.
- Provides `<memory_guidelines>` with detailed instructions on when and how to update memories.
- Specifically teaches the agent to update memory as its FIRST action when relevant.
- Lists examples of what to remember (preferences, credentials) vs. what NOT to remember (transient info, API keys).

## Class: `MemoryMiddleware(AgentMiddleware[MemoryState, ContextT, ResponseT])`

### Constructor

```python
MemoryMiddleware(
    *,
    backend: BACKEND_TYPES,
    sources: list[str],
)
```

**Parameters:**
- `backend` — Backend instance or factory function.
- `sources` — List of file paths to load (e.g., `["~/.deepagents/AGENTS.md", "./.deepagents/AGENTS.md"]`). Display names are derived from paths. Sources are loaded in order and concatenated.

### Methods

#### `before_agent(state, runtime, config) -> MemoryStateUpdate | None`
Synchronous pre-agent hook. Loads all memory sources via `backend.download_files()`. Skips loading if `memory_contents` is already in state (prevents redundant loads on subsequent turns). Returns `None` if already loaded, otherwise returns `MemoryStateUpdate`.

#### `abefore_agent(state, runtime, config) -> MemoryStateUpdate | None`
Async version. Uses `backend.adownload_files()`.

#### `modify_request(request) -> ModelRequest`
Formats loaded memory contents via `_format_agent_memory` and appends to the system message using `append_to_system_message`.

#### `wrap_model_call(request, handler) -> ModelResponse`
Calls `modify_request` then forwards to `handler`.

#### `awrap_model_call(request, handler) -> ModelResponse`
Async version.

#### `_get_backend(state, runtime, config) -> BackendProtocol`
Resolves the backend: if `self._backend` is callable (factory), constructs a `ToolRuntime` and calls it; otherwise returns the backend directly.

#### `_format_agent_memory(contents: dict[str, str]) -> str`
Formats memory with section separators. If no content loaded, returns `"(No memory loaded)"`. Otherwise, combines path + content pairs for each source and formats via `MEMORY_SYSTEM_PROMPT`.

## Dependencies

- `langchain.agents.middleware.types` — `AgentMiddleware`, `AgentState`, `PrivateStateAttr`
- `deepagents.backends.protocol` — `BACKEND_TYPES`, `BackendProtocol`
- `deepagents.middleware._utils.append_to_system_message`
