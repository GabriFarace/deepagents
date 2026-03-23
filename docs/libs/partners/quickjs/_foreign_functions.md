# `langchain_quickjs/_foreign_functions.py`

## High-Level Purpose

Bridges Python callables and LangChain tools into QuickJS JavaScript contexts. Handles:
- Normalizing Python functions and LangChain tools into a common callable form for QuickJS registration.
- Serializing complex Python return values (lists, dicts) to JSON so QuickJS can receive them as JavaScript objects/arrays.
- Installing JavaScript shim functions that transparently parse JSON-encoded return values.
- Routing async callables to a background event loop when called from synchronous QuickJS code.

## Classes

### `_AsyncLoopThread`

**Purpose:** Maintains a dedicated daemon-thread event loop for bridging async Python callables into synchronous QuickJS execution.

**Attributes:**
- `_loop`: The background `asyncio.AbstractEventLoop`.
- `_thread`: The daemon thread running the loop.

**Methods:**

##### `submit(coroutine: Coroutine) -> Future`
Schedules a coroutine on the background event loop using `asyncio.run_coroutine_threadsafe()`.

## Module-Level Singleton

`_ASYNC_LOOP_THREAD = _AsyncLoopThread()` — Created at import time. Shared across all `QuickJSMiddleware` instances.

## Functions

### `get_ptc_implementations(ptc: list | None) -> dict[str, Callable | BaseTool]`

**Purpose:** Normalize the `ptc` list into a name-keyed dict of implementations.

- `BaseTool` instances are keyed by `tool.name`.
- Plain callables are keyed by `.__name__`.

---

### `install_external_functions(context: quickjs.Context, implementations, *, execution_mode, runtime) -> None`

**Purpose:** Register all foreign functions and their JavaScript shims in a QuickJS context.

**Parameters:**
- `context`: The QuickJS context to install into.
- `implementations`: Name-keyed dict of callables/tools.
- `execution_mode`: `"sync"` or `"async"` — controls whether tools prefer async invocation.
- `runtime`: `ToolRuntime` for injected-argument tools.

**Key Logic:**
1. Builds wrapped callables using `_build_external_functions()`.
2. Registers each as `__python_<name>` in the context.
3. Calls `inject_external_function_shims()` to add the JavaScript bridge.

---

### `inject_external_function_shims(context: quickjs.Context, external_functions: list[str] | None) -> None`

**Purpose:** Install JavaScript shim functions that parse JSON-encoded return values from Python.

Each shim is a JavaScript arrow function that:
1. Calls the underlying `__python_<name>` callable.
2. If the result is a string starting with `[` or `{`, JSON-parses it.
3. Otherwise returns the value as-is.

This transparently gives JavaScript code native arrays and objects from Python functions that return lists or dicts.

---

### `_build_external_functions(implementations, *, prefer_async, runtime) -> dict[str, Callable]`

Converts each implementation to a QuickJS-registerable callable:
- `BaseTool` → wrapped via `_wrap_tool_for_js()`
- Plain callable → wrapped via `_wrap_function_for_js()`
- All registered under the `__python_<name>` key.

---

### `_wrap_tool_for_js(tool, *, prefer_async, runtime) -> Callable`

Creates a plain sync callable that: builds the tool payload from positional/keyword args using `_build_tool_payload()`, then invokes the tool via `_invoke_tool()`.

---

### `_wrap_function_for_js(implementation: Callable) -> Callable`

Wraps a Python callable to: await coroutines using `_await_if_needed()`, then JSON-encode complex return values using `_serialize_for_js()`.

---

### `_invoke_tool(tool, payload, *, prefer_async) -> Any`

Invokes a LangChain tool via its sync or async entrypoint. Resolves awaitables using `_ASYNC_LOOP_THREAD`.

---

### `_build_tool_payload(tool, args, kwargs, *, runtime) -> str | dict`

Maps positional/keyword JavaScript call arguments to a LangChain tool input payload, accounting for the tool's input schema field names and injected runtime arguments.

---

### `_serialize_for_js(value: Any) -> Any`

Returns primitives unchanged; JSON-encodes everything else as a string.

---

### `_await_if_needed(value: Any) -> Any`

If the value is awaitable, submits it to `_ASYNC_LOOP_THREAD` and blocks until done; otherwise returns it directly.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `quickjs` | `quickjs` | JS context and callable registration |
| `BaseTool` | `langchain_core.tools` | LangChain tool type |
| `_is_injected_arg_type`, `get_all_basemodel_annotations` | `langchain_core.tools.base` | Introspecting injected parameters |
| `asyncio`, `threading`, `json`, `inspect` | stdlib | Async bridge and introspection |
