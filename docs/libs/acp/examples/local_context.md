# `acp/examples/local_context.py`

> Example middleware that probes a local project and appends compact environment context to an agent system prompt.

## Position in the system

This package sits at the boundary between deepagents and ACP clients. It imports the SDK graph/backends and ACP schema objects, then translates protocol calls into LangGraph invocations and session updates.

## Imports and module-level state

This file imports `__future__, logging, typing, langchain.agents.middleware.types`.
Module constants worth noticing: `_TOOL_NAME_DISPLAY_LIMIT`, `DETECT_CONTEXT_SCRIPT`.

## Functions and classes

### `build_detect_script()`

Concatenate all section functions into the full detection script. Independent sections run as parallel background jobs writing to temp files, then results are concatenated in the original display order. The header (CWD / IN_GIT) and project section (sets ROOT) run first because later sections depend on their variables. Returns: Complete bash heredoc ready for `backend.execute()`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `join`. It runs synchronously in the caller and returns directly.

### `LocalContextState`

State for local context middleware. This class inherits from `AgentState` and is the main object for this part of the module.

### `LocalContextMiddleware`

Inject local context (git state, project structure, etc.) into the system prompt. Runs a bash detection script via `backend.execute()` on first interaction and again after each summarization event, stores the result in state, and appends it to the system prompt on every model call. Because the script runs inside the backend, it works for both local shells and remote sandboxes. This class inherits from `AgentMiddleware` and is the main object for this part of the module.

#### `LocalContextMiddleware.__init__(self, backend: _ExecutableBackend)`

Initialize with a backend that supports shell execution. Args: backend: Backend instance that provides shell command execution. Key arguments are `backend`. It mutates `self.backend`. It runs synchronously in the caller and returns directly.

#### `LocalContextMiddleware.before_agent(self, state: LocalContextState, runtime: Runtime)`

Run context detection on first interaction and refresh after summarization. On the first invocation, runs the detection script and stores the result. After a summarization event (indicated by a new `_summarization_event` in state), re-runs the script to capture any environment changes that occurred during the session. Args: state: Current agent state. runtime: Runtime context. Returns: State update with `local_context` populated on success. On a post-summarization refresh failure, returns a state update recording the cutoff (without `local_context`) to prevent retry loops. Returns `None` if context is already set and no refresh is needed, or if initial detection fails. Key arguments are `state`, `runtime`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get`. It runs synchronously in the caller and returns directly.

#### `LocalContextMiddleware.wrap_model_call(self, request: ModelRequest, handler: Callable[[ModelRequest], ModelResponse])`

Inject local context into system prompt. Args: request: The model request being processed. handler: The handler function to call with the modified request. Returns: The model response from the handler. Key arguments are `request`, `handler`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `handler`. It runs synchronously in the caller and returns directly.

#### `LocalContextMiddleware.awrap_model_call(self, request: ModelRequest, handler: Callable[[ModelRequest], Awaitable[ModelResponse]])`

Inject local context into system prompt (async). Args: request: The model request being processed. handler: The async handler function to call with the modified request. Returns: The model response from the handler. Key arguments are `request`, `handler`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `handler`. This is asynchronous and awaits I/O or framework operations before returning.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
