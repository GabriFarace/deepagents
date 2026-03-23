# `langchain_quickjs/middleware.py`

## High-Level Purpose

Provides `QuickJSMiddleware`, an `AgentMiddleware` that injects a JavaScript REPL tool into any Deep Agent. The REPL executes code using the embedded QuickJS JavaScript engine, supports Python foreign functions accessible from JS, and configures the system prompt with usage instructions.

## Module-Level Constants

| Constant | Description |
|----------|-------------|
| `REPL_TOOL_DESCRIPTION` | Short description for the `repl` tool: `"Evaluates code using a QuickJS-backed JavaScript REPL."` |
| `REPL_SYSTEM_PROMPT` | Detailed prompt fragment instructing the model how to use the REPL, including statelessness warning and `{external_functions_section}` placeholder |

## Classes

### `QuickJSMiddleware`

**Purpose:** An `AgentMiddleware` that adds a `repl` tool backed by QuickJS to the agent's tool set, and injects usage instructions into the system prompt.

**Inherits from:** `AgentMiddleware[AgentState[Any], ContextT, ResponseT]`

#### `__init__`

```python
def __init__(
    self,
    *,
    ptc: list[Callable | BaseTool] | None = None,
    add_ptc_docs: bool = False,
    timeout: int | None = None,
    memory_limit: int | None = None,
) -> None
```

**Parameters:**
- `ptc`: Python functions or LangChain tools to expose inside the QuickJS REPL as foreign functions (PTC = "pass-through callables").
- `add_ptc_docs`: If `True`, includes TypeScript-like signatures and docstrings for the foreign functions in the system prompt.
- `timeout`: Optional global timeout in seconds for each JS evaluation.
- `memory_limit`: Optional memory limit in bytes for each evaluation.

**Key Setup:** Calls `_create_repl_tool()` and stores it in `self.tools`.

#### Key Methods

##### `modify_request(request: ModelRequest) -> ModelRequest`
Appends the REPL system prompt fragment (including optional foreign function docs) to the existing system message.

##### `wrap_model_call(request, handler) -> ModelResponse`
Synchronous wrapper: modifies the request before passing it to `handler`.

##### `awrap_model_call(request, handler) -> ModelResponse`
Async wrapper: same as above, but awaits the handler.

##### `_format_repl_system_prompt() -> str`
Builds the system prompt fragment by calling `render_external_functions_section()` for the configured PTC implementations.

##### `_create_context(timeout, print_callback, *, prefer_async, runtime) -> quickjs.Context`
Creates a fresh QuickJS context for a single evaluation. Sets the time limit and memory limit if configured, installs a `print` callback, and registers all foreign functions and their JavaScript shims.

##### `_evaluate(code: str, *, timeout, prefer_async, runtime) -> str`

**Purpose:** Execute JavaScript code and return the captured output.

**Return Value:**
- If `print()` was called: joined printed lines.
- If no output but a final expression value: `str(value)`.
- If `JSException` raised: the exception string.
- If context creation fails: `"Error: ..."`.

##### `_create_repl_tool() -> BaseTool`
Creates a `StructuredTool` named `"repl"` with both a sync (`_sync_quickjs`) and async (`_async_quickjs`) implementation. Both accept `code: str` and `timeout: int | None`, and call `_evaluate()`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `quickjs` | `quickjs` | Embedded JS engine |
| `AgentMiddleware`, `AgentState`, `ModelRequest`, `ModelResponse` | `langchain.agents.middleware.types` | Middleware protocol |
| `StructuredTool` | `langchain_core.tools` | Tool creation |
| `append_to_system_message` | `deepagents.middleware._utils` | System prompt modification utility |
| `render_external_functions_section` | `langchain_quickjs._foreign_function_docs` | Prompt generation for foreign functions |
| `get_ptc_implementations`, `install_external_functions` | `langchain_quickjs._foreign_functions` | Foreign function management |
