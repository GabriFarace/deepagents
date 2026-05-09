# `libs/deepagents/deepagents/middleware/filesystem.py`

> Filesystem middleware. It exposes `ls`, `read_file`, `write_file`,
> `edit_file`, `glob`, `grep`, and optionally `execute`, all backed by
> `BackendProtocol`.

## Position in the system

`create_deep_agent()` installs `FilesystemMiddleware` in the main agent and in
declarative subagents. It is the bridge between model-facing tools and backend
storage/execution.

```
model tool call
  │
  ▼
FilesystemMiddleware tool wrapper
  ├─ validate absolute path
  ├─ enforce filesystem permissions
  ├─ call BackendProtocol / SandboxBackendProtocol
  ├─ format ToolMessage
  └─ optionally evict large content to backend files
```

The middleware also mutates model requests through `wrap_model_call()`: it
injects filesystem instructions, hides `execute` when the backend cannot run
commands, and replaces oversized human messages with short previews while
keeping the full content in backend storage.

## Imports and module-level state

The module imports `wcmatch.glob` for permission pattern matching,
LangChain/LangGraph middleware types for model/tool wrappers, and backend
result types from `deepagents.backends.protocol`. It re-exports
`BACKEND_TYPES` and `FileData` for backward compatibility.

Important constants include glob flags, default read pagination, the
approximate `NUM_CHARS_PER_TOKEN` conversion, tool descriptions, prompt
templates, and `TOOLS_EXCLUDED_FROM_EVICTION`. The eviction exclusions are
intentional: search/list/read tools already truncate or have tricky reread
semantics, and write/edit tools normally return tiny confirmations.

## Functions and classes

### `FilesystemPermission`

`FilesystemPermission` is a dataclass access rule with `operations`, `paths`,
and `mode`. `operations` is a list of `"read"` and/or `"write"`; `paths` are
absolute POSIX-style glob patterns; `mode` is `"allow"` or `"deny"` and
defaults to allow.

`__post_init__()` validates paths eagerly. Patterns must start with `/`, cannot
contain `..`, and currently cannot contain `~`. That prevents common path
escape or home-expansion ambiguities before any tool call reaches a backend.

### Permission helper functions

`_check_fs_permission()` walks rules in declaration order and returns the first
matching mode for the requested operation/path. If no rule matches, access is
allowed. `_filter_paths_by_permission()`, `_filter_file_infos_by_permission()`,
`_filter_grep_matches_by_permission()`, `_apply_permissions_to_ls_results()`,
and `_apply_permissions_to_glob_results()` apply that same rule engine to
lists returned by backend searches.

`_all_paths_scoped_to_routes()` is a safety guard for `CompositeBackend` with
execution support. Filesystem permissions do not yet constrain shell commands,
so permissions are allowed with executable composite backends only when every
permission path is scoped to a configured route rather than the executable
default backend.

### `_file_data_reducer(left, right)`

Reducer for the `files` state channel. It merges right-hand file updates into
existing state and treats `None` values as deletion markers. This lets state
backends remove files through normal LangGraph reducer semantics.

### `FilesystemState`

Extends `AgentState` with an optional `files` channel annotated with
`_file_data_reducer`. `StateBackend` relies on this channel for the virtual
filesystem; disk and sandbox backends may ignore it.

### Tool schema classes

`LsSchema`, `ReadFileSchema`, `WriteFileSchema`, `EditFileSchema`,
`GlobSchema`, `GrepSchema`, and `ExecuteSchema` are Pydantic schemas used by
`StructuredTool.from_function()`. They make the model-facing tool arguments
explicit and carry argument descriptions into the tool schema.

`ReadFileSchema` defaults to offset `0` and limit `100`, enforcing paginated
reads by default. `ExecuteSchema` includes optional `timeout`, which is later
bounded by `max_execute_timeout` and checked against backend support.

### `supports_execution(backend)`

Returns whether a backend can run commands. For a plain backend it checks
`isinstance(backend, SandboxBackendProtocol)`. For a `CompositeBackend`, it
checks the default backend because `execute` is not path-routed like file
operations.

This helper is used in both constructor validation and `wrap_model_call()` so
the `execute` tool appears only when it can succeed.

### Human-message and tool-result eviction helpers

`_extract_text_from_message()` joins text content blocks and ignores media.
`_create_content_preview()` renders a numbered head/tail preview with a
truncation marker. `_build_evicted_human_content()` and
`_build_evicted_content()` replace text blocks with an eviction notice while
preserving non-text blocks such as images.

`_build_truncated_human_message()` creates the lightweight message sent to the
model after a human message has been evicted. Full content remains in the
conversation state and in the backend path recorded in `additional_kwargs`.

### `FilesystemMiddleware`

`FilesystemMiddleware` declares `state_schema = FilesystemState`, builds the
file tools, injects system instructions, filters `execute` by backend
capability, and evicts oversized content.

The constructor accepts a backend instance or deprecated backend factory,
optional system prompt override, optional custom tool descriptions, token
limits for tool and human-message eviction, a maximum execute timeout, and
internal permission rules. It defaults to `StateBackend()`, computes paths for
large tool results and conversation history, stores config, and creates all
seven tools. The `execute` tool is always present in `self.tools`, but
`wrap_model_call()` removes it from requests when the resolved backend does not
support execution.

#### `_get_backend(runtime)`

Resolves `self.backend` into a `BackendProtocol`. If `backend` is callable,
the method emits a deprecation warning and calls it with `ToolRuntime`; direct
backend instances are returned unchanged.

#### `_create_ls_tool()`

Builds the `ls` tool. Both sync and async wrappers resolve the backend,
validate the absolute path, deny access if permissions reject the read, call
`ls()`/`als()`, filter returned entries by read permissions, truncate the path
list if needed, and return a success or error `ToolMessage`.

#### `_create_read_file_tool()`

Builds `read_file`. The nested `_truncate()` enforces line limits and a
character budget derived from the token limit. `_handle_read_result()` supports
deprecated string results, handles backend errors, formats text files with line
numbers, returns an empty-file warning when appropriate, and converts non-text
files into multimodal content blocks with MIME metadata.

The sync and async wrappers share the same path validation and permission
checks, then call `read()` or `aread()`.

#### `_create_write_file_tool()`

Builds `write_file`. The wrappers validate the path, check write permission,
call backend `write()`/`awrite()`, and return either the backend error or a
short success message naming the written path.

#### `_create_edit_file_tool()`

Builds `edit_file`. It validates path and write permission, then delegates
exact-string replacement to backend `edit()`/`aedit()`. On success it reports
the number of replacements and the edited path.

#### `_create_glob_tool()`

Builds `glob`. The sync wrapper runs backend `glob()` in a one-worker thread
and applies a 20-second timeout; the async wrapper uses `asyncio.wait_for()`.
Both validate the base path, check read permission, wrap timeout/backend
errors, filter matches by permission, truncate the resulting paths, and return
a `ToolMessage`.

#### `_create_grep_tool()`

Builds `grep`. It optionally validates and permission-checks the search path,
then calls backend `grep()`/`agrep()` with the literal pattern and optional
file glob. Matches are filtered by permission and formatted according to
`output_mode`: file list, matching content, or counts.

#### `_create_execute_tool()`

Builds `execute`. It validates that timeout is non-negative and no greater
than `max_execute_timeout`, resolves the backend, checks execution support,
checks whether that backend's `execute()` accepts timeout, and calls
`execute()`/`aexecute()`. It catches `NotImplementedError` and `ValueError` as
tool-facing errors.

Successful command output is formatted as combined output plus an exit-code
footer and optional truncation footer. This keeps command results simple for
the LLM.

#### `wrap_model_call()` and `awrap_model_call()`

These hooks inspect the available tools, remove `execute` if the backend cannot
execute commands, append filesystem and optional execution instructions to the
system message, and process oversized human messages before the model call.

When a new human message is evicted, the hook returns
`ExtendedModelResponse` so LangGraph state receives a `Command` tagging the
original message with `lc_evicted_to`. The model request sees a truncated
preview, but state retains the original message content.

#### `_process_large_message()` and `_aprocess_large_message()`

These helpers inspect a tool result's text content. If it exceeds the tool
eviction threshold, they write the full text under
`/large_tool_results/<tool_call_id>` (or the composite artifacts root), build a
head/tail preview, and return a replacement `ToolMessage` preserving id,
artifact, status, metadata, and non-text blocks.

If eviction is disabled, content is under the threshold, or the backend write
fails, the original message is returned unchanged.

#### `_get_backend_from_runtime()`

Resolves a backend from a bare LangGraph `Runtime`, used by model-call hooks
that do not receive `ToolRuntime`. For deprecated backend factories, it builds
a synthetic `ToolRuntime` from the current state, runtime context, writer,
store, and config.

#### `_check_eviction_needed()`, `_apply_eviction_and_truncate()`, `_evict_and_truncate_messages()`, `_aevict_and_truncate_messages()`

These helpers implement human-message eviction. `_check_eviction_needed()`
finds already-tagged human messages and detects whether the newest human
message crosses the threshold. `_evict_and_truncate_messages()` writes newly
oversized content to `/conversation_history/<uuid>.md`, then
`_apply_eviction_and_truncate()` tags the original message in state and replaces
all tagged human messages in the model request with previews.

The async variant mirrors the sync flow with `awrite()`.

#### `_intercept_large_tool_result()` and `_aintercept_large_tool_result()`

These helpers run after tool execution. They handle both direct `ToolMessage`
results and `Command(update={"messages": [...]})` results, replacing only large
tool messages and leaving non-tool messages untouched.

#### `wrap_tool_call()` and `awrap_tool_call()`

These hooks call the downstream tool handler first. If tool-result eviction is
disabled or the tool name is excluded from eviction, they return the raw
result. Otherwise they run the interception helpers above.

## System prompts and tool descriptions

`FILESYSTEM_SYSTEM_PROMPT`:

```text
## Following Conventions

- Read files before editing — understand existing content before making changes
- Mimic existing style, naming conventions, and patterns

## Filesystem Tools `ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`

You have access to a filesystem which you can interact with using these tools.
All file paths must start with a /. Follow the tool docs for the available tools, and use pagination (offset/limit) when reading large files.

- ls: list files in a directory (requires absolute path)
- read_file: read a file from the filesystem
- write_file: write to a file in the filesystem
- edit_file: edit a file in the filesystem
- glob: find files matching a pattern (e.g., "**/*.py")
- grep: search for text within files

## Large Tool Results

When a tool result is too large, it may be offloaded into the filesystem instead of being returned inline. In those cases, use `read_file` to inspect the saved result in chunks, or use `grep` within `/large_tool_results/` if you need to search across offloaded tool results and do not know the exact file path. Offloaded tool results are stored under `/large_tool_results/<tool_call_id>`.
```

`EXECUTION_SYSTEM_PROMPT`:

```text
## Execute Tool `execute`

You have access to an `execute` tool for running shell commands in a sandboxed environment.
Use this tool to run commands, scripts, tests, builds, and other shell operations.

- execute: run a shell command in the sandbox (returns output and exit code)
```

`LIST_FILES_TOOL_DESCRIPTION`:

```text
Lists all files in a directory.

This is useful for exploring the filesystem and finding the right file to read or edit.
You should almost ALWAYS use this tool before using the read_file or edit_file tools.
```

`READ_FILE_TOOL_DESCRIPTION`:

```text
Reads a file from the filesystem.

Assume this tool is able to read all files. If the User provides a path to a file assume that path is valid. It is okay to read a file that does not exist; an error will be returned.

Usage:
- By default, it reads up to 100 lines starting from the beginning of the file
- **IMPORTANT for large files and codebase exploration**: Use pagination with offset and limit parameters to avoid context overflow
  - First scan: read_file(path, limit=100) to see file structure
  - Read more sections: read_file(path, offset=100, limit=200) for next 200 lines
  - Only omit limit (read full file) when necessary for editing
- Specify offset and limit: read_file(path, offset=0, limit=100) reads first 100 lines
- Results are returned using cat -n format, with line numbers starting at 1
- Lines longer than 5,000 characters will be split into multiple lines with continuation markers (e.g., 5.1, 5.2, etc.). When you specify a limit, these continuation lines count towards the limit.
- You have the capability to call multiple tools in a single response. It is always better to speculatively read multiple files as a batch that are potentially useful.
- If you read a file that exists but has empty contents you will receive a system reminder warning in place of file contents.
- Image files (`.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, etc.), audio and video files, and PDFs are returned as multimodal content blocks (see https://docs.langchain.com/oss/python/langchain/messages#multimodal).

For multimodal reads (image, audio, video, PDF, etc.):
- Use `read_file(file_path=...)`
- Do NOT use `offset`/`limit` for images (pagination is text-only)
- If file details were compacted from history, call `read_file` again on the same path

- You should ALWAYS make sure a file has been read before editing it.
```

`EDIT_FILE_TOOL_DESCRIPTION`:

```text
Performs exact string replacements in files.

Usage:
- You must read the file before editing. This tool will error if you attempt an edit without reading the file first.
- When editing, preserve the exact indentation (tabs/spaces) from the read output. Never include line number prefixes in old_string or new_string.
- ALWAYS prefer editing existing files over creating new ones.
- Only use emojis if the user explicitly requests it.
```

`WRITE_FILE_TOOL_DESCRIPTION`:

```text
Writes to a new file in the filesystem.

Usage:
- The write_file tool will create the a new file.
- Prefer to edit existing files (with the edit_file tool) over creating new ones when possible.
```

`GLOB_TOOL_DESCRIPTION`:

```text
Find files matching a glob pattern.

Supports standard glob patterns: `*` (any characters), `**` (any directories), `?` (single character).
Returns a list of absolute file paths that match the pattern.

Examples:
- `**/*.py` - Find all Python files
- `*.txt` - Find all text files in root
- `/subdir/**/*.md` - Find all markdown files under /subdir
```

`GREP_TOOL_DESCRIPTION`:

```text
Search for a text pattern across files.

Searches for literal text (not regex) and returns matching files or content based on output_mode.
Special characters like parentheses, brackets, pipes, etc. are treated as literal characters, not regex operators.

Examples:
- Search all files: `grep(pattern="TODO")`
- Search Python files only: `grep(pattern="import", glob="*.py")`
- Show matching lines: `grep(pattern="error", output_mode="content")`
- Search for code with special chars: `grep(pattern="def __init__(self):")`
```

`EXECUTE_TOOL_DESCRIPTION`:

```text
Executes a shell command in an isolated sandbox environment.

Usage:
Executes a given command in the sandbox environment with proper handling and security measures.
Before executing the command, please follow these steps:
1. Directory Verification:
   - If the command will create new directories or files, first use the ls tool to verify the parent directory exists and is the correct location
   - For example, before running "mkdir foo/bar", first use ls to check that "foo" exists and is the intended parent directory
2. Command Execution:
   - Always quote file paths that contain spaces with double quotes (e.g., cd "path with spaces/file.txt")
   - Examples of proper quoting:
     - cd "/Users/name/My Documents" (correct)
     - cd /Users/name/My Documents (incorrect - will fail)
     - python "/path/with spaces/script.py" (correct)
     - python /path/with spaces/script.py (incorrect - will fail)
   - After ensuring proper quoting, execute the command
   - Capture the output of the command
Usage notes:
  - Commands run in an isolated sandbox environment
  - Returns combined stdout/stderr output with exit code
  - If the output is very large, it may be truncated
  - For long-running commands, use the optional timeout parameter to override the default timeout (e.g., execute(command="make build", timeout=300))
  - A timeout of 0 may disable timeouts on backends that support no-timeout execution
  - VERY IMPORTANT: You MUST avoid using search commands like find and grep. Instead use the grep, glob tools to search. You MUST avoid read tools like cat, head, tail, and use read_file to read files.
  - When issuing multiple commands, use the ';' or '&&' operator to separate them. DO NOT use newlines (newlines are ok in quoted strings)
    - Use '&&' when commands depend on each other (e.g., "mkdir dir && cd dir")
    - Use ';' only when you need to run commands sequentially but don't care if earlier commands fail
  - Try to maintain your current working directory throughout the session by using absolute paths and avoiding usage of cd

Examples:
  Good examples:
    - execute(command="pytest /foo/bar/tests")
    - execute(command="python /path/to/script.py")
    - execute(command="npm install && npm test")
    - execute(command="make build", timeout=300)

  Bad examples (avoid these):
    - execute(command="cd /foo/bar && pytest tests")  # Use absolute path instead
    - execute(command="cat file.txt")  # Use read_file tool instead
    - execute(command="find . -name '*.py'")  # Use glob tool instead
    - execute(command="grep -r 'pattern' .")  # Use grep tool instead

Note: This tool is only available if the backend supports execution (SandboxBackendProtocol).
If execution is not supported, the tool will return an error message.
```

## Flow walk-through

1. `filesystem.py:737` validates `max_execute_timeout`.
2. `filesystem.py:741` chooses `StateBackend()` unless a backend was supplied.
3. `filesystem.py:742` rejects unsupported permission/execution combinations.
4. `filesystem.py:769` creates all file and execute tools.
5. Each tool wrapper validates paths, applies permissions, calls backend
   methods, and returns a `ToolMessage`.
6. `filesystem.py:1649` filters `execute`, appends system prompt text, and
   evicts/truncates large human messages before model invocation.
7. `filesystem.py:2126` wraps tool calls after execution and evicts large tool
   results unless the tool is excluded.

## Gotchas

The `execute` tool may be present on `self.tools` but removed from a model
request if the runtime backend does not support `SandboxBackendProtocol`.

Permissions do not currently govern shell execution. That is why the
constructor rejects most permission configurations paired with executable
backends.

Large human-message eviction writes the full content to the backend and tags
state, while the model sees a preview. This preserves state fidelity while
protecting the context window.
