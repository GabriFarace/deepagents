# `repl/langchain_repl/middleware.py`

> Agent middleware that exposes the custom imperative REPL as a `repl` tool and injects prompt guidance.

## Position in the system

This package provides an optional REPL tool middleware for LangChain/deepagents. It is independent of the main SDK and can be mounted as middleware when an agent needs compact programmable tool orchestration.

## Imports and module-level state

This file imports `__future__, typing, deepagents.middleware._utils, langchain.agents.middleware.types, langchain.tools, langchain_core.tools, langchain_core.tools.base, langchain_repl._foreign_function_docs, langchain_repl.interpreter`.
Module constants worth noticing: `REPL_TOOL_DESCRIPTION`, `REPL_SYSTEM_PROMPT`.

## Functions and classes

### `ReplMiddleware`

Provide a REPL-backed `repl` tool to an agent. This class inherits from `AgentMiddleware[AgentState[Any], ContextT, ResponseT]` and is the main object for this part of the module.

#### `ReplMiddleware.__init__(self, *, ptc: list[Callable[..., Any] | BaseTool] | None=None, add_ptc_docs: bool=False, max_concurrency: int | None=None)`

Initialize the middleware and register the `repl` tool. Key arguments are `ptc`, `add_ptc_docs`, `max_concurrency`. It mutates `self._ptc`, `self._add_ptc_docs`, `self._max_concurrency`, `self.tools`. It runs synchronously in the caller and returns directly.

#### `ReplMiddleware.modify_request(self, request: ModelRequest[ContextT])`

Append the REPL prompt guidance to the request system message. Key arguments are `request`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `append_to_system_message`, `override`. It runs synchronously in the caller and returns directly.

#### `ReplMiddleware.wrap_model_call(self, request: ModelRequest[ContextT], handler: Callable[[ModelRequest[ContextT]], ModelResponse[ResponseT]])`

Apply request modifications before a synchronous model call. Key arguments are `request`, `handler`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `modify_request`, `handler`. It runs synchronously in the caller and returns directly.

#### `ReplMiddleware.awrap_model_call(self, request: ModelRequest[ContextT], handler: Callable[[ModelRequest[ContextT]], Awaitable[ModelResponse[ResponseT]]])`

Apply request modifications before an asynchronous model call. Key arguments are `request`, `handler`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `modify_request`, `handler`. This is asynchronous and awaits I/O or framework operations before returning.

## System prompts and tool descriptions

### `REPL_TOOL_DESCRIPTION`

```text
Evaluates code using a small imperative REPL.

CRITICAL: The REPL does NOT retain state between calls. Each `repl` invocation is evaluated from scratch.
Do NOT assume variables, functions, or helper values from prior `repl` calls are available.

Capabilities and limitations:
- The language supports assignment, `if ... then ... else ... end`, `for ... in ... do ... end`, indexing, function calls, and `parallel(...)`.
- Use `print(value)` to emit output. The tool returns printed lines joined with newlines.
- The final expression value is returned only if nothing was printed.
- Values include strings, `None`, `True`, `False`, integers, floats, lists, and dicts.
- `parallel([defer(call1(...)), defer(call2(...))])` evaluates independent callable invocations concurrently using isolated snapshots of the current bindings.
- There is no filesystem or network access unless you expose Python callables as foreign functions.
{external_functions_section}
```

### `REPL_SYSTEM_PROMPT`

```text
## REPL tool

You have access to a `repl` tool.

CRITICAL: The REPL does NOT retain state between calls. Each `repl` invocation is evaluated from scratch.
Do NOT assume variables, functions, or helper values from prior `repl` calls are available.

- The REPL executes a small imperative language.
- Write assignments like `user = lookup_fn("value")`.
- Use indexing like `items[0]` and `user["id"]`.
- Use `if cond then ... else ... end` for branching.
- Use `for item in items do ... end` for loops.
- Use `print(value)` to emit output. The tool returns printed lines joined with newlines.
- The final expression value is returned only if nothing was printed.
- Use `parallel([defer(call1(...)), defer(call2(...))])` only for independent callable invocations that can run concurrently.
- The REPL can only use the language features above and the foreign functions listed below.
- If the task needs multiple foreign function calls, prefer writing one complete REPL program instead of splitting the work across multiple `repl` invocations.
- When writing REPL scripts, always pipeline dependent lookups within a single call when possible.
- If a result from one foreign function is needed as input to later foreign function calls, write one REPL program that performs the full sequence of dependent calls instead of returning intermediate results to the model between steps.
- Only split work across multiple `repl` invocations when you genuinely cannot determine what to do next without additional model reasoning or user input.
- If one foreign function returns an ID or other value that can be passed directly into the next foreign function, trust it and chain the calls instead of stopping to double-check it.
- If you want to inspect an intermediate value, print it inside the same REPL program; otherwise, try to fetch as much information as possible in one program.
- Example syntax only - this shows the language shape, not specific available foreign functions:
  `items = lookup_fn("value")`
  `first_item = items[0]`
  `item_id = first_item["id"]`
  `print(parallel([defer(detail_fn(item_id)), defer(status_fn(item_id))]))`
- Use the repl for small computations, collection manipulation, branching, loops, and calling externally registered foreign functions.
{external_functions_section}
```

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
