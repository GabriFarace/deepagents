# `partners/quickjs/langchain_quickjs/middleware.py`

> ``REPLMiddleware``: exposes a persistent JavaScript REPL as an agent tool.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `asyncio, contextlib, logging, uuid, collections.abc, typing, deepagents.middleware._utils, langchain.agents.middleware.types, langchain.tools, langchain_core._api, langchain_core.messages, langchain_core.tools` and other helpers.
Module constants worth noticing: `_DEFAULT_MEMORY_LIMIT`, `_DEFAULT_TIMEOUT`, `_DEFAULT_MAX_PTC_CALLS`, `_DEFAULT_MAX_RESULT_CHARS`, `_DEFAULT_TOOL_NAME`.

## Functions and classes

### `REPLState`

State schema for ``REPLMiddleware``. This class inherits from `AgentState` and is the main object for this part of the module.

### `EvalSchema`

Input schema for the `eval` tool. This class inherits from `BaseModel` and is the main object for this part of the module.

### `REPLMiddleware`

Middleware exposing a persistent JS REPL to the agent. Each LangGraph thread gets its own QuickJS slot (worker + runtime + context), so globals from one conversation cannot leak into another. Args: memory_limit: Bytes the QuickJS heap may use. Shared across all contexts under the same Runtime. Default 64 MiB. timeout: Per-call wall-clock timeout in seconds. Applied to every ``eval`` on every context. Default 5. max_ptc_calls: Maximum number of ``tools.*`` bridge calls allowed during one ``eval`` execution. Exceeding this budget throws from the host-function bridge before invoking the tool. Uncaught overflows surface as ``PTCCallBudgetExceeded``. ``None`` disables the budget (unsafe for untrusted prompts; enables PTC-call DoS patterns). Default 256. !!! warning Setting ``max_ptc_calls=None`` disables the call budget and can allow unbounded PTC host-call loops (DoS risk). Only disable in trusted environments. tool_name: Name of the tool exposed to the model. Default ``eval``. max_result_chars: Result and stdout blocks are independently truncated to this many characters before being sent back to the model. Console buffering is also bounded to this value during collection. Default 4000. capture_console: If ``True``, install a ``console`` object that buffers ``console.log/warn/error`` calls and emits them in ``<stdout>`` blocks alongside the result. Default ``True``. skills_backend: Optional ``BackendProtocol`` the REPL reads skill source files from. When set and a paired ``SkillsMiddleware`` populates ``skills_metadata`` in state, skills with a ``module`` frontmatter key become dynamic- importable from the REPL as ``await import("@/skills/<name>")``. When ``None``, skill modules are not installed (``import(...)`` fails at the resolver). This must be the same backend ``SkillsMiddleware`` uses. ptc: Programmatic tool calling — expose agent tools inside the REPL as ``tools.<camelCase>(input) => Promise<string>``. One ``eval`` call can then orchestrate many tool calls (loops, ``Promise.all``, conditional branching). Accepts: - ``None`` (default) — disabled. - ``list[str | BaseTool]`` — allowlist entries may be: - ``str`` tool names, matched against the agent's toolset. - ``BaseTool`` instances, exposed directly even if not on the agent's tool list. Mixed lists are supported. Explicit ``BaseTool`` entries are considered first; then name-matched agent tools are added. Duplicate names are deduplicated. !!! warning PTC calls currently execute through the REPL bridge and do **not** go through the normal `ToolNode` path. As a result, `interrupt_on` / HITL approval workflows are not enforced per PTC-invoked tool call. The REPL's own tool is always excluded; a model asking for ``tools.eval("...")`` would recurse pointlessly. snapshot_between_turns: If ``True`` (default), persist REPL state across agent turns by creating a snapshot in ``after_agent`` and restoring it in ``before_agent``. If ``False``, preserve the previous behavior where state resets each turn. max_snapshot_bytes: Maximum serialized snapshot payload size allowed in middleware state. If a snapshot exceeds this size, it is dropped (``_quickjs_snapshot_payload=None``). Defaults to ``memory_limit``. Example: ```python from deepagents import create_deep_agent from langchain_quickjs import REPLMiddleware agent = create_deep_agent( model="claude-sonnet-4-6", middleware=[REPLMiddleware()], ) ``` This class inherits from `AgentMiddleware[REPLState, ContextT, ResponseT]` and is the main object for this part of the module.

#### `REPLMiddleware.__init__(self, *, memory_limit: int=_DEFAULT_MEMORY_LIMIT, timeout: float=_DEFAULT_TIMEOUT, max_ptc_calls: int | None=_DEFAULT_MAX_PTC_CALLS, tool_name: str=_DEFAULT_TOOL_NAME, max_result_chars: int=_DEFAULT_MAX_RESULT_CHARS, capture_console: bool=True, ptc: PTCOption | None=None, skills_backend: 'BackendProtocol | None'=None, snapshot_between_turns: bool=True, max_snapshot_bytes: int | None=None)`

Initialize REPL middleware state and build the exposed eval tool. Key arguments are `memory_limit`, `timeout`, `max_ptc_calls`, `tool_name`, `max_result_chars`, `capture_console`, `ptc`. It mutates `self._memory_limit`, `self._timeout`, `self._max_ptc_calls`, `self._tool_name`, `self._max_result_chars`, `self._capture_console`. Internally it delegates to `render_repl_system_prompt`, `ValueError`, `super`, `uuid4`. It runs synchronously in the caller and returns directly.

#### `REPLMiddleware.before_agent(self, state: REPLState, runtime: 'Runtime[ContextT]')`

Restore REPL snapshot bytes into the current thread slot. Key arguments are `state`, `runtime`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get`, `restore_snapshot`, `warning`. It runs synchronously in the caller and returns directly.

#### `REPLMiddleware.abefore_agent(self, state: REPLState, runtime: 'Runtime[ContextT]')`

Async variant of ``before_agent`` snapshot restore. Key arguments are `state`, `runtime`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get`, `arestore_snapshot`, `warning`. This is asynchronous and awaits I/O or framework operations before returning.

#### `REPLMiddleware.wrap_model_call(self, request: ModelRequest[ContextT], handler: Callable[[ModelRequest[ContextT]], ModelResponse[ResponseT]])`

Inject the REPL's system-prompt snippet on every model call. Key arguments are `request`, `handler`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `handler`, `override`. It runs synchronously in the caller and returns directly.

#### `REPLMiddleware.awrap_model_call(self, request: ModelRequest[ContextT], handler: Callable[[ModelRequest[ContextT]], Awaitable[ModelResponse[ResponseT]]])`

(async) Inject the REPL's system-prompt snippet on every model call. Key arguments are `request`, `handler`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `handler`, `override`. This is asynchronous and awaits I/O or framework operations before returning.

#### `REPLMiddleware.after_agent(self, state: REPLState, runtime: 'Runtime[ContextT]')`

Snapshot REPL state (optional) and evict this turn's REPL slot. Key arguments are `state`, `runtime`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get_if_exists`, `evict`, `warning`, `create_snapshot`. It runs synchronously in the caller and returns directly.

#### `REPLMiddleware.aafter_agent(self, state: REPLState, runtime: 'Runtime[ContextT]')`

Async variant of ``after_agent`` snapshot+evict behavior. Key arguments are `state`, `runtime`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get_if_exists`, `aevict`, `warning`, `acreate_snapshot`. This is asynchronous and awaits I/O or framework operations before returning.

## System prompts and tool descriptions

The `EvalSchema.code` field uses this tool-facing description:

```text
JavaScript expression or statement(s) to evaluate. State persists across calls. No fs/network/real-clock access.
```

`_build_tool()` exposes the StructuredTool with this description:

```text
Evaluate JavaScript in a persistent QuickJS REPL. State persists across calls within the same thread.
```

The larger system-prompt snippet is rendered by `_prompt.py:render_repl_system_prompt()` and appended on every model call. When PTC is enabled, `_prepare_for_call()` appends an extra `tools` namespace reference generated from the allowed tool schemas.

## Flow walk-through

1. `__init__()` validates budgets, creates a `_Registry`, renders the base REPL prompt, and builds one LangChain `StructuredTool`.
2. `before_agent()` restores any serialized QuickJS snapshot from private LangGraph state into the thread slot. `after_agent()` snapshots the slot back into `_quickjs_snapshot_payload` and evicts the live runtime.
3. `wrap_model_call()` and `awrap_model_call()` call `_prepare_for_call()`, which installs skill modules and PTC bridge tools for the current thread before extending the system message.
4. The `eval` tool resolves the active `thread_id`, loads any referenced skill modules, evaluates JavaScript through `_Registry`, and formats stdout/result/error into a `ToolMessage`.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
