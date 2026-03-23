# `deepagents/middleware/filesystem.py`

## High-Level Purpose

`FilesystemMiddleware` is the core middleware that provides file system tools to agents: `ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`, and `execute`. It also handles:
- Dynamic tool availability (removes `execute` if the backend doesn't support it)
- Large tool result eviction (offloads oversized results to the filesystem rather than returning them inline)
- Filesystem state management via the `FilesystemState` schema

## Key Constants

| Constant | Value | Purpose |
|---|---|---|
| `GLOB_TIMEOUT` | `20.0` | Seconds before async glob is cancelled |
| `DEFAULT_READ_OFFSET` | `0` | Default starting line for read_file |
| `DEFAULT_READ_LIMIT` | `100` | Default max lines for read_file |
| `NUM_CHARS_PER_TOKEN` | `4` | Approximate characters per token for truncation |
| `TOOLS_EXCLUDED_FROM_EVICTION` | tuple | Tools that skip the large-result eviction logic |

`TOOLS_EXCLUDED_FROM_EVICTION` = `("ls", "glob", "grep", "read_file", "edit_file", "write_file")` — these tools either have built-in truncation or are too small to ever exceed limits.

## Important Types

### `FilesystemState(AgentState)`
LangGraph state schema extension for filesystem middleware.

**Fields:**
- `files: Annotated[NotRequired[dict[str, FileData]], _file_data_reducer]` — In-memory file dictionary. Uses the custom `_file_data_reducer` which supports file deletion via `None` values.

### `_file_data_reducer(left, right) -> dict`
Custom LangGraph state reducer for the `files` field. Supports deletion: if a key in `right` maps to `None`, the corresponding key is removed from the merged result.

## Tool Description Constants

Large docstrings defining the agent-facing descriptions for each tool:
- `LIST_FILES_TOOL_DESCRIPTION` — `ls`
- `READ_FILE_TOOL_DESCRIPTION` — `read_file` (includes pagination guidance, image handling instructions)
- `EDIT_FILE_TOOL_DESCRIPTION` — `edit_file`
- `WRITE_FILE_TOOL_DESCRIPTION` — `write_file`
- `GLOB_TOOL_DESCRIPTION` — `glob`
- `GREP_TOOL_DESCRIPTION` — `grep`
- `EXECUTE_TOOL_DESCRIPTION` — `execute`
- `FILESYSTEM_SYSTEM_PROMPT` — Injected into the system message; explains filesystem tools
- `EXECUTION_SYSTEM_PROMPT` — Injected when execution is available

## Key Helper Functions

### `_supports_execution(backend) -> bool`
Checks if a backend supports `execute()`. For `CompositeBackend`, inspects the `default` sub-backend.

### `_create_content_preview(content_str, *, head_lines=5, tail_lines=5) -> str`
Creates a formatted preview of large content showing the first and last few lines with a truncation marker.

### `_extract_text_from_message(message: ToolMessage) -> str`
Extracts the text content from a tool message for eviction analysis.

## Class: `FilesystemMiddleware(AgentMiddleware[FilesystemState, ContextT, ResponseT])`

The primary middleware class. Wraps every LLM call to inject filesystem tools and handle results.

### Constructor

```python
FilesystemMiddleware(backend: BackendProtocol | BackendFactory)
```

**Parameters:**
- `backend` — Backend instance or factory function. A factory (`lambda rt: StateBackend(rt)`) is resolved at tool-call time against the `ToolRuntime`.

### State Schema
`state_schema = FilesystemState`

### Key Methods

#### `wrap_model_call(request, handler) -> ModelResponse`
The core interception point. Before forwarding to the LLM:
1. Conditionally adds `EXECUTION_SYSTEM_PROMPT` to the system message if the backend supports execution.
2. Filters out the `execute` tool if execution is not supported.

#### `awrap_model_call(request, handler) -> ModelResponse`
Async version of `wrap_model_call`.

#### Tool Implementations

All tools are defined as `StructuredTool` instances returned by internal builder methods. Each tool:
- Receives a `ToolRuntime` parameter injected by LangGraph
- Resolves the backend from `self._backend` (instance or factory)
- Calls the appropriate backend method
- Handles errors and formats results

**`ls` tool:** Lists directory contents. Formats `FileInfo` entries as a table. Truncates if too many results.

**`read_file` tool:** Reads a file with optional offset/limit. Adds line numbers via `format_content_with_line_numbers`. Handles images as multimodal content blocks. Returns truncation message for very long lines.

**`write_file` tool:** Creates a new file. Returns `Command(update={"files": ...})` for checkpoint backends (merges `files_update` into LangGraph state). Returns plain string for external backends.

**`edit_file` tool:** Replaces a string in an existing file. Requires the file to have been read first (validates via conversation history). Returns `Command` or string similar to `write_file`.

**`glob` tool:** Finds files matching a pattern. Has an async timeout of `GLOB_TIMEOUT` seconds.

**`grep` tool:** Searches file contents for a literal text pattern. Supports `output_mode` parameter (`"files_with_matches"`, `"content"`, `"count"`).

**`execute` tool:** Runs a shell command via the backend's `execute()`. Forwards `timeout` if the backend supports it. Returns error if execution is not supported.

#### Large Result Eviction

After tool execution, `FilesystemMiddleware` inspects the result's token count. If a tool result exceeds the threshold and is not in `TOOLS_EXCLUDED_FROM_EVICTION`, the result is:
1. Written to a file under `/large_tool_results/{sanitized_tool_call_id}`
2. Replaced in the message with `TOO_LARGE_TOOL_MSG` (which includes a content preview and instructions to use `read_file`)

## Dependencies

- `langchain.agents.middleware.types` — `AgentMiddleware`, `AgentState`, etc.
- `deepagents.backends.*` — all backend types
- `deepagents.backends.utils` — formatting helpers
- `deepagents.middleware._utils.append_to_system_message`
- `langchain_core.messages`, `langchain_core.tools`
- `langgraph.types.Command`
