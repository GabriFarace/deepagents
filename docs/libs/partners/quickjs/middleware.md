# `langchain_quickjs/middleware.py`

## High-Level Purpose

Provides `REPLMiddleware`, an `AgentMiddleware` that injects a persistent JavaScript REPL tool (`eval`) into any Deep Agent. The REPL is backed by `quickjs-rs` (a Rust-native QuickJS binding), runs one `Runtime` per middleware instance, and gives each LangGraph thread its own isolated `Context` for stateful JS execution across turns.

> **Breaking change from older versions:** The class was previously named `QuickJSMiddleware` and used the `quickjs` Python package. It is now `REPLMiddleware` backed by `quickjs-rs`. The tool is now named `"eval"` (not `"repl"`). Skill module imports and programmatic tool calling (PTC) are new capabilities.

## Module-Level Constants

| Constant | Description |
|---|---|
| `REPL_TOOL_DESCRIPTION` | Short description for the `eval` tool |
| `REPL_SYSTEM_PROMPT` | Detailed prompt fragment with usage instructions and placeholders for PTC docs and skill modules |

## Type Aliases

### `PTCOption`
`list[str | BaseTool] | None` — Programmatic tool calling configuration passed to `REPLMiddleware`.
- `None` (default): PTC disabled.
- `list[str | BaseTool]`: Allowlist entries. Tool names as strings or `BaseTool` instances.

## Class: `REPLMiddleware`

**Purpose:** An `AgentMiddleware` that adds a persistent JS `eval` tool to the agent, injects usage instructions into the system prompt, and optionally supports skill module imports and programmatic tool calling.

**Inherits from:** `AgentMiddleware[AgentState[Any], ContextT, ResponseT]`

### `__init__`

```python
REPLMiddleware(
    *,
    memory_limit: int = 67108864,
    timeout: float = 5.0,
    max_ptc_calls: int | None = 256,
    tool_name: str = "eval",
    max_result_chars: int = 4000,
    capture_console: bool = True,
    skills_backend: BackendProtocol | None = None,
    ptc: PTCOption = None,
) -> None
```

**Parameters:**

| Parameter | Default | Description |
|---|---|---|
| `memory_limit` | `67108864` (64 MB) | QuickJS heap limit in bytes, shared across all contexts for this runtime |
| `timeout` | `5.0` | Per-call wall-clock timeout in seconds |
| `max_ptc_calls` | `256` | Maximum `tools.*` bridge calls allowed per `eval` invocation. `None` disables the budget (DoS risk for untrusted prompts). Overflow raises `PTCCallBudgetExceeded`. |
| `tool_name` | `"eval"` | Name of the exposed tool |
| `max_result_chars` | `4000` | Truncation limit for result and stdout blocks |
| `capture_console` | `True` | Install a `console` object that buffers `console.log`/`warn`/`error` output |
| `skills_backend` | `None` | Backend for reading skill source files. When set, skills with a `module` field become dynamically importable as `await import("@/skills/<name>")` from JS |
| `ptc` | `None` | Programmatic tool calling allowlist (see `PTCOption`) |

### Key Design Points

**Persistent REPL per thread:**
- One `quickjs_rs.Runtime` is created per `REPLMiddleware` instance (shared across all LangGraph threads).
- Each LangGraph `thread_id` gets its own `quickjs_rs.Context` — variables, functions, and state persist across `eval` calls within the same thread.
- Contexts are isolated: no global leakage between threads.

**Programmatic Tool Calling (PTC):**
- Tools in the PTC allowlist become callable from JS as `tools.<camelCase>(input)` returning a `Promise<string>`.
- PTC calls do **not** go through `ToolNode` and do **not** enforce `interrupt_on` / HITL workflows. Use with trusted prompts only.
- `max_ptc_calls` budget guards against runaway loops. Default is 256 calls per `eval`.

**Skill Module Imports:**
- Requires `skills_backend` **and** a paired `SkillsMiddleware` that populates `skills_metadata` in state.
- Skills with a `module` field (e.g., `module: index.ts`) in their `SKILL.md` frontmatter are installed as dynamic ES modules importable via `await import("@/skills/<name>")`.
- The module source is loaded from `skills_backend` at import time.

### Key Methods

#### `modify_request(request: ModelRequest) -> ModelRequest`
Appends the REPL system prompt fragment (including optional PTC docs and skill module listing) to the existing system message.

#### `wrap_model_call(request, handler) -> ModelResponse`
Synchronous wrapper: modifies the request before passing it to `handler`.

#### `awrap_model_call(request, handler) -> ModelResponse`
Async wrapper: same as above, but awaits the handler.

## `eval` Tool Schema

| Field | Type | Description |
|---|---|---|
| `code` | `str` | JavaScript expression or statement(s) to execute |

**Returns:** Captured `console.log` output (if any), or the final expression value as a string. On error, returns the JS exception string.

**State:** Persistent across calls within the same LangGraph thread. Variables, functions, and imports defined in one call are available in subsequent calls.

**Limitations:**
- No filesystem, network, or real-clock access by default.
- Top-level `await` is supported (skill modules and PTC are async).

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `quickjs_rs` | `quickjs-rs` | Rust-native embedded JS engine (replaces `quickjs`) |
| `AgentMiddleware`, `AgentState`, `ModelRequest`, `ModelResponse` | `langchain.agents.middleware.types` | Middleware protocol |
| `StructuredTool` | `langchain_core.tools` | Tool creation |
| `append_to_system_message` | `deepagents.middleware._utils` | System prompt modification |
| `deepagents_quickjs._ptc` | internal | PTC bridge and budget enforcement |
| `deepagents_quickjs._skills` | internal | Skill module loader |
| `deepagents_quickjs._prompt` | internal | System prompt builder |
| `deepagents_quickjs._repl` | internal | Per-thread context management |
| `deepagents_quickjs._format` | internal | Result formatting and truncation |
