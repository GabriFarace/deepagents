# `langchain_quickjs/_ptc.py` (formerly `_foreign_functions.py`)

> **Note:** This file was renamed and restructured in v0.1.0. The old `_foreign_functions.py` bridged Python callables into the `quickjs` (Python) engine. The new `_ptc.py` bridges agent tools into `quickjs-rs` (Rust) contexts with budget enforcement. See also `_repl.py` for context management and `_skills.py` for skill module loading.

## High-Level Purpose

Implements **Programmatic Tool Calling (PTC)** — the bridge that makes agent tools callable directly from JavaScript as `tools.<camelCase>(input)` returning a `Promise<string>`.

Key responsibilities:
- Installing the `tools` object into a `quickjs_rs.Context`
- Routing JS `tools.*` calls to LangChain tool invocations
- Enforcing the `max_ptc_calls` budget per `eval` invocation
- Raising `PTCCallBudgetExceeded` when the budget is exhausted

## Classes

### `PTCCallBudgetExceeded`

**Type:** `Exception`

Raised when the number of `tools.*` calls in a single `eval` invocation exceeds `max_ptc_calls`. Surfaces to the agent as an error string from the `eval` tool.

### `PTCBridge`

Manages the PTC installation for a single `quickjs_rs.Context`.

**Constructor:**
```python
PTCBridge(
    context: quickjs_rs.Context,
    tools: dict[str, BaseTool],
    *,
    max_ptc_calls: int | None,
    runtime: ToolRuntime,
    prefer_async: bool,
)
```

**Methods:**

#### `install() -> None`
Installs the `tools` object into the context. Each allowed tool becomes accessible as `tools.<camelCase>(input)` returning `Promise<string>`.

#### `reset_budget() -> None`
Resets the call counter to zero. Called at the start of each `eval` invocation.

## Functions

### `build_ptc_tools(ptc: PTCOption, all_tools: list[BaseTool]) -> dict[str, BaseTool]`

Resolves `PTCOption` entries to actual `BaseTool` objects from the agent's tool list.

- String entries are matched by tool name.
- `BaseTool` instances are used directly.
- Unresolved names raise `ValueError`.

### `to_camel_case(name: str) -> str`

Converts a snake_case tool name to camelCase for the JS `tools.*` API.

Example: `"read_file"` → `"readFile"`, `"grep"` → `"grep"`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `quickjs_rs` | `quickjs-rs` | JS context and value types |
| `BaseTool` | `langchain_core.tools` | LangChain tool type |
| `asyncio`, `threading` | stdlib | Async bridge for sync JS context |
