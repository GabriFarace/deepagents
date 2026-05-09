# `libs/cli/deepagents_cli/local_context.py`

> Middleware for injecting local context into system prompt.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from langchain.agents.middleware.types import AgentMiddleware, AgentState, ModelRequest, ModelResponse, PrivateStateAttr`


## Functions and classes

### `_build_mcp_context(servers: list[MCPServerInfo])`

Format MCP server/tool inventory for the system prompt.

Additional notes from the source docstring:

```text
Args:
    servers: List of connected MCP server metadata.

Returns:
    Formatted markdown string, or `""` if no servers.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_ExecutableBackend`

Any backend that supports `execute(command) -> ExecuteResponse`.

Methods worth reading inside this class:

- `execute(self, command: str, *, timeout: int | None=None)`: This command runner bridges parsed CLI options to the underlying helper functions and performs the user-visible side effects.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_AsyncExecutableBackend`

Any backend that provides an async `aexecute` method.

Methods worth reading inside this class:

- `aexecute(self, command: str, *, timeout: int | None=None)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_section_header()`

CWD line and IN_GIT flag (used by other sections).

Additional notes from the source docstring:

```text
Returns:
    Bash snippet that prints the header and sets `CWD` / `IN_GIT`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_project()`

Language, monorepo, git root, virtual-env detection.

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (requires `CWD` / `IN_GIT` from header).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_package_managers()`

Python and Node package manager detection.

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (standalone).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_runtimes()`

Python and Node runtime version detection.

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (standalone).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_git()`

Git branch or detached HEAD commit, main branches, uncommitted changes.

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (requires `IN_GIT` from header).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_test_command()`

Test command detection (make test / pytest / npm test).

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (standalone).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_files()`

Directory listing (filtered, capped at 20).

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (standalone).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_tree()`

`tree -L 3` output.

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (standalone).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_section_makefile()`

First 20 lines of Makefile (falls back to git root in monorepos).

Additional notes from the source docstring:

```text
Returns:
    Bash snippet (requires `ROOT` from `_section_project` and `CWD` from header).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `build_detect_script()`

Concatenate all section functions into the full detection script.

Additional notes from the source docstring:

```text
Independent sections run as parallel background jobs writing to temp
files, then results are concatenated in the original display order.
The header (CWD / IN_GIT) and project section (sets ROOT) run first
because later sections depend on their variables.

Returns:
    Complete bash heredoc ready for `backend.execute()`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `LocalContextState`

State for local context middleware.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `LocalContextMiddleware`

Inject local context (git state, project structure, etc.) into the system prompt.

Additional notes from the source docstring:

```text
Runs a bash detection script via `backend.execute()` on first interaction
and again after each summarization event, stores the result in state, and
appends it to the system prompt on every model call.

Because the script runs inside the backend, it works for both local shells
and remote sandboxes.
```

Methods worth reading inside this class:

- `_handle_detect_result(result: ExecuteResponse)`: Validate detection script output and normalize it for state storage.

- `_run_detect_script(self)`: Run the environment detection script.

- `before_agent(self, state: LocalContextState, runtime: Runtime)`: Run context detection on first interaction and refresh after summarization.

- `_arun_detect_script(self)`: Run the environment detection script asynchronously.

- `abefore_agent(self, state: LocalContextState, runtime: Runtime)`: Async variant of `before_agent` for use in async execution contexts.

- `_get_modified_request(self, request: ModelRequest)`: Append local context and MCP info to the system prompt if available.

- `wrap_model_call(self, request: ModelRequest, handler: Callable[[ModelRequest], ModelResponse])`: Inject local context into system prompt.

- `awrap_model_call(self, request: ModelRequest, handler: Callable[[ModelRequest], Awaitable[ModelResponse]])`: Inject local context into system prompt (async).


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
